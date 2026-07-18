# 11 · Project — REST API + Database

A complete backend project combining everything from Level 3: Express
routing and middleware, SQLite persistence, input validation, centralized
error handling, and automated tests with Jest + supertest.

## What you'll build

A **Book Catalog API** — a small REST service backed by a real SQLite
database file, with full CRUD:

- `GET /books` — list all books
- `GET /books/:id` — get one book
- `POST /books` — create a book (validated)
- `PUT /books/:id` — update a book (validated)
- `DELETE /books/:id` — delete a book
- Centralized error-handling middleware so nothing crashes the server
- A Jest + supertest suite covering every route

## Project layout

```text
book-catalog-api/
    package.json
    src/
        db.js
        booksController.js
        routes.js
        app.js
        server.js
    test/
        books.test.js
```

## package.json

```json
{
  "name": "book-catalog-api",
  "version": "1.0.0",
  "type": "module",
  "main": "src/server.js",
  "scripts": {
    "start": "node src/server.js",
    "test": "jest"
  },
  "dependencies": {
    "better-sqlite3": "^11.0.0",
    "express": "^4.19.0"
  },
  "devDependencies": {
    "jest": "^29.7.0",
    "supertest": "^7.0.0"
  }
}
```

## src/db.js — database setup

```javascript
// src/db.js
import Database from "better-sqlite3";

export function createDb(filename = "catalog.db") {
  const db = new Database(filename);
  db.pragma("journal_mode = WAL");

  db.exec(`
    CREATE TABLE IF NOT EXISTS books (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      title TEXT NOT NULL,
      author TEXT NOT NULL,
      year INTEGER
    )
  `);

  return db;
}

export default createDb;
```

Using an in-memory database (`:memory:`) for tests, and a real file for the
running app, is what lets the same schema/logic be exercised safely in both
places — tests never touch `catalog.db` on disk.

## src/booksController.js — data access + validation

```javascript
// src/booksController.js

export function createBooksController(db) {
  const insertStmt = db.prepare(
    "INSERT INTO books (title, author, year) VALUES (?, ?, ?)"
  );
  const selectAllStmt = db.prepare("SELECT * FROM books ORDER BY id");
  const selectByIdStmt = db.prepare("SELECT * FROM books WHERE id = ?");
  const updateStmt = db.prepare(
    "UPDATE books SET title = ?, author = ?, year = ? WHERE id = ?"
  );
  const deleteStmt = db.prepare("DELETE FROM books WHERE id = ?");

  function validateBook(body, { partial = false } = {}) {
    const errors = [];

    if (!partial || body.title !== undefined) {
      if (typeof body.title !== "string" || body.title.trim() === "") {
        errors.push("title is required and must be a non-empty string");
      }
    }
    if (!partial || body.author !== undefined) {
      if (typeof body.author !== "string" || body.author.trim() === "") {
        errors.push("author is required and must be a non-empty string");
      }
    }
    if (body.year !== undefined && !Number.isInteger(body.year)) {
      errors.push("year must be an integer when provided");
    }

    return errors;
  }

  function listBooks() {
    return selectAllStmt.all();
  }

  function getBook(id) {
    return selectByIdStmt.get(id);
  }

  function createBook({ title, author, year }) {
    const result = insertStmt.run(title, author, year ?? null);
    return getBook(result.lastInsertRowid);
  }

  function updateBook(id, existing, updates) {
    const merged = {
      title: updates.title ?? existing.title,
      author: updates.author ?? existing.author,
      year: updates.year ?? existing.year,
    };
    updateStmt.run(merged.title, merged.author, merged.year, id);
    return getBook(id);
  }

  function deleteBook(id) {
    const result = deleteStmt.run(id);
    return result.changes > 0;
  }

  return { validateBook, listBooks, getBook, createBook, updateBook, deleteBook };
}
```

## src/routes.js — Express router

```javascript
// src/routes.js
import { Router } from "express";
import { createBooksController } from "./booksController.js";

export function createBooksRouter(db) {
  const router = Router();
  const controller = createBooksController(db);

  router.get("/", (req, res) => {
    res.json(controller.listBooks());
  });

  router.get("/:id", (req, res, next) => {
    try {
      const book = controller.getBook(Number(req.params.id));
      if (!book) return res.status(404).json({ error: "book not found" });
      res.json(book);
    } catch (error) {
      next(error);
    }
  });

  router.post("/", (req, res, next) => {
    try {
      const errors = controller.validateBook(req.body);
      if (errors.length > 0) return res.status(400).json({ errors });
      const book = controller.createBook(req.body);
      res.status(201).json(book);
    } catch (error) {
      next(error);
    }
  });

  router.put("/:id", (req, res, next) => {
    try {
      const id = Number(req.params.id);
      const existing = controller.getBook(id);
      if (!existing) return res.status(404).json({ error: "book not found" });

      const errors = controller.validateBook(req.body, { partial: true });
      if (errors.length > 0) return res.status(400).json({ errors });

      const updated = controller.updateBook(id, existing, req.body);
      res.json(updated);
    } catch (error) {
      next(error);
    }
  });

  router.delete("/:id", (req, res, next) => {
    try {
      const deleted = controller.deleteBook(Number(req.params.id));
      if (!deleted) return res.status(404).json({ error: "book not found" });
      res.status(204).end();
    } catch (error) {
      next(error);
    }
  });

  return router;
}
```

## src/app.js — wiring it together

```javascript
// src/app.js
import express from "express";
import { createBooksRouter } from "./routes.js";

export function createApp(db) {
  const app = express();
  app.use(express.json());

  app.use((req, res, next) => {
    console.log(`${req.method} ${req.url}`);
    next();
  });

  app.use("/books", createBooksRouter(db));

  // 404 fallback for unknown routes
  app.use((req, res) => {
    res.status(404).json({ error: "route not found" });
  });

  // Centralized error handler — catches anything passed to next(error)
  app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).json({ error: "internal server error" });
  });

  return app;
}
```

## src/server.js — entry point

```javascript
// src/server.js
import { createDb } from "./db.js";
import { createApp } from "./app.js";

const db = createDb("catalog.db");
const app = createApp(db);

const PORT = process.env.PORT ?? 3000;
app.listen(PORT, () => {
  console.log(`Book Catalog API listening on http://localhost:${PORT}`);
});
```

## test/books.test.js — Jest + supertest

```javascript
// test/books.test.js
import request from "supertest";
import { createDb } from "../src/db.js";
import { createApp } from "../src/app.js";

let app;

beforeEach(() => {
  const db = createDb(":memory:"); // fresh in-memory database per test
  app = createApp(db);
});

test("GET /books starts empty", async () => {
  const response = await request(app).get("/books");
  expect(response.status).toBe(200);
  expect(response.body).toEqual([]);
});

test("POST /books creates a book", async () => {
  const response = await request(app)
    .post("/books")
    .send({ title: "Dune", author: "Frank Herbert", year: 1965 });

  expect(response.status).toBe(201);
  expect(response.body).toMatchObject({ title: "Dune", author: "Frank Herbert", year: 1965 });
  expect(response.body.id).toBeDefined();
});

test("POST /books rejects a missing title", async () => {
  const response = await request(app).post("/books").send({ author: "Unknown" });

  expect(response.status).toBe(400);
  expect(response.body.errors).toContain("title is required and must be a non-empty string");
});

test("GET /books/:id returns a created book", async () => {
  const created = await request(app)
    .post("/books")
    .send({ title: "Foundation", author: "Isaac Asimov" });

  const response = await request(app).get(`/books/${created.body.id}`);

  expect(response.status).toBe(200);
  expect(response.body.title).toBe("Foundation");
});

test("GET /books/:id returns 404 for an unknown id", async () => {
  const response = await request(app).get("/books/999");
  expect(response.status).toBe(404);
});

test("PUT /books/:id updates only the provided fields", async () => {
  const created = await request(app)
    .post("/books")
    .send({ title: "Old Title", author: "Someone", year: 2000 });

  const response = await request(app)
    .put(`/books/${created.body.id}`)
    .send({ title: "New Title" });

  expect(response.status).toBe(200);
  expect(response.body.title).toBe("New Title");
  expect(response.body.author).toBe("Someone"); // untouched fields preserved
});

test("DELETE /books/:id removes the book", async () => {
  const created = await request(app)
    .post("/books")
    .send({ title: "Temporary", author: "Nobody" });

  const deleteResponse = await request(app).delete(`/books/${created.body.id}`);
  expect(deleteResponse.status).toBe(204);

  const getResponse = await request(app).get(`/books/${created.body.id}`);
  expect(getResponse.status).toBe(404);
});

test("unknown routes return 404", async () => {
  const response = await request(app).get("/not-a-real-route");
  expect(response.status).toBe(404);
});
```

## Running it

```bash
npm install
npm start
```

```bash
curl http://localhost:3000/books
# []

curl -X POST http://localhost:3000/books \
  -H "Content-Type: application/json" \
  -d '{"title":"Dune","author":"Frank Herbert","year":1965}'
# {"id":1,"title":"Dune","author":"Frank Herbert","year":1965}
```

```bash
npm test
# PASS  test/books.test.js
#  ✓ GET /books starts empty
#  ✓ POST /books creates a book
#  ✓ POST /books rejects a missing title
#  ✓ GET /books/:id returns a created book
#  ✓ GET /books/:id returns 404 for an unknown id
#  ✓ PUT /books/:id updates only the provided fields
#  ✓ DELETE /books/:id removes the book
#  ✓ unknown routes return 404
```

## Stretch goals

- Add a `GET /books?author=...` query filter, and pagination via
  `?page=1&limit=10`.
- Add a second resource, `authors`, with a foreign key from `books` to
  `authors` (see the schema-modeling section of
  [Module 4](04-databases.md)), and a `GET /authors/:id/books` route.
- Add basic API-key middleware (as in
  [Module 3](03-rest-api-express.md)) protecting the `POST`/`PUT`/`DELETE`
  routes, plus a supertest case asserting `401` without a key.
- Add request validation with `zod` (see
  [Module 9](09-security-basics.md)) instead of the hand-written
  `validateBook` checks.
- Swap SQLite for MongoDB (see [Module 4](04-databases.md)) and adjust the
  controller to use the `mongodb` driver instead of prepared statements,
  keeping the routes/tests unchanged to see how cleanly the data layer can
  be swapped out.

Completing this project means you're ready for **Level 4 · Master**.
