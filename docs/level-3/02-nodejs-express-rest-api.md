---
description: "Node.js Fundamentals & REST APIs with Express — Part 1: Node.js Fundamentals; Part 2: Building a REST API with Express."
---

# 02 · Node.js Fundamentals & REST APIs with Express

This module takes JavaScript to the server in two steps: first Node.js itself (modules, the file system, JSON and streams), then building a REST API on top of it with Express (routing, middleware, routers and error handling).

## 🎥 Video walkthroughs

**Part 1 — Node.js Fundamentals**

<iframe width="100%" height="400" style="max-width:720px;aspect-ratio:16/9;height:auto;" src="https://www.youtube.com/embed/RuuMhtb45Rc" title="Video walkthrough" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Level 3 moves off the browser and onto the server. Node.js runs JavaScript
outside a browser, giving you access to the file system, network sockets,
and the OS — none of which exist in browser JavaScript. This module covers
Node's two module systems, reading/writing files, and streams.

**Part 2 — Building a REST API with Express**

<iframe width="100%" height="400" style="max-width:720px;aspect-ratio:16/9;height:auto;" src="https://www.youtube.com/embed/4UUPYCWH17I" title="Video walkthrough" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Express is the most widely used Node.js web framework — a thin layer over
Node's built-in `http` module that adds routing, middleware, and convenient
request/response helpers. This module builds a small REST API from scratch.

## Part 1 · Node.js Fundamentals

### CommonJS modules

CommonJS (`require`/`module.exports`) is Node's original module system and
still the default for plain `.js` files.

```javascript
// math.cjs
function add(a, b) {
  return a + b;
}

function multiply(a, b) {
  return a * b;
}

module.exports = { add, multiply };
```

```javascript
// main.cjs
const { add, multiply } = require("./math.cjs");

console.log(add(2, 3));      // 5
console.log(multiply(4, 5)); // 20
```

```bash
node main.cjs
```

CommonJS `require` calls are synchronous and can happen anywhere in a file
(even conditionally), which is one reason the newer ESM system was
introduced with stricter, more static import rules.

### ES modules in Node

To use `import`/`export` syntax directly, either name files `.mjs` or set
`"type": "module"` in `package.json` — then plain `.js` files are treated as
ES modules.

```json
{
  "name": "node-fundamentals-demo",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node main.js"
  }
}
```

```javascript
// math.js
export function add(a, b) {
  return a + b;
}

export function multiply(a, b) {
  return a * b;
}
```

```javascript
// main.js
import { add, multiply } from "./math.js"; // note the required .js extension

console.log(add(2, 3));      // 5
console.log(multiply(4, 5)); // 20
```

### Mixing systems: what to know

You can't `require()` an ES module, but ESM code can `import` most CommonJS
packages, and Node exposes a handful of globals differently between the two
systems (`__dirname`/`__filename` exist only in CommonJS).

```javascript
// ESM equivalent of __dirname / __filename
import { fileURLToPath } from "node:url";
import { dirname } from "node:path";

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

console.log(__dirname);  // absolute path to this file's folder
console.log(__filename); // absolute path to this file
```

| Feature | CommonJS | ES Modules |
|---------|----------|------------|
| Import | `require("./file")` | `import x from "./file.js"` |
| Export | `module.exports = ...` | `export`/`export default` |
| File extension in imports | optional | required |
| `__dirname` / `__filename` | built in | derive via `import.meta.url` |
| Enabled by | default for `.js` | `"type": "module"` or `.mjs` |
| Loading | synchronous | can be async (top-level `await`) |

### The file system module — callbacks vs. promises

Node's original `fs` module is callback-based. `fs/promises` wraps the same
operations in promises, which is almost always the better fit for modern
`async`/`await` code.

```javascript
// Old style: callback-based fs
import fs from "node:fs";

fs.readFile("notes.txt", "utf8", (error, data) => {
  if (error) {
    console.error("read failed:", error.message);
    return;
  }
  console.log("contents:", data);
});
```

```javascript
// Modern style: fs/promises
import { readFile, writeFile, appendFile, mkdir } from "node:fs/promises";

async function saveNote(text) {
  await mkdir("data", { recursive: true }); // create the folder if missing
  await writeFile("data/notes.txt", text, "utf8");
}

async function readNote() {
  try {
    const contents = await readFile("data/notes.txt", "utf8");
    console.log("note contents:", contents);
  } catch (error) {
    if (error.code === "ENOENT") {
      console.log("no note saved yet");
    } else {
      throw error;
    }
  }
}

await saveNote("Buy milk\n");
await appendFile("data/notes.txt", "Walk the dog\n");
await readNote();
// note contents: Buy milk
// Walk the dog
```

### Working with JSON files

A very common Node task is treating a JSON file as lightweight storage —
read it, parse it, mutate the in-memory data, then write it back.

```javascript
import { readFile, writeFile } from "node:fs/promises";

const DB_PATH = "data/users.json";

async function loadUsers() {
  try {
    const raw = await readFile(DB_PATH, "utf8");
    return JSON.parse(raw);
  } catch (error) {
    if (error.code === "ENOENT") return []; // first run — file doesn't exist yet
    throw error;
  }
}

async function saveUsers(users) {
  await writeFile(DB_PATH, JSON.stringify(users, null, 2), "utf8");
}

const users = await loadUsers();
users.push({ id: users.length + 1, name: "Grace" });
await saveUsers(users);
console.log(users); // [ { id: 1, name: 'Grace' } ]
```

### Streams — processing data in chunks

Reading a whole file into memory with `readFile` is fine for small files,
but large files (or network data) should be processed as a **stream** —
data arrives in chunks so memory usage stays flat regardless of file size.

```javascript
import { createReadStream, createWriteStream } from "node:fs";

const readStream = createReadStream("data/notes.txt", { encoding: "utf8" });
const writeStream = createWriteStream("data/notes-upper.txt");

readStream.on("data", (chunk) => {
  writeStream.write(chunk.toUpperCase()); // transform each chunk as it arrives
});

readStream.on("end", () => {
  writeStream.end();
  console.log("done transforming file");
});

readStream.on("error", (error) => {
  console.error("stream failed:", error.message);
});
```

### Piping streams

`.pipe()` connects a readable stream to a writable one and handles
backpressure automatically (pausing the source if the destination can't keep
up) — almost always preferable to manual `.on("data")` handling.

```javascript
import { createReadStream, createWriteStream } from "node:fs";
import { createGzip } from "node:zlib";

createReadStream("data/notes.txt")
  .pipe(createGzip()) // Transform stream: compresses chunks as they pass through
  .pipe(createWriteStream("data/notes.txt.gz"))
  .on("finish", () => console.log("compressed notes.txt -> notes.txt.gz"));
```

### Stream types

| Type | Direction | Example |
|------|-----------|---------|
| Readable | data flows out | `fs.createReadStream`, an incoming HTTP request body |
| Writable | data flows in | `fs.createWriteStream`, an HTTP response |
| Duplex | both | a TCP socket |
| Transform | in, modified, out | `zlib.createGzip()`, a CSV parser |

### How It Actually Works

Node's event loop, implemented by libuv, isn't a single queue — it's a sequence of
distinct **phases** that run in a fixed order every tick: timers (due `setTimeout`/
`setInterval` callbacks), pending callbacks, idle/prepare (internal), poll (retrieve new
I/O events, execute I/O callbacks — this phase can block here waiting for events if
there's nothing else to do), check (`setImmediate` callbacks specifically live here),
and close callbacks. Microtasks (promise `.then`, `queueMicrotask`) and
`process.nextTick` callbacks are **not** a phase — Node drains the `nextTick` queue and
then the microtask queue completely after *every single callback*, not just once per
loop iteration, which is why `process.nextTick` callbacks can starve I/O entirely if
they keep scheduling more of themselves.

The reason Node can do non-blocking file I/O in a single-threaded language is that
CPU/blocking-style operations (`fs.readFile`, DNS lookups, some crypto) are handed off
to libuv's **thread pool** (default size 4, tunable via `UV_THREADPOOL_SIZE`) — actual
OS threads do the blocking read, and when it completes, libuv queues the JS callback to
run on the single main thread during the poll phase. Network I/O (sockets, HTTP) on
Linux doesn't even need the thread pool — it uses the OS's native async I/O
notification (`epoll`), so `fetch`/`http.get` calls scale to far more concurrent
connections than thread-pool-bound file operations, because there's no fixed pool size
limiting them.
### Exercise

Write a small Node script (ESM, `"type": "module"`) that reads a directory
path from `process.argv`, lists all `.txt` files in it using `fs/promises`,
and streams the concatenated contents of every file into a single
`combined.txt` using `createReadStream`/`createWriteStream` and `.pipe()`.
Handle the case where the directory doesn't exist with a clear error message
instead of a crash.

## Part 2 · Building a REST API with Express

### Installing and starting a server

```bash
npm install express
```

```javascript
// server.js
import express from "express";

const app = express();
const PORT = 3000;

app.get("/", (req, res) => {
  res.send("API is running");
});

app.listen(PORT, () => {
  console.log(`Server listening on http://localhost:${PORT}`);
});
```

```bash
node server.js
# Server listening on http://localhost:3000
```

### Routing

Routes match an HTTP method and a URL path to a handler function. Route
parameters (`:id`) capture dynamic segments of the URL.

```javascript
import express from "express";

const app = express();
app.use(express.json()); // parses JSON request bodies into req.body

const notes = [
  { id: 1, text: "Buy milk" },
  { id: 2, text: "Walk the dog" },
];

app.get("/notes", (req, res) => {
  res.json(notes); // sends JSON + sets Content-Type automatically
});

app.get("/notes/:id", (req, res) => {
  const note = notes.find((n) => n.id === Number(req.params.id));
  if (!note) {
    return res.status(404).json({ error: "note not found" });
  }
  res.json(note);
});

app.post("/notes", (req, res) => {
  const { text } = req.body;
  if (!text) {
    return res.status(400).json({ error: "text is required" });
  }
  const note = { id: notes.length + 1, text };
  notes.push(note);
  res.status(201).json(note); // 201 Created
});

app.listen(3000);
```

```bash
curl http://localhost:3000/notes
# [{"id":1,"text":"Buy milk"},{"id":2,"text":"Walk the dog"}]

curl -X POST http://localhost:3000/notes \
  -H "Content-Type: application/json" \
  -d '{"text":"Read a book"}'
# {"id":3,"text":"Read a book"}
```

### Query strings and request data

```javascript
// GET /search?q=milk&limit=5
app.get("/search", (req, res) => {
  const { q, limit = "10" } = req.query; // req.query holds parsed query-string params
  res.json({
    query: q,
    limit: Number(limit),
    results: notes.filter((n) => n.text.toLowerCase().includes((q ?? "").toLowerCase())),
  });
});
```

| Request data | Accessed via | Example |
|--------------|---------------|---------|
| Route params | `req.params` | `/notes/:id` -> `req.params.id` |
| Query string | `req.query` | `?q=milk` -> `req.query.q` |
| JSON body | `req.body` (needs `express.json()`) | POST payload |
| Headers | `req.headers` | `req.headers["authorization"]` |

### Middleware

Middleware functions run between the request arriving and the route handler
— they can inspect/modify `req`/`res`, or short-circuit the request. Calling
`next()` passes control to the next middleware or route handler; not calling
it leaves the request hanging.

```javascript
// A simple request logger, applied to every route
app.use((req, res, next) => {
  console.log(`${new Date().toISOString()} ${req.method} ${req.url}`);
  next(); // must be called or the request never reaches its route
});

// Middleware scoped to one route: a naive API-key check
function requireApiKey(req, res, next) {
  if (req.headers["x-api-key"] !== "secret123") {
    return res.status(401).json({ error: "invalid or missing API key" });
  }
  next();
}

app.delete("/notes/:id", requireApiKey, (req, res) => {
  const index = notes.findIndex((n) => n.id === Number(req.params.id));
  if (index === -1) return res.status(404).json({ error: "note not found" });
  notes.splice(index, 1);
  res.status(204).end(); // 204 No Content — success, nothing to return
});
```

### Express Router — organizing routes across files

Real APIs split routes by resource instead of putting everything in one
file. `express.Router()` creates a mini standalone router that the main app
mounts under a path prefix.

```javascript
// routes/notes.js
import { Router } from "express";

const router = Router();
const notes = [{ id: 1, text: "Buy milk" }];

router.get("/", (req, res) => res.json(notes));

router.get("/:id", (req, res) => {
  const note = notes.find((n) => n.id === Number(req.params.id));
  if (!note) return res.status(404).json({ error: "not found" });
  res.json(note);
});

export default router;
```

```javascript
// server.js
import express from "express";
import notesRouter from "./routes/notes.js";

const app = express();
app.use(express.json());
app.use("/notes", notesRouter); // all routes above are now prefixed with /notes

app.listen(3000);
```

### Error-handling middleware

Express recognizes an error-handling middleware by its **four** parameters
`(err, req, res, next)`. Place it last, after all routes, to catch anything
thrown or passed to `next(error)`.

```javascript
app.get("/boom", () => {
  throw new Error("something went wrong");
});

// Error-handling middleware — must be defined AFTER all routes
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: "internal server error" });
});
```

### HTTP status codes cheat sheet

| Code | Meaning | Typical use |
|------|---------|--------------|
| 200 | OK | successful GET/PUT |
| 201 | Created | successful POST that creates a resource |
| 204 | No Content | successful DELETE, nothing to return |
| 400 | Bad Request | validation failure |
| 401 | Unauthorized | missing/invalid credentials |
| 404 | Not Found | resource doesn't exist |
| 500 | Internal Server Error | unhandled exception |

### How It Actually Works

Express's entire request-handling model is a **middleware chain**: each `app.use` or
route handler is stored in an internal array along with the path pattern it matches,
and dispatching a request walks that array in registration order, calling each matching
middleware with a `next` function that, when invoked, resumes the walk to the next
entry. This is just a manually-managed continuation-passing pattern — there's no magic
scheduling involved, `next()` is a plain synchronous function call that recurses into
the next middleware, which is exactly why forgetting to call `next()` (or `res.end()`)
in a middleware hangs the request forever: the walk simply never continues.

Because Node handles one request per "tick" of JS execution but the underlying HTTP
server accepts many connections concurrently via the OS's async I/O, a single Express
process genuinely serves many simultaneous requests — but only if your handlers yield
control back to the event loop between blocking-ish work (via `await`, callbacks, or
`setImmediate`). A synchronous CPU-heavy handler (e.g., hashing a huge payload
in a tight loop) blocks the *entire* process, delaying every other in-flight request's
processing, because there is exactly one JS thread regardless of how many TCP
connections libuv is juggling underneath it.
### 🔀 See this in another language

- [TypeScript — 10 · Project — REST API with Express](https://learnmasterypath.com/typescript-mastery-path/level-3/10-project-rest-api-express/)
- [Python — 11 · Project — REST API + Database](https://learnmasterypath.com/python-mastery-path/level-3/10-project-rest-api-db/)
- [Java — 11 · Project — REST API + Database CRUD Service](https://learnmasterypath.com/java-mastery-path/level-3/10-project-rest-api-db/)

### Exercise

Build an Express API for a `/tasks` resource with an in-memory array:
`GET /tasks` (list all), `GET /tasks/:id`, `POST /tasks` (validate that
`title` is a non-empty string, respond `400` otherwise), `PATCH /tasks/:id`
to toggle a `done` boolean, and `DELETE /tasks/:id`. Add a logging
middleware and a final error-handling middleware that catches anything
unexpected and responds with a `500` JSON error instead of crashing the
server.
