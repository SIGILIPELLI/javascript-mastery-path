---
description: "Working with Databases — better-sqlite3 is synchronous (no promises needed) and very fast for typical app workloads, which makes it an easy first database…"
---

# 04 · Working with Databases

## 🎥 Video walkthrough

<iframe width="100%" height="400" style="max-width:720px;aspect-ratio:16/9;height:auto;" src="https://www.youtube.com/embed/VPvOJLWewwc" title="Video walkthrough" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Arrays in memory disappear when the process restarts — real applications
persist data in a database. This module covers SQLite (a file-based
relational database, perfect for learning and for small/medium apps) with
`better-sqlite3`, and introduces MongoDB (a document database) conceptually
with the official `mongodb` driver.

## Why a real database?

| | In-memory array | SQLite | MongoDB |
|---|---|---|---|
| Survives restart | no | yes (file on disk) | yes (server process) |
| Query language | manual `.filter()` | SQL | MongoDB Query Language (JSON-like) |
| Structure | whatever you want | fixed columns per table (schema) | flexible documents (schema-optional) |
| Good for | prototypes, tests | most small/medium apps | rapidly changing / nested data |

## Setting up SQLite with better-sqlite3

`better-sqlite3` is synchronous (no promises needed) and very fast for
typical app workloads, which makes it an easy first database to learn.

```bash
npm install better-sqlite3
```

```javascript
// db.js
import Database from "better-sqlite3";

const db = new Database("app.db"); // creates app.db on disk if it doesn't exist

db.pragma("journal_mode = WAL"); // better concurrency for reads/writes

db.exec(`
  CREATE TABLE IF NOT EXISTS books (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    author TEXT NOT NULL,
    year INTEGER
  )
`);

export default db;
```

Running `db.exec(...)` with `CREATE TABLE IF NOT EXISTS` is a simple form of
**schema migration** — it defines the shape of your data (columns and
types) up front, unlike a schema-less document store.

## CRUD — Create

```javascript
// books.js
import db from "./db.js";

const insertStmt = db.prepare(
  "INSERT INTO books (title, author, year) VALUES (?, ?, ?)"
);

function createBook({ title, author, year }) {
  const result = insertStmt.run(title, author, year);
  return { id: result.lastInsertRowid, title, author, year };
}

console.log(createBook({ title: "Dune", author: "Frank Herbert", year: 1965 }));
// { id: 1, title: 'Dune', author: 'Frank Herbert', year: 1965 }
```

Prepared statements (`db.prepare(...)`) with `?` placeholders are the
standard defense against **SQL injection** — never build SQL strings with
template literals from user input.

## CRUD — Read

```javascript
const getAllStmt = db.prepare("SELECT * FROM books ORDER BY year");
const getByIdStmt = db.prepare("SELECT * FROM books WHERE id = ?");

function getAllBooks() {
  return getAllStmt.all(); // returns an array of row objects
}

function getBookById(id) {
  return getByIdStmt.get(id); // returns a single row object, or undefined
}

console.log(getAllBooks());
// [ { id: 1, title: 'Dune', author: 'Frank Herbert', year: 1965 } ]
console.log(getBookById(1));
// { id: 1, title: 'Dune', author: 'Frank Herbert', year: 1965 }
console.log(getBookById(999)); // undefined
```

## CRUD — Update and Delete

```javascript
const updateStmt = db.prepare(
  "UPDATE books SET title = ?, author = ?, year = ? WHERE id = ?"
);
const deleteStmt = db.prepare("DELETE FROM books WHERE id = ?");

function updateBook(id, { title, author, year }) {
  const result = updateStmt.run(title, author, year, id);
  return result.changes > 0; // true if a row was actually updated
}

function deleteBook(id) {
  const result = deleteStmt.run(id);
  return result.changes > 0;
}

console.log(updateBook(1, { title: "Dune", author: "Frank Herbert", year: 1965 })); // true
console.log(deleteBook(999)); // false — no matching row
```

## Node's built-in node:sqlite (newer Node versions)

Recent Node.js versions ship an experimental built-in SQLite module,
removing the need for a third-party package for simple cases.

```javascript
import { DatabaseSync } from "node:sqlite"; // available behind a flag on some Node versions

const db = new DatabaseSync("app.db");
db.exec("CREATE TABLE IF NOT EXISTS books (id INTEGER PRIMARY KEY, title TEXT)");

const insert = db.prepare("INSERT INTO books (title) VALUES (?)");
insert.run("Foundation");

const rows = db.prepare("SELECT * FROM books").all();
console.log(rows); // [ { id: 1, title: 'Foundation' } ]
```

## Schema/data modeling basics

Relational schemas model relationships with **foreign keys** — a column in
one table referencing the primary key of another.

```javascript
db.exec(`
  CREATE TABLE IF NOT EXISTS authors (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL
  );

  CREATE TABLE IF NOT EXISTS books (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    author_id INTEGER NOT NULL,
    FOREIGN KEY (author_id) REFERENCES authors(id)
  );
`);

// A join fetches related rows across both tables in one query
const booksWithAuthors = db.prepare(`
  SELECT books.title, authors.name AS author
  FROM books
  JOIN authors ON books.author_id = authors.id
`).all();
```

## MongoDB — a document database (conceptual + example)

MongoDB stores flexible JSON-like **documents** in **collections** instead
of rows in tables — there's no fixed set of columns, which suits data whose
shape varies or nests deeply.

```bash
npm install mongodb
```

```javascript
import { MongoClient } from "mongodb";

const client = new MongoClient("mongodb://localhost:27017");

async function run() {
  await client.connect();
  const db = client.db("library");
  const books = db.collection("books");

  await books.insertOne({
    title: "Dune",
    author: "Frank Herbert",
    year: 1965,
    tags: ["sci-fi", "classic"], // arrays/nested data are natural in documents
  });

  const found = await books.findOne({ title: "Dune" });
  console.log(found);
  // { _id: ..., title: 'Dune', author: 'Frank Herbert', year: 1965, tags: [...] }

  await books.updateOne({ title: "Dune" }, { $set: { year: 1965 } });
  await books.deleteOne({ title: "Dune" });

  await client.close();
}

run();
```

## SQL vs. document model

| | SQLite (relational) | MongoDB (document) |
|---|---|---|
| Unit of storage | row in a table | document in a collection |
| Schema | fixed columns, enforced | flexible, optional |
| Relationships | foreign keys + `JOIN` | embed related data, or reference by `_id` |
| Query language | SQL | MongoDB Query Language (method calls + JSON filters) |
| Good fit | structured, tabular data | nested/variable-shape data, rapid iteration |

## How It Actually Works

A database driver's `.query()` call in Node never blocks the event loop waiting on the
network round-trip to the database server — it opens a TCP socket, writes the query as
bytes according to the database's wire protocol, and returns a promise immediately;
libuv's epoll-based socket handling notifies Node when response bytes arrive, at which
point the driver parses the wire-protocol response and resolves your promise from
within the poll phase's I/O callback. This is why a Node app can hold open dozens of
concurrent, in-flight database queries on a single thread — each is just a registered
"wake me up when this socket is readable" entry, not a thread of its own.

Connection **pooling** exists because opening a fresh TCP connection (plus, for many
databases, an auth handshake and TLS negotiation) costs multiple round-trips — a pool
pre-opens a fixed number of connections and hands them out to queries as they arrive,
queuing extra requests until a connection frees up. This directly trades memory/open
sockets for latency: too small a pool serializes concurrent requests behind each other
waiting for a free connection; too large a pool can overwhelm the database server's own
per-connection resource limits (each open connection typically costs the database
server real memory for its own session state, independent of your Node process).
## 🔀 See this in another language

- [C# — 07 · Working with Databases (EF Core migrations)](https://sigilipelli.github.io/csharp-mastery-path/level-3/07-ef-core-migrations/)

## Exercise

Using `better-sqlite3`, create a `students` table (`id`, `name`, `grade`)
and write four functions — `addStudent`, `listStudents`, `updateGrade`, and
`removeStudent` — each using a prepared statement. Then write a small script
that adds three students, updates one's grade, deletes another, and prints
the final list sorted by grade descending.
