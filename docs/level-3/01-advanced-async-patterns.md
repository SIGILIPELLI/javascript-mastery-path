# 01 · Advanced Async Patterns

Level 2 covered `Promise` and `async`/`await` for single asynchronous
operations. Real backend code usually has to juggle several async operations
at once — running them in parallel, racing them, or tolerating some of them
failing. This module covers the combinator methods on `Promise`, async
iteration, and generator-based async flows.

## Promise.all — run in parallel, fail fast

`Promise.all` runs promises concurrently and resolves with an array of
results in the same order they were passed in. If **any** promise rejects,
`Promise.all` rejects immediately with that error — the other promises keep
running, but their results are discarded.

```javascript
function fetchUser(id) {
  return new Promise((resolve) => {
    setTimeout(() => resolve({ id, name: `User ${id}` }), 50);
  });
}

function fetchOrders(userId) {
  return new Promise((resolve) => {
    setTimeout(() => resolve([`order-${userId}-1`, `order-${userId}-2`]), 30);
  });
}

async function loadDashboard(userId) {
  const [user, orders] = await Promise.all([
    fetchUser(userId),
    fetchOrders(userId),
  ]);
  return { user, orders };
}

loadDashboard(7).then(console.log);
// { user: { id: 7, name: 'User 7' }, orders: [ 'order-7-1', 'order-7-2' ] }
// Both requests ran concurrently — total wait is ~50ms, not ~80ms.
```

```javascript
async function failingAll() {
  try {
    await Promise.all([
      Promise.resolve(1),
      Promise.reject(new Error("second call failed")),
      Promise.resolve(3),
    ]);
  } catch (error) {
    console.log("caught:", error.message); // caught: second call failed
  }
}

failingAll();
```

## Promise.allSettled — wait for everything, fail-safe

`Promise.allSettled` never short-circuits — it waits for every promise to
finish and reports the outcome of each one, whether it fulfilled or rejected.
Use it when you want partial results even if some operations fail (e.g.
pinging several services and reporting which ones are up).

```javascript
async function checkServices() {
  const results = await Promise.allSettled([
    Promise.resolve("auth-service: ok"),
    Promise.reject(new Error("payments-service: timeout")),
    Promise.resolve("search-service: ok"),
  ]);

  for (const result of results) {
    if (result.status === "fulfilled") {
      console.log("OK:", result.value);
    } else {
      console.log("FAILED:", result.reason.message);
    }
  }
}

checkServices();
// OK: auth-service: ok
// FAILED: payments-service: timeout
// OK: search-service: ok
```

## Promise.race and Promise.any

`Promise.race` settles as soon as the *first* promise settles — fulfilled or
rejected. `Promise.any` settles as soon as the first promise **fulfills**,
ignoring rejections unless every promise rejects (in which case it rejects
with an `AggregateError`).

```javascript
function delay(ms, value) {
  return new Promise((resolve) => setTimeout(() => resolve(value), ms));
}

async function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error("timed out")), ms)
  );
  return Promise.race([promise, timeout]); // whichever settles first wins
}

withTimeout(delay(200, "slow response"), 50)
  .then(console.log)
  .catch((error) => console.log("race error:", error.message));
// race error: timed out — the 50ms timeout beat the 200ms delay
```

```javascript
async function fastestMirror() {
  const winner = await Promise.any([
    delay(100, "mirror-a"),
    delay(20, "mirror-b"),
    delay(60, "mirror-c"),
  ]);
  console.log("first to respond:", winner); // first to respond: mirror-b
}

fastestMirror();
```

## Combinator cheat sheet

| Method | Settles when | Rejects when |
|--------|---------------|--------------|
| `Promise.all` | all fulfill | any one rejects (immediately) |
| `Promise.allSettled` | all settle (fulfilled or rejected) | never — always fulfills |
| `Promise.race` | the first one settles (either way) | first settled one is a rejection |
| `Promise.any` | the first one fulfills | all reject (`AggregateError`) |

## Async iterators and for-await-of

An **async iterable** is an object whose values arrive over time, each
wrapped in a promise. `for await...of` consumes them one at a time, awaiting
each promise automatically.

```javascript
function delay(ms, value) {
  return new Promise((resolve) => setTimeout(() => resolve(value), ms));
}

const asyncNumbers = {
  [Symbol.asyncIterator]() {
    let current = 0;
    return {
      next() {
        current += 1;
        if (current > 3) return Promise.resolve({ done: true });
        return delay(20, { done: false, value: current });
      },
    };
  },
};

async function consume() {
  for await (const value of asyncNumbers) {
    console.log("received:", value);
  }
  console.log("done");
}

consume();
// received: 1
// received: 2
// received: 3
// done
```

## Async generators

An **async generator** (`async function*`) is the ergonomic way to build an
async iterable — it can `await` inside the generator body and `yield` values
as they become ready, which is exactly the shape you need for paginated API
calls or reading a large file in chunks.

```javascript
function delay(ms, value) {
  return new Promise((resolve) => setTimeout(() => resolve(value), ms));
}

async function* fetchPages(totalPages) {
  for (let page = 1; page <= totalPages; page += 1) {
    const data = await delay(15, `page-${page}-data`); // simulate a network call
    yield data;
  }
}

async function loadAll() {
  const collected = [];
  for await (const page of fetchPages(3)) {
    collected.push(page);
  }
  console.log(collected); // [ 'page-1-data', 'page-2-data', 'page-3-data' ]
}

loadAll();
```

## Generator-based async flows (the idea behind async/await)

Before `async`/`await` existed, libraries drove plain generators to express
async control flow: a generator `yield`s a promise, a runner function awaits
it and resumes the generator with the result. Understanding this "runner"
pattern demystifies what `async`/`await` compiles down to conceptually.

```javascript
function delay(ms, value) {
  return new Promise((resolve) => setTimeout(() => resolve(value), ms));
}

function* workflow() {
  const user = yield delay(20, { id: 1, name: "Ada" });
  const orders = yield delay(20, [`order-${user.id}`]);
  return { user, orders };
}

// A minimal runner that drives any generator of this shape to completion.
function run(generatorFn) {
  const iterator = generatorFn();

  function step(input) {
    const { value, done } = iterator.next(input);
    if (done) return Promise.resolve(value);
    return Promise.resolve(value).then(step); // await the yielded promise, then resume
  }

  return step();
}

run(workflow).then(console.log);
// { user: { id: 1, name: 'Ada' }, orders: [ 'order-1' ] }
```

## Exercise

Write an async function `pingAll(urls, timeoutMs)` that takes an array of
"URLs" (simulate each with a `delay()` helper that resolves or randomly
rejects) and returns an array of `{ url, status: "ok" | "failed" | "timeout" }`
objects — one per URL — using `Promise.allSettled` combined with a
per-request timeout built from `Promise.race`. No URL's failure should stop
the others from reporting.
