---
description: "Performance & Memory — The first rule of performance work: measure, don't guess. console.time gives quick, rough timings for comparing approaches."
---

# 08 · Performance & Memory

## 🎥 Video walkthrough

<iframe width="100%" height="400" style="max-width:720px;aspect-ratio:16/9;height:auto;" src="https://www.youtube.com/embed/g9WwMlntARk" title="Video walkthrough" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Correct code isn't automatically fast or memory-safe code. This module
covers profiling to find real bottlenecks (instead of guessing), common
memory-leak patterns in long-running Node processes, and general
performance best practices.

## Measuring before optimizing

The first rule of performance work: **measure, don't guess**. `console.time`
gives quick, rough timings for comparing approaches.

```javascript
function sumWithLoop(n) {
  let total = 0;
  for (let i = 0; i < n; i += 1) {
    total += i;
  }
  return total;
}

function sumWithReduce(n) {
  return Array.from({ length: n }, (_, i) => i).reduce((a, b) => a + b, 0);
}

console.time("loop");
sumWithLoop(10_000_000);
console.timeEnd("loop");   // loop: ~5ms

console.time("reduce");
sumWithReduce(10_000_000);
console.timeEnd("reduce"); // reduce: ~250ms — building + reducing an array is much slower
```

## Profiling with Node's built-in profiler

For a real bottleneck (not a 10-line example), Node can produce a CPU
profile you can inspect for hot functions.

```bash
node --prof server.js
# ... exercise the app, then stop it ...
node --prof-process isolate-0x*-v8.log > profile.txt
```

```text
 [Summary]:
   ticks  total  nonlib   name
   4820   62.4%   68.1%  JavaScript
   1830   23.7%   25.9%  C++
    950   12.3%          GC

 [Bottom up (heavy) profile]:
   ticks parent  name
   2103   43.6%  parseAndValidate (server.js:42)
   ...
```

`GC` (garbage collection) time appearing high in the summary is itself a
signal — it often means too many short-lived objects are being allocated,
which points back to a memory-usage problem rather than a raw CPU one.

## The `--inspect` flag and Chrome DevTools

For interactive profiling (flame charts, heap snapshots), run Node with
`--inspect` and attach Chrome DevTools.

```bash
node --inspect server.js
# Debugger listening on ws://127.0.0.1:9229/...
# Open chrome://inspect in Chrome, click "inspect" under Remote Target
```

From DevTools you can record a **CPU profile** (flame chart of where time
goes) or take a **heap snapshot** (a picture of every object currently
allocated, useful for finding what's holding memory).

## Common memory leak patterns in Node

A memory leak happens when references to objects are kept alive
unintentionally, so the garbage collector can never reclaim them — memory
usage grows over the life of the process until it crashes.

```javascript
// LEAK: an ever-growing array with no eviction
const requestLog = [];

function handleRequest(req) {
  requestLog.push(req); // never removed — grows forever as traffic comes in
}
```

```javascript
// FIX: bound the size, or use a rolling window
const MAX_LOG_SIZE = 1000;
const requestLog = [];

function handleRequest(req) {
  requestLog.push(req);
  if (requestLog.length > MAX_LOG_SIZE) {
    requestLog.shift(); // evict the oldest entry to keep memory bounded
  }
}
```

```javascript
// LEAK: forgetting to remove event listeners that are no longer needed
import { EventEmitter } from "node:events";

const bus = new EventEmitter();

function attachPerRequestListener(bus) {
  bus.on("data", (chunk) => console.log(chunk)); // a new listener added on every call!
}
```

```javascript
// FIX: remove listeners when they're no longer needed, or use `.once`
function attachOneTimeListener(bus) {
  bus.once("data", (chunk) => console.log(chunk)); // fires once, then auto-removes itself
}

function attachAndCleanUp(bus) {
  function handler(chunk) {
    console.log(chunk);
  }
  bus.on("data", handler);
  return () => bus.off("data", handler); // caller invokes this to clean up
}
```

```javascript
// LEAK: closures accidentally capturing large objects
function makeHandler(hugeBuffer) {
  return function handler(req, res) {
    res.send("ok"); // hugeBuffer is never used here...
  };
  // ...but it's still captured in the closure and kept alive for as long
  // as `handler` exists, because closures keep their entire enclosing scope.
}
```

```javascript
// FIX: only capture what you need
function makeHandler(hugeBuffer) {
  const summary = hugeBuffer.length; // extract just the needed value
  return function handler(req, res) {
    res.send(`ok (${summary} bytes)`);
  };
  // hugeBuffer itself can now be garbage collected once makeHandler returns
}
```

## Detecting a leak: watch memory over time

```javascript
setInterval(() => {
  const usage = process.memoryUsage();
  console.log(`heapUsed: ${(usage.heapUsed / 1024 / 1024).toFixed(1)} MB`);
}, 5000);

// heapUsed: 42.3 MB
// heapUsed: 44.1 MB
// heapUsed: 61.7 MB   <- climbing steadily under steady traffic is a red flag
// heapUsed: 89.4 MB
```

A healthy process's heap usage should plateau (rise, then fall back after
garbage collection) under steady load. Continual, unbounded growth is the
classic signature of a leak.

## Performance best practices

| Practice | Why |
|----------|-----|
| Avoid unnecessary object/array allocation in hot loops | fewer allocations = less GC pressure |
| Use streams for large files/responses | keeps memory flat regardless of data size |
| Paginate database queries | avoids loading huge result sets into memory at once |
| Cache expensive, repeatable computations | trades memory for CPU when appropriate |
| Debounce/throttle high-frequency events | limits how often expensive work runs |
| Prefer `Map`/`Set` over objects/arrays for large lookup-heavy data | O(1) lookups, no prototype chain overhead |
| Profile before optimizing | fixes the actual bottleneck instead of a guessed one |

## Debouncing an expensive operation

```javascript
function debounce(fn, delayMs) {
  let timeoutId;
  return function debounced(...args) {
    clearTimeout(timeoutId); // cancel any pending call
    timeoutId = setTimeout(() => fn(...args), delayMs);
  };
}

function expensiveSearch(query) {
  console.log(`searching for "${query}"...`);
}

const debouncedSearch = debounce(expensiveSearch, 300);

debouncedSearch("a");
debouncedSearch("ap");
debouncedSearch("app"); // only this call actually runs expensiveSearch, ~300ms later
```

## How It Actually Works

V8's garbage collector is **generational**: it splits the heap into a small
"young generation" (further split into a tiny nursery and an intermediate space) and a
much larger "old generation." Nearly all objects die young — short-lived intermediate
values, one-off closures, temporary arrays — so V8 optimizes for that case with
**Scavenge**, a fast copying collector that runs frequently on just the young
generation and copies surviving objects out to a survivor space; objects that survive
two scavenges get **promoted** to the old generation. The old generation is collected
far less often, using a slower mark-sweep-compact algorithm (**Mark-Compact**), because
scanning and compacting a large heap is expensive and old objects are statistically
much less likely to have become garbage.

Memory leaks in long-running Node processes almost always trace back to something
holding a reference the developer thinks is temporary: an ever-growing array pushed to
but never trimmed, event listeners attached repeatedly without ever being removed (each
one keeps its closure's entire captured environment alive), or a `Map`/`Set` used as a
cache with no eviction policy. `WeakMap`/`WeakSet` exist specifically to break this
class of leak — their keys don't count as strong references for garbage collection
purposes, so an entry disappears on its own once nothing else references the key,
without needing manual cleanup code at all.
## Exercise

Write a `LRUCache` class (least-recently-used cache) backed by a `Map` with
a fixed `maxSize`: `get(key)` returns the value and marks it recently used;
`set(key, value)` adds/updates an entry and evicts the least-recently-used
entry when the cache exceeds `maxSize`. Then write a small script that adds
more entries than `maxSize` allows and logs the final contents, confirming
the oldest unused entries were evicted rather than the cache growing
unbounded.
