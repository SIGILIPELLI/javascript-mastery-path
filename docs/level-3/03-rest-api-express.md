# 03 · Building a REST API with Express

Express is the most widely used Node.js web framework — a thin layer over
Node's built-in `http` module that adds routing, middleware, and convenient
request/response helpers. This module builds a small REST API from scratch.

## Installing and starting a server

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

## Routing

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

## Query strings and request data

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

## Middleware

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

## Express Router — organizing routes across files

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

## Error-handling middleware

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

## HTTP status codes cheat sheet

| Code | Meaning | Typical use |
|------|---------|--------------|
| 200 | OK | successful GET/PUT |
| 201 | Created | successful POST that creates a resource |
| 204 | No Content | successful DELETE, nothing to return |
| 400 | Bad Request | validation failure |
| 401 | Unauthorized | missing/invalid credentials |
| 404 | Not Found | resource doesn't exist |
| 500 | Internal Server Error | unhandled exception |

## Exercise

Build an Express API for a `/tasks` resource with an in-memory array:
`GET /tasks` (list all), `GET /tasks/:id`, `POST /tasks` (validate that
`title` is a non-empty string, respond `400` otherwise), `PATCH /tasks/:id`
to toggle a `done` boolean, and `DELETE /tasks/:id`. Add a logging
middleware and a final error-handling middleware that catches anything
unexpected and responds with a `500` JSON error instead of crashing the
server.
