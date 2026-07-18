# 07 · Error Handling Advanced

This module extends
[Level 1 · Module 8](../level-1/08-error-handling.md) into custom error
types and the asynchronous cases that basic `try`/`catch` doesn't cover on
its own.

## Custom `Error` subclasses

Extending the built-in `Error` class lets calling code distinguish failure
kinds by type, instead of parsing message strings.

```javascript
class ValidationError extends Error {
  constructor(message, field) {
    super(message);       // sets this.message
    this.name = "ValidationError"; // overrides the default "Error"
    this.field = field;    // extra context specific to this error type
  }
}

class NotFoundError extends Error {
  constructor(resource, id) {
    super(`${resource} with id ${id} not found`);
    this.name = "NotFoundError";
    this.resource = resource;
    this.id = id;
  }
}

function findUser(users, id) {
  const user = users.find((u) => u.id === id);
  if (!user) throw new NotFoundError("User", id);
  return user;
}

try {
  findUser([{ id: 1, name: "Ada" }], 99);
} catch (error) {
  console.log(error.name);    // NotFoundError
  console.log(error.message); // User with id 99 not found
  console.log(error instanceof Error); // true — still a real Error
}
```

## Branching on error type

With custom classes, `catch` blocks can handle different failures
differently using `instanceof`, instead of one generic handler for
everything.

```javascript
function validateAge(age) {
  if (typeof age !== "number") {
    throw new ValidationError("age must be a number", "age");
  }
  if (age < 0) {
    throw new ValidationError("age cannot be negative", "age");
  }
  return age;
}

function processSignup(age) {
  try {
    validateAge(age);
    console.log("signup accepted");
  } catch (error) {
    if (error instanceof ValidationError) {
      console.log(`validation problem on '${error.field}': ${error.message}`);
    } else {
      console.log("unexpected error:", error.message);
      throw error; // re-throw errors we don't know how to handle
    }
  }
}

processSignup(-5);    // validation problem on 'age': age cannot be negative
processSignup("old"); // validation problem on 'age': age must be a number
processSignup(30);    // signup accepted
```

## Errors in `async`/`await` code

`try`/`catch` around `await` catches both synchronous throws and rejected
Promises — this is the main reason `async`/`await` is easier to reason about
than raw `.then()` chains.

```javascript
async function fetchWithValidation(id) {
  if (id <= 0) {
    throw new ValidationError("id must be positive", "id"); // sync throw
  }

  const response = await delay(id); // could reject — both are caught below
  return response;
}

function delay(id) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (id > 100) reject(new NotFoundError("Item", id));
      else resolve({ id, name: `item-${id}` });
    }, 50);
  });
}

async function run() {
  for (const id of [-1, 500, 5]) {
    try {
      const item = await fetchWithValidation(id);
      console.log("loaded:", item);
    } catch (error) {
      console.log(`${error.name}: ${error.message}`);
    }
  }
}

run();
// ValidationError: id must be positive
// NotFoundError: Item with id 500 not found
// loaded: { id: 5, name: 'item-5' }
```

## Unhandled promise rejections

A rejected Promise with no `.catch()` (and not awaited inside a `try`) is an
**unhandled rejection** — silent in older code, but modern runtimes warn or
crash on it. Always attach error handling to every Promise chain you start.

```javascript
function riskyOperation() {
  return new Promise((resolve, reject) => {
    reject(new Error("boom"));
  });
}

// Missing entirely: riskyOperation(); would trigger an unhandledRejection warning

// Handled correctly:
riskyOperation().catch((error) => {
  console.log("caught it:", error.message); // caught it: boom
});
```

In Node.js, you can add a last-resort safety net for anything that slips
through — useful for logging, not for routine error handling:

```javascript
process.on("unhandledRejection", (reason) => {
  console.error("Unhandled rejection:", reason);
});

process.on("uncaughtException", (error) => {
  console.error("Uncaught exception:", error);
  process.exit(1); // the process is in an unknown state — restart rather than continue
});
```

Browsers have an equivalent event for the same purpose:

```javascript
window.addEventListener("unhandledrejection", (event) => {
  console.error("Unhandled rejection:", event.reason);
});
```

## Wrapping and re-throwing with context

When an error crosses a boundary (e.g. from a data layer into business
logic), wrapping it preserves the original cause while adding useful
context — the `cause` option keeps the chain traceable.

```javascript
async function loadConfig(path) {
  try {
    const response = await fetch(path);
    if (!response.ok) throw new Error(`status ${response.status}`);
    return await response.json();
  } catch (error) {
    throw new Error(`failed to load config from ${path}`, { cause: error });
  }
}

loadConfig("/missing.json").catch((error) => {
  console.log(error.message);      // failed to load config from /missing.json
  console.log(error.cause?.message); // the original underlying error
});
```

## Error-handling strategy cheat sheet

| Situation | Strategy |
|-----------|----------|
| Expected, recoverable failure (bad input) | throw a specific `Error` subclass, `catch` and handle it |
| Unexpected failure inside `async` code | `try`/`catch` around `await`, or `.catch()` on the promise chain |
| Failure crossing a module boundary | wrap with `{ cause: originalError }` for traceability |
| Promise created but never awaited/caught | always attach `.catch()`, or listen for `unhandledrejection`/`unhandledRejection` as a safety net |
| Truly unknown/corrupted state | log and crash/restart (`uncaughtException`) rather than continue silently |

## Exercise

Create a `TimeoutError` class extending `Error`. Then write an `async`
function `withRetry(fn, retries)` that calls `fn()` (an async function that
may reject), retrying up to `retries` times on failure, and finally throwing
a `TimeoutError` with a message like `"failed after 3 attempts"` if every
attempt fails.
