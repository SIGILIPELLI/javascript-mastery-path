# 07 · Testing Frameworks Deep Dive

Level 2 introduced Jest for unit tests. This module goes deeper: mocking
dependencies, spying on function calls, and writing integration tests that
exercise a real Express API with `supertest`.

## Setup

```bash
npm install --save-dev jest supertest
```

```json
{
  "scripts": {
    "test": "jest"
  }
}
```

## Mocking modules with jest.mock

A **mock** replaces a real dependency (a database call, an email service, an
external API) with a fake version you control, so tests are fast and don't
depend on network/external state.

```javascript
// emailService.js
export function sendEmail(to, subject) {
  // in real life this would call a third-party API
  throw new Error("sendEmail should never run for real in tests");
}
```

```javascript
// signup.js
import { sendEmail } from "./emailService.js";

export function signUp(user) {
  sendEmail(user.email, "Welcome!");
  return { ...user, welcomed: true };
}
```

```javascript
// signup.test.js
import { signUp } from "./signup.js";
import { sendEmail } from "./emailService.js";

jest.mock("./emailService.js"); // replaces every export with an auto-mocked jest.fn()

test("signUp sends a welcome email and marks the user welcomed", () => {
  const user = { email: "ada@example.com" };

  const result = signUp(user);

  expect(sendEmail).toHaveBeenCalledWith("ada@example.com", "Welcome!");
  expect(sendEmail).toHaveBeenCalledTimes(1);
  expect(result.welcomed).toBe(true);
});
```

## Spies — watching real functions without replacing their behavior

`jest.spyOn` wraps an existing method so you can assert on calls to it,
while still (optionally) letting the real implementation run.

```javascript
// logger.js
export const logger = {
  info(message) {
    console.log(`[INFO] ${message}`);
  },
};
```

```javascript
// logger.test.js
import { logger } from "./logger.js";

test("logger.info is called with the right message", () => {
  const spy = jest.spyOn(logger, "info").mockImplementation(() => {}); // suppress real console.log

  logger.info("server started");

  expect(spy).toHaveBeenCalledWith("server started");
  spy.mockRestore(); // restore the original implementation afterward
});
```

## Mock return values and implementations

```javascript
function fetchUserRole(userId, api) {
  return api.getRole(userId);
}

test("fetchUserRole returns whatever the api client reports", () => {
  const mockApi = {
    getRole: jest.fn().mockReturnValue("admin"),
  };

  expect(fetchUserRole(1, mockApi)).toBe("admin");
  expect(mockApi.getRole).toHaveBeenCalledWith(1);
});

test("mockImplementation can vary behavior per call", () => {
  const mockApi = { getRole: jest.fn() };
  mockApi.getRole
    .mockImplementationOnce(() => "admin")   // first call
    .mockImplementationOnce(() => "viewer"); // second call

  expect(fetchUserRole(1, mockApi)).toBe("admin");
  expect(fetchUserRole(2, mockApi)).toBe("viewer");
});
```

## Testing async code and rejected promises

```javascript
async function fetchWithRetry(fetcher, retries = 1) {
  try {
    return await fetcher();
  } catch (error) {
    if (retries <= 0) throw error;
    return fetchWithRetry(fetcher, retries - 1);
  }
}

test("fetchWithRetry retries once before giving up", async () => {
  const fetcher = jest
    .fn()
    .mockRejectedValueOnce(new Error("network blip"))
    .mockResolvedValueOnce("success");

  const result = await fetchWithRetry(fetcher, 1);

  expect(result).toBe("success");
  expect(fetcher).toHaveBeenCalledTimes(2);
});

test("fetchWithRetry throws after exhausting retries", async () => {
  const fetcher = jest.fn().mockRejectedValue(new Error("still failing"));

  await expect(fetchWithRetry(fetcher, 1)).rejects.toThrow("still failing");
});
```

## Integration testing an Express API with supertest

Unit tests mock dependencies away; **integration tests** exercise the real
app — routes, middleware, and all — by making actual HTTP requests against
it in-process, without needing a running server or open port.

```javascript
// app.js
import express from "express";

export function createApp() {
  const app = express();
  app.use(express.json());

  const books = [{ id: 1, title: "Dune" }];

  app.get("/books", (req, res) => res.json(books));

  app.post("/books", (req, res) => {
    const { title } = req.body;
    if (!title) return res.status(400).json({ error: "title is required" });
    const book = { id: books.length + 1, title };
    books.push(book);
    res.status(201).json(book);
  });

  app.get("/books/:id", (req, res) => {
    const book = books.find((b) => b.id === Number(req.params.id));
    if (!book) return res.status(404).json({ error: "not found" });
    res.json(book);
  });

  return app;
}
```

```javascript
// app.test.js
import request from "supertest";
import { createApp } from "./app.js";

const app = createApp();

test("GET /books returns the seeded book", async () => {
  const response = await request(app).get("/books");

  expect(response.status).toBe(200);
  expect(response.body).toEqual([{ id: 1, title: "Dune" }]);
});

test("POST /books creates a new book", async () => {
  const response = await request(app)
    .post("/books")
    .send({ title: "Foundation" })
    .set("Content-Type", "application/json");

  expect(response.status).toBe(201);
  expect(response.body).toMatchObject({ title: "Foundation" });
});

test("POST /books without a title responds 400", async () => {
  const response = await request(app).post("/books").send({});

  expect(response.status).toBe(400);
  expect(response.body.error).toBe("title is required");
});

test("GET /books/:id returns 404 for an unknown id", async () => {
  const response = await request(app).get("/books/999");

  expect(response.status).toBe(404);
});
```

```bash
npm test
# PASS  ./app.test.js
#  ✓ GET /books returns the seeded book
#  ✓ POST /books creates a new book
#  ✓ POST /books without a title responds 400
#  ✓ GET /books/:id returns 404 for an unknown id
```

Notice `createApp()` returns the Express `app` **without** calling
`.listen()` — `supertest` starts an ephemeral server internally per test, so
routes are testable without binding a real port or worrying about test runs
colliding on the same port.

## Unit vs. integration tests

| | Unit test | Integration test |
|---|---|---|
| Scope | one function/module in isolation | multiple pieces working together (routes + middleware) |
| Dependencies | mocked | mostly real (real Express app) |
| Speed | very fast | fast, but slower than pure unit tests |
| Example here | `signUp` with `sendEmail` mocked | `GET /books` through the real Express app |

## Exercise

Extend the `createApp()` example with a `DELETE /books/:id` route. Write
three supertest tests for it: deleting an existing book returns `204` and a
subsequent `GET` for that id returns `404`; deleting a non-existent id
returns `404`. Then add one Jest mock-based unit test for a hypothetical
`notifyBookRemoved(id, notifier)` function, asserting it calls
`notifier.send` with the correct message using `jest.fn()`.
