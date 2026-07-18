# 04 · Promises & Async/Await

This module builds on the event loop and microtask queue from
[Module 3](03-async-javascript-i.md) — Promises are the mechanism that
schedules work onto that microtask queue.

## What a Promise represents

A Promise is an object representing a value that may not be available yet.
It's always in one of three states: **pending**, **fulfilled**, or
**rejected** — and once it settles (fulfills or rejects), it never changes
again.

```javascript
const willSucceed = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("done!"); // moves the promise to "fulfilled"
  }, 500);
});

willSucceed.then((value) => console.log(value)); // done! (after ~500ms)
```

## Creating a Promise around async work

Wrap any callback-based operation in `new Promise` to give it a `.then`/
`.catch` interface, calling `resolve` on success and `reject` on failure.

```javascript
function delay(ms, value) {
  return new Promise((resolve, reject) => {
    if (ms < 0) {
      reject(new Error("delay must be non-negative"));
      return;
    }
    setTimeout(() => resolve(value), ms);
  });
}

delay(100, "hello")
  .then((result) => console.log(result)) // hello
  .catch((error) => console.log(error.message));

delay(-1, "hello").catch((error) => console.log(error.message));
// delay must be non-negative
```

## Chaining `.then`

Each `.then` returns a new Promise, so steps that depend on the previous
result can be chained instead of nested — this directly replaces the
callback pyramid from Module 3.

```javascript
function getUser(id) {
  return delay(100, { id, name: "Ada" });
}
function getOrders(user) {
  return delay(100, [`order-1 for ${user.name}`, `order-2 for ${user.name}`]);
}

getUser(1)
  .then((user) => getOrders(user))
  .then((orders) => console.log(orders[0])) // order-1 for Ada
  .catch((error) => console.log("something failed:", error.message));
```

## `async`/`await`

`async`/`await` is syntax sugar over Promises: an `async` function always
returns a Promise, and `await` pauses execution inside it until the awaited
Promise settles — without blocking the rest of the program.

```javascript
async function loadOrderSummary() {
  const user = await getUser(1);        // pauses here until resolved
  const orders = await getOrders(user); // then continues
  return `${user.name}: ${orders.length} orders`;
}

loadOrderSummary().then((summary) => console.log(summary));
// Ada: 2 orders
```

The same logic written with raw `.then` chains is functionally identical,
but `await` reads top-to-bottom like synchronous code.

## Error handling with `try`/`catch`

Wrap `await` calls in `try`/`catch` exactly like synchronous code — this is
the big ergonomic win over `.then().catch()` chains for multi-step logic.

```javascript
async function safeLoadOrderSummary(id) {
  try {
    const user = await getUser(id);
    const orders = await getOrders(user);
    return `${user.name}: ${orders.length} orders`;
  } catch (error) {
    console.log("failed to load summary:", error.message);
    return null;
  }
}

safeLoadOrderSummary(1).then((summary) => console.log(summary));
// Ada: 2 orders
```

## Running promises in parallel

Sequential `await` calls that don't depend on each other waste time. Kick
off independent operations together and await them as a group instead.

```javascript
async function loadDashboard() {
  // Sequential — slower, waits for each one before starting the next
  const a = await delay(200, "profile");
  const b = await delay(200, "settings");
  // total: ~400ms

  // Parallel — starts both immediately, waits for whichever is slower
  const [profile, settings] = await Promise.all([
    delay(200, "profile"),
    delay(200, "settings"),
  ]);
  // total: ~200ms

  return { a, b, profile, settings };
}

loadDashboard().then((result) => console.log(result));
```

## `Promise.all` vs. `Promise.allSettled`

`Promise.all` rejects as soon as *any* promise rejects, discarding the
others' results. `Promise.allSettled` always waits for every promise and
reports each outcome individually — useful when partial failure is fine.

```javascript
const tasks = [delay(50, "ok-1"), Promise.reject(new Error("task failed")), delay(50, "ok-2")];

Promise.all(tasks).catch((error) => console.log("all() rejected:", error.message));
// all() rejected: task failed

Promise.allSettled(tasks).then((results) => console.log(results));
// [
//   { status: 'fulfilled', value: 'ok-1' },
//   { status: 'rejected', reason: Error: task failed },
//   { status: 'fulfilled', value: 'ok-2' }
// ]
```

## Promise combinators cheat sheet

| Method | Resolves when | Rejects when |
|--------|----------------|----------------|
| `Promise.all` | every promise fulfills | any single promise rejects (fails fast) |
| `Promise.allSettled` | every promise settles (fulfilled or rejected) | never — always fulfills with a status report |
| `Promise.race` | the first promise to settle, settles | the first promise to settle rejects |
| `Promise.any` | the first promise to fulfill, fulfills | only if *all* promises reject |

## Exercise

Write an `async` function `fetchWithTimeout(promise, ms)` that returns
whatever `promise` resolves to, but rejects with `new Error("timed out")` if
it takes longer than `ms` milliseconds. Hint: use `Promise.race` with a
second promise that rejects after a `setTimeout`.
