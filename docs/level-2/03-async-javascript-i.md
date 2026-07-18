# 03 · Asynchronous JavaScript I

JavaScript runs on a single thread, yet it handles timers, network requests,
and user input without blocking. This module covers the mechanics that make
that possible — the foundation for Promises and `async`/`await` in the next
module.

## Synchronous code and the call stack

JavaScript executes one thing at a time, in order, using a call stack: each
function call is pushed on, and popped off when it returns.

```javascript
function third() {
  console.log("third");
}
function second() {
  console.log("second");
  third();
}
function first() {
  console.log("first");
  second();
}

first();
// first
// second
// third
```

Each call waits for the ones it invokes to finish completely before
continuing — this is what "synchronous" and "blocking" mean.

## Callbacks: functions passed to run later

A callback is simply a function passed as an argument, to be called at some
later point — often after an operation finishes.

```javascript
function fetchUserName(id, callback) {
  console.log("starting lookup...");
  setTimeout(() => {
    // simulate work that takes time, e.g. a network request
    callback(`user-${id}`);
  }, 1000);
}

fetchUserName(42, (name) => {
  console.log(`got: ${name}`); // runs ~1 second later
});

console.log("lookup requested, moving on"); // runs immediately, before the callback
```

Output order:

```text
starting lookup...
lookup requested, moving on
got: user-42
```

The last line prints *after* the synchronous code has already finished,
because `setTimeout` doesn't block — it schedules the callback for later.

## The event loop, conceptually

Node.js and browsers use an event loop to coordinate the call stack with
pending work. The mental model:

1. Run everything on the call stack until it's empty.
2. Check whether there's pending callback work ready to run.
3. Push the next ready callback onto the stack and run it.
4. Repeat forever.

```javascript
console.log("1: sync");

setTimeout(() => console.log("3: timeout callback"), 0);

console.log("2: sync");
// 1: sync
// 2: sync
// 3: timeout callback
```

Even with a delay of `0`, the timeout callback still runs *after* all
synchronous code — it has to wait for the call stack to empty first.

## Task queue vs. microtask queue

Not all "later" work is treated equally. Promise callbacks (`.then`,
`async`/`await` continuations, covered next module) go into the **microtask
queue**, which the event loop always fully drains before touching the
**task queue** (also called the macrotask queue) that holds things like
`setTimeout` and DOM events.

```javascript
console.log("1: sync start");

setTimeout(() => console.log("4: setTimeout (task queue)"), 0);

Promise.resolve().then(() => console.log("3: promise (microtask queue)"));

console.log("2: sync end");
// 1: sync start
// 2: sync end
// 3: promise (microtask queue)
// 4: setTimeout (task queue)
```

Even though both are scheduled with a "0 delay," the microtask always wins —
all queued microtasks run before the next macrotask is picked up.

## Callback hell — the problem Promises solve

Nesting callback after callback to sequence dependent async steps quickly
becomes hard to read and error-prone (each level needs its own error
handling).

```javascript
function getUser(id, callback) {
  setTimeout(() => callback({ id, name: "Ada" }), 100);
}
function getOrders(userId, callback) {
  setTimeout(() => callback(["order-1", "order-2"]), 100);
}
function getOrderDetails(orderId, callback) {
  setTimeout(() => callback(`${orderId}: shipped`), 100);
}

// Deeply nested — the "pyramid of doom"
getUser(1, (user) => {
  getOrders(user.id, (orders) => {
    getOrderDetails(orders[0], (details) => {
      console.log(details); // order-1: shipped
    });
  });
});
```

Each step must wait on the previous one, and error handling would need a
check at every single level. Promises (next module) exist specifically to
flatten this structure and centralize error handling.

## Queue behavior cheat sheet

| Source | Queue | Runs relative to sync code | Runs relative to other queue |
|--------|-------|------------------------------|-------------------------------|
| `console.log`, loops, direct calls | call stack | immediately | — |
| `Promise.then`/`catch`/`finally`, `async` continuations | microtask queue | after current sync code finishes | before any pending macrotask |
| `setTimeout`, `setInterval`, DOM events | task (macro) queue | after current sync code finishes | after all pending microtasks drain |

## Exercise

Predict the exact console output order of the snippet below, then run it to
check your answer:

```javascript
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve()
  .then(() => console.log("C"))
  .then(() => console.log("D"));

console.log("E");
```
