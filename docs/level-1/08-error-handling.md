# 08 · Error Handling Basics

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

## Exercise

Write a function `divideSafely(a, b)` that returns the division result, or a
descriptive error message string if `b` is zero or either argument isn't a
number — without crashing the program either way.
