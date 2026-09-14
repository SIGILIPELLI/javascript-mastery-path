# 08 · Error Handling Basics

## 🎥 Video walkthrough

<iframe width="100%" height="400" style="max-width:720px;aspect-ratio:16/9;height:auto;" src="https://www.youtube.com/embed/h6oo5a2G4fE" title="Video walkthrough" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

## try / catch

```javascript
try {
  const result = JSON.parse("not valid json");
} catch (error) {
  console.log("parsing failed:", error.message);
}
```

## Catching specific error types

JavaScript doesn't have typed `catch` clauses like some languages — you check
`error` inside the block. Always inspect what you're catching; a bare `catch`
that silently swallows everything hides bugs.

```javascript
function safeParseInt(value) {
  const parsed = Number(value);
  if (Number.isNaN(parsed)) {
    console.log(`'${value}' is not a valid number`);
    return null;
  }
  return parsed;
}

safeParseInt("42");  // 42
safeParseInt("abc"); // prints message, returns null
```

## finally

```javascript
try {
  const number = Number("42");
  if (Number.isNaN(number)) throw new Error("conversion failed");
  console.log(`conversion succeeded: ${number}`);
} catch (error) {
  console.log(error.message);
} finally {
  console.log("this always runs"); // cleanup, runs no matter what
}
```

## Raising your own errors

```javascript
function withdraw(balance, amount) {
  if (amount > balance) {
    throw new Error("insufficient funds");
  }
  return balance - amount;
}

try {
  withdraw(100, 150);
} catch (error) {
  console.log(error.message); // insufficient funds
}
```

## The Error object

```javascript
try {
  throw new Error("something broke");
} catch (error) {
  console.log(error.name);    // Error
  console.log(error.message); // something broke
  console.log(error.stack);   // stack trace — useful for debugging
}
```

## Errors in asynchronous code (a preview)

`try`/`catch` only catches errors thrown synchronously in the same call
stack — errors from callbacks or unhandled promise rejections need different
handling, covered fully in
[Level 2 · Module 7](../level-2/07-error-handling-advanced.md).

```javascript
try {
  setTimeout(() => {
    throw new Error("too late for this try/catch to see me");
  }, 0);
} catch (error) {
  console.log("this never runs");
}
```

## Common built-in error types

| Error type | When it happens |
|------------|------------------|
| `SyntaxError` | invalid code, e.g. `JSON.parse("{bad")` |
| `TypeError` | wrong type entirely, e.g. calling something that isn't a function |
| `ReferenceError` | using a variable that doesn't exist |
| `RangeError` | a value is outside an allowed range, e.g. invalid array length |

## How It Actually Works

`throw` doesn't return a value — it unwinds the call stack by walking upward from the
current stack frame looking for the nearest enclosing `try` block whose code region
covers the current instruction pointer. V8 tracks this via a per-function table of
"protected regions" generated at compile time; when an exception is thrown, the engine
consults that table frame by frame, popping stack frames (running any `finally` blocks
it finds along the way) until it either finds a matching `catch` or reaches the top of
the stack, at which point the error becomes an uncaught exception reported to the host
(Node prints it and exits; browsers fire a `window.onerror` event).

The `Error` object's `.stack` string is itself lazily computed: when you write `new
Error()`, V8 doesn't format the whole string immediately — it captures a lightweight
array of stack frame pointers, and only serializes them into the human-readable
`"at functionName (file:line:col)"` text the first time `.stack` is actually read. This
is a deliberate performance optimization (`Error.captureStackTrace` exists specifically
to control this), because constructing full formatted stack traces for errors that are
caught and ignored would be wasted work. `finally` blocks are guaranteed to run even if
the `try` or `catch` returns or re-throws, because the engine's unwind logic treats
"execute finally" as a mandatory step of leaving the protected region, regardless of
how control flow is trying to leave it.
## Exercise

Write a function `divideSafely(a, b)` that returns the division result, or a
descriptive error message string if `b` is zero or either argument isn't a
number — without crashing the program either way.
