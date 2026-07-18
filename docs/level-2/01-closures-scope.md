# 01 · Closures & Scope Deep Dive

This module builds directly on lexical scope from
[Level 1 · Module 4](../level-1/04-functions-scope.md) — if function scope and
hoisting aren't solid yet, revisit that page first.

## Lexical scope, revisited

A function's scope is determined by *where it is written in the source*, not
by where it's called from. Every function "remembers" the scope chain that
existed at the moment it was defined.

```javascript
function makeGreeter() {
  const greeting = "Hello"; // lives in makeGreeter's scope

  return function greet(name) { // defined here, so it can see `greeting`
    return `${greeting}, ${name}!`;
  };
}

const greet = makeGreeter();
console.log(greet("Ada")); // Hello, Ada!
```

`greet` still works even though `makeGreeter()` already finished running.
That's the whole idea of a closure.

## What a closure actually is

A closure is the combination of a function and the lexical environment it was
declared in. The inner function keeps a live reference to the outer
variables — not a snapshot — so changes are visible across calls.

```javascript
function makeCounter() {
  let count = 0; // private state, not accessible from outside

  return {
    increment() {
      count += 1;
      return count;
    },
    reset() {
      count = 0;
    },
  };
}

const counter = makeCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
counter.reset();
console.log(counter.increment()); // 1

// `count` cannot be reached directly — no counter.count property exists
console.log(counter.count); // undefined
```

Each call to `makeCounter()` creates a brand-new, independent `count`:

```javascript
const counterA = makeCounter();
const counterB = makeCounter();

counterA.increment();
counterA.increment();
counterB.increment();

console.log(counterA.increment()); // 3
console.log(counterB.increment()); // 2 — completely separate closure
```

## The module pattern

Before ES modules existed, closures were the standard way to create private
state and a public API in the same object — this is still useful inside a
single file or function today.

```javascript
const bankAccount = (function () {
  let balance = 0; // private — only reachable through the returned methods

  function deposit(amount) {
    if (amount <= 0) throw new Error("deposit must be positive");
    balance += amount;
    return balance;
  }

  function withdraw(amount) {
    if (amount > balance) throw new Error("insufficient funds");
    balance -= amount;
    return balance;
  }

  function getBalance() {
    return balance;
  }

  return { deposit, withdraw, getBalance }; // public API
})(); // IIFE — Immediately Invoked Function Expression, runs right away

bankAccount.deposit(100);
bankAccount.withdraw(30);
console.log(bankAccount.getBalance()); // 70
console.log(bankAccount.balance);      // undefined — truly private
```

## The classic loop pitfall

A common bug: capturing a loop variable declared with `var` inside a
callback, expecting each callback to remember "its" value.

```javascript
// Broken: var is function-scoped, so all three callbacks share ONE `i`
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log("var i:", i), 0);
}
// var i: 3
// var i: 3
// var i: 3
```

By the time the callbacks run, the loop has already finished and `i` is `3`
for all of them, because there was only ever one `i` in the whole loop.

```javascript
// Fixed: let creates a fresh binding for each iteration
for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log("let j:", j), 0);
}
// let j: 0
// let j: 1
// let j: 2
```

If you're stuck with `var` (e.g. maintaining older code), wrap the body in an
IIFE to force a new scope per iteration:

```javascript
for (var k = 0; k < 3; k++) {
  (function (capturedK) {
    setTimeout(() => console.log("captured k:", capturedK), 0);
  })(k); // pass the current value in explicitly
}
// captured k: 0
// captured k: 1
// captured k: 2
```

## Closures over mutable state — a caution

Closures capture *variables*, not values. If several functions share a
closure and one mutates the variable, every function sees the new value —
which is often exactly what you want, but can also cause subtle bugs.

```javascript
function createSharedState() {
  let value = "initial";

  return {
    read: () => value,
    write: (newValue) => {
      value = newValue;
    },
  };
}

const state = createSharedState();
console.log(state.read()); // initial
state.write("changed");
console.log(state.read()); // changed — both functions share the same `value`
```

## Scope vs. closures cheat sheet

| Concept | What it means |
|---------|----------------|
| Lexical scope | Where a variable is accessible, decided by source-code position |
| Block scope | `let`/`const` are confined to the nearest `{ }` |
| Function scope | `var` is confined to the nearest function, ignoring blocks |
| Closure | A function plus the scope it was created in, kept alive after the outer function returns |
| Module pattern | Using a closure (often an IIFE) to expose a public API while hiding private state |

## Exercise

Write a function `createQueue()` that uses a closure to keep a private
array. Return an object with `enqueue(item)` (adds to the end),
`dequeue()` (removes and returns the front item), and `size()` (returns the
current length) — with no way for outside code to access the underlying
array directly.
