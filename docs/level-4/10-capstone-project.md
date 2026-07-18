# 10 · Capstone Project

This is the final project of the entire Master path — and of the whole
JavaScript Mastery curriculum. It pulls together everything from all four
levels into one production-shaped service: a REST API with a real data
layer, JWT authentication, automated tests, a container image, and a CI
pipeline that builds and verifies all of it.

## What you'll build

A **Task Tracker API** — users register, log in, and manage their own tasks
through a JWT-protected REST API backed by SQLite (swappable for Postgres/
MySQL without touching business logic, since the repository layer is the
only place that knows about SQL).

## Project layout

```text
task-tracker-api/
  src/
    app.js                  # Express app wiring — routes, middleware, error handling
    server.js                # entry point — starts the HTTP server
    db.js                    # SQLite connection + schema setup
    config.js                # environment-driven configuration
    auth/
      auth.routes.js          # /auth/register, /auth/login
      auth.middleware.js      # JWT verification middleware
      tokens.js                # sign/verify JWT helpers
    users/
      users.repository.js
    tasks/
      tasks.routes.js
      tasks.repository.js
  tests/
    tasks.test.js             # Jest + supertest integration tests
  Dockerfile
  docker-compose.yml
  package.json
  .env.example
  .github/
    workflows/
      ci.yml
```

## package.json

```json
{
  "name": "task-tracker-api",
  "version": "1.0.0",
  "type": "module",
  "main": "src/server.js",
  "scripts": {
    "start": "node src/server.js",
    "dev": "node --watch src/server.js",
    "lint": "eslint src tests",
    "test": "NODE_ENV=test jest --runInBand"
  },
  "dependencies": {
    "better-sqlite3": "^11.3.0",
    "bcrypt": "^5.1.1",
    "express": "^4.19.2",
    "jsonwebtoken": "^9.0.2"
  },
  "devDependencies": {
    "eslint": "^9.9.0",
    "jest": "^29.7.0",
    "supertest": "^7.0.0"
  }
}
```

## src/config.js — environment-driven configuration

```javascript
// src/config.js
export const config = {
  port: process.env.PORT || 3000,
  jwtSecret: process.env.JWT_SECRET || "dev-secret-change-me",
  dbPath: process.env.DB_PATH || (process.env.NODE_ENV === "test" ? ":memory:" : "./data/tasks.db"),
};
```

## src/db.js — SQLite connection and schema

`better-sqlite3` gives synchronous, fast access to a file-based (or in-memory,
for tests) SQLite database — no separate server process to run locally.

```javascript
// src/db.js
import Database from "better-sqlite3";
import { config } from "./config.js";

export const db = new Database(config.dbPath);

db.pragma("journal_mode = WAL");

db.exec(`
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    created_at TEXT NOT NULL DEFAULT (datetime('now'))
  );

  CREATE TABLE IF NOT EXISTS tasks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL REFERENCES users(id),
    title TEXT NOT NULL,
    done INTEGER NOT NULL DEFAULT 0,
    created_at TEXT NOT NULL DEFAULT (datetime('now'))
  );
`);
```

## src/users/users.repository.js — the only file that knows SQL for users

```javascript
// src/users/users.repository.js
import { db } from "../db.js";

export const usersRepository = {
  findByEmail(email) {
    return db.prepare("SELECT * FROM users WHERE email = ?").get(email);
  },

  create({ email, passwordHash }) {
    const result = db
      .prepare("INSERT INTO users (email, password_hash) VALUES (?, ?)")
      .run(email, passwordHash);
    return { id: result.lastInsertRowid, email };
  },
};
```

## src/auth/tokens.js — JWT signing and verification

```javascript
// src/auth/tokens.js
import jwt from "jsonwebtoken";
import { config } from "../config.js";

export function signToken(user) {
  return jwt.sign({ sub: user.id, email: user.email }, config.jwtSecret, {
    expiresIn: "1h",
  });
}

export function verifyToken(token) {
  return jwt.verify(token, config.jwtSecret); // throws if invalid/expired
}
```

## src/auth/auth.middleware.js

```javascript
// src/auth/auth.middleware.js
import { verifyToken } from "./tokens.js";

export function requireAuth(req, res, next) {
  const header = req.headers.authorization;
  const token = header?.startsWith("Bearer ") ? header.slice(7) : null;

  if (!token) {
    return res.status(401).json({ error: "missing authorization token" });
  }

  try {
    req.user = verifyToken(token); // { sub, email, iat, exp }
    next();
  } catch {
    res.status(401).json({ error: "invalid or expired token" });
  }
}
```

## src/auth/auth.routes.js — register and login

```javascript
// src/auth/auth.routes.js
import { Router } from "express";
import bcrypt from "bcrypt";
import { usersRepository } from "../users/users.repository.js";
import { signToken } from "./tokens.js";

export const authRouter = Router();
const SALT_ROUNDS = 10;

authRouter.post("/register", async (req, res) => {
  const { email, password } = req.body;

  if (!email || !password || password.length < 8) {
    return res.status(400).json({ error: "email and an 8+ character password are required" });
  }

  if (usersRepository.findByEmail(email)) {
    return res.status(409).json({ error: "email already registered" });
  }

  const passwordHash = await bcrypt.hash(password, SALT_ROUNDS);
  const user = usersRepository.create({ email, passwordHash });
  const token = signToken(user);

  res.status(201).json({ token, user: { id: user.id, email: user.email } });
});

authRouter.post("/login", async (req, res) => {
  const { email, password } = req.body;
  const user = usersRepository.findByEmail(email);

  if (!user || !(await bcrypt.compare(password, user.password_hash))) {
    return res.status(401).json({ error: "invalid email or password" });
  }

  const token = signToken(user);
  res.json({ token, user: { id: user.id, email: user.email } });
});
```

## src/tasks/tasks.repository.js and tasks.routes.js — the protected resource

```javascript
// src/tasks/tasks.repository.js
import { db } from "../db.js";

export const tasksRepository = {
  listForUser(userId) {
    return db.prepare("SELECT * FROM tasks WHERE user_id = ? ORDER BY id").all(userId);
  },

  create(userId, title) {
    const result = db
      .prepare("INSERT INTO tasks (user_id, title) VALUES (?, ?)")
      .run(userId, title);
    return { id: result.lastInsertRowid, userId, title, done: false };
  },

  findOwnedById(userId, id) {
    return db.prepare("SELECT * FROM tasks WHERE id = ? AND user_id = ?").get(id, userId);
  },

  setDone(userId, id, done) {
    const result = db
      .prepare("UPDATE tasks SET done = ? WHERE id = ? AND user_id = ?")
      .run(done ? 1 : 0, id, userId);
    return result.changes > 0; // false means no matching row for this user — not found or not owned
  },
};
```

```javascript
// src/tasks/tasks.routes.js
import { Router } from "express";
import { requireAuth } from "../auth/auth.middleware.js";
import { tasksRepository } from "./tasks.repository.js";

export const tasksRouter = Router();
tasksRouter.use(requireAuth); // every route below requires a valid JWT

tasksRouter.get("/", (req, res) => {
  const tasks = tasksRepository.listForUser(req.user.sub);
  res.json(tasks);
});

tasksRouter.post("/", (req, res) => {
  const { title } = req.body;
  if (!title || !title.trim()) {
    return res.status(400).json({ error: "title is required" });
  }
  const task = tasksRepository.create(req.user.sub, title.trim());
  res.status(201).json(task);
});

tasksRouter.patch("/:id/done", (req, res) => {
  const { id } = req.params;
  const { done } = req.body;

  // findOwnedById scopes by user_id — this is what stops one user from
  // reading or editing another user's tasks by guessing IDs (see Module 4)
  const task = tasksRepository.findOwnedById(req.user.sub, id);
  if (!task) {
    return res.status(404).json({ error: "task not found" });
  }

  tasksRepository.setDone(req.user.sub, id, Boolean(done));
  res.json({ ...task, done: Boolean(done) });
});
```

## src/app.js — wiring it all together

```javascript
// src/app.js
import express from "express";
import { authRouter } from "./auth/auth.routes.js";
import { tasksRouter } from "./tasks/tasks.routes.js";

export const app = express();
app.use(express.json());

app.get("/health", (req, res) => res.json({ status: "ok" }));
app.use("/auth", authRouter);
app.use("/tasks", tasksRouter);

// Centralized error handler — catches anything thrown/rejected in routes above
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: "internal server error" });
});
```

## src/server.js — entry point

```javascript
// src/server.js
import { app } from "./app.js";
import { config } from "./config.js";

app.listen(config.port, () => {
  console.log(`task-tracker-api listening on port ${config.port}`);
});
```

## tests/tasks.test.js — Jest + supertest

Because `db.js` reads `config.dbPath`, and `config.js` uses an in-memory
SQLite database whenever `NODE_ENV=test`, the test suite runs against a real,
fresh database with zero setup — no mocking the data layer.

```javascript
// tests/tasks.test.js
import request from "supertest";
import { app } from "../src/app.js";

async function registerAndLogin(email = "ada@example.com", password = "s3curepass") {
  const response = await request(app).post("/auth/register").send({ email, password });
  return response.body.token;
}

describe("auth", () => {
  it("registers a new user and returns a token", async () => {
    const response = await request(app)
      .post("/auth/register")
      .send({ email: "grace@example.com", password: "s3curepass" });

    expect(response.status).toBe(201);
    expect(response.body.token).toBeDefined();
    expect(response.body.user.email).toBe("grace@example.com");
  });

  it("rejects duplicate registration with 409", async () => {
    await request(app).post("/auth/register").send({ email: "dup@example.com", password: "s3curepass" });
    const response = await request(app).post("/auth/register").send({ email: "dup@example.com", password: "s3curepass" });
    expect(response.status).toBe(409);
  });

  it("rejects login with a wrong password", async () => {
    await request(app).post("/auth/register").send({ email: "bob@example.com", password: "s3curepass" });
    const response = await request(app).post("/auth/login").send({ email: "bob@example.com", password: "wrong" });
    expect(response.status).toBe(401);
  });
});

describe("tasks", () => {
  it("rejects unauthenticated requests", async () => {
    const response = await request(app).get("/tasks");
    expect(response.status).toBe(401);
  });

  it("creates and lists tasks for the authenticated user only", async () => {
    const token = await registerAndLogin("owner@example.com");

    await request(app)
      .post("/tasks")
      .set("Authorization", `Bearer ${token}`)
      .send({ title: "Write capstone tests" });

    const response = await request(app)
      .get("/tasks")
      .set("Authorization", `Bearer ${token}`);

    expect(response.status).toBe(200);
    expect(response.body).toHaveLength(1);
    expect(response.body[0]).toMatchObject({ title: "Write capstone tests", done: 0 });
  });

  it("marks a task done", async () => {
    const token = await registerAndLogin("done-test@example.com");
    const createResponse = await request(app)
      .post("/tasks")
      .set("Authorization", `Bearer ${token}`)
      .send({ title: "Ship it" });

    const taskId = createResponse.body.id;

    const patchResponse = await request(app)
      .patch(`/tasks/${taskId}/done`)
      .set("Authorization", `Bearer ${token}`)
      .send({ done: true });

    expect(patchResponse.status).toBe(200);
    expect(patchResponse.body.done).toBe(true);
  });

  it("returns 404 when patching another user's task", async () => {
    const ownerToken = await registerAndLogin("owner2@example.com");
    const createResponse = await request(app)
      .post("/tasks")
      .set("Authorization", `Bearer ${ownerToken}`)
      .send({ title: "Private task" });

    const strangerToken = await registerAndLogin("stranger@example.com");
    const response = await request(app)
      .patch(`/tasks/${createResponse.body.id}/done`)
      .set("Authorization", `Bearer ${strangerToken}`)
      .send({ done: true });

    expect(response.status).toBe(404); // ownership check in findOwnedById does its job
  });
});
```

## Dockerfile

```dockerfile
# Dockerfile
FROM node:20-slim AS base
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci --omit=dev

COPY src ./src

ENV NODE_ENV=production
ENV PORT=3000
EXPOSE 3000

CMD ["node", "src/server.js"]
```

## docker-compose.yml

```yaml
# docker-compose.yml — runs the API with a persisted SQLite file on a volume
services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      JWT_SECRET: ${JWT_SECRET:-dev-secret-change-me}
      DB_PATH: /data/tasks.db
      PORT: 3000
    volumes:
      - task-data:/data

volumes:
  task-data:
```

## .env.example

```text
PORT=3000
JWT_SECRET=replace-with-a-long-random-string
DB_PATH=./data/tasks.db
```

## .github/workflows/ci.yml

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm test

  docker-build:
    runs-on: ubuntu-latest
    needs: lint-and-test
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - run: docker build -t task-tracker-api:${{ github.sha }} .
```

## Running it locally

```bash
npm install
cp .env.example .env    # then edit JWT_SECRET
npm test                 # runs the full Jest + supertest suite against an in-memory DB
npm run dev               # starts the API on http://localhost:3000
```

```bash
# try it end-to-end with curl
curl -X POST http://localhost:3000/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"ada@example.com","password":"s3curepass"}'
# => { "token": "...", "user": { "id": 1, "email": "ada@example.com" } }

TOKEN="paste-the-token-here"

curl -X POST http://localhost:3000/tasks \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"title":"Finish the Master path"}'

curl http://localhost:3000/tasks -H "Authorization: Bearer $TOKEN"
```

## Running it with Docker

```bash
docker compose up --build
# API available at http://localhost:3000, SQLite file persisted in the
# "task-data" named volume across container restarts
```

## Where to go from here

If you want to keep extending this capstone as a portfolio piece: swap
`better-sqlite3` for Postgres via a repository-layer change only (the routes
and services shouldn't need to change — that's the point of the layered
architecture from [Module 1](01-fullstack-architecture.md)); add structured
logging and a `/health/ready` check from [Module 9](09-observability.md); add
rate limiting to `/auth/login` from [Module 4](04-advanced-security.md); and
add a Playwright E2E suite driving a small frontend against this API, wired
into the CI workflow as its own job, per [Module 5](05-testing-at-scale.md)
and [Module 8](08-cicd.md).

## Congratulations

You've completed the full JavaScript Mastery Path — from your first
`console.log` in Level 1 through closures, async patterns, and design
patterns in Levels 2 and 3, to architecture, security, scaling,
observability, and a real production-shaped service here in Level 4. The
skills in this capstone — layered code you can test in isolation,
authentication done properly, a CI pipeline that actually gates bad changes,
and a container image anyone can run with one command — are exactly what
separates a script that works on your machine from a service a team can run
in production. Take this project, make it yours, and keep building.
