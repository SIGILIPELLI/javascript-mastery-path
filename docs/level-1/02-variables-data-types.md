---
description: "Variables, Data Types & Operators — Prefer const by default, and let when you know the value needs to change. Avoid var — its function-level (rather than…"
---

# 02 · Variables, Data Types & Operators

## Declaring variables: let, const, var

```javascript
let age = 25;          // block-scoped, reassignable
const name = "Ada";    // block-scoped, cannot be reassigned
var height = 1.68;     // function-scoped — legacy, avoid in new code
```

Prefer `const` by default, and `let` when you know the value needs to change.
Avoid `var` — its function-level (rather than block-level) scoping is a common
source of bugs, covered more in [Module 4](04-functions-scope.md).

```javascript
const isStudent = false;
age = 26;        // fine — let allows reassignment
// name = "Grace"; // TypeError: Assignment to constant variable.
```

## Primitive data types

```javascript
const wholeNumber = 42;        // number (JS has one numeric type)
const piIsh = 3.14159;         // also number
const message = "hi there";    // string
const flag = true;             // boolean
const nothing = null;          // intentional "no value"
let notAssignedYet;            // undefined — declared but not assigned
const big = 9007199254740993n; // BigInt, for integers beyond Number.MAX_SAFE_INTEGER

console.log(typeof wholeNumber); // "number"
console.log(typeof message);      // "string"
console.log(typeof nothing);      // "object" — a famous long-standing JS quirk
console.log(typeof notAssignedYet); // "undefined"
```

## Numeric operators

```javascript
const a = 7;
const b = 2;

console.log(a + b);  // 9   addition
console.log(a - b);  // 5   subtraction
console.log(a * b);  // 14  multiplication
console.log(a / b);  // 3.5 division (always float-capable)
console.log(a % b);  // 1   modulo (remainder)
console.log(a ** b); // 49  exponentiation
console.log(Math.floor(a / b)); // 3   floor division equivalent
```

## Comparison & logical operators

```javascript
console.log(5 > 3);        // true
console.log(5 == "5");     // true  — loose equality, coerces types
console.log(5 === "5");    // false — strict equality, checks type too (prefer this)
console.log(5 !== "5");    // true

console.log(true && false); // false
console.log(true || false); // true
console.log(!true);         // false
```

Always prefer `===` and `!==` over `==` and `!=` — loose equality's coercion
rules are a frequent source of bugs.

## Type conversion

```javascript
String(42);        // "42"
Number("42");       // 42
Number("3.5");      // 3.5
parseInt("42px");   // 42 — parses leading digits, ignores the rest
Boolean(0);         // false
Boolean("");        // false
Boolean("x");        // true — any non-empty string is truthy
Boolean(null);       // false
Boolean(undefined);  // false
```

## Truthy and falsy values

```javascript
// Falsy: false, 0, "", null, undefined, NaN — everything else is truthy
if ("hello") {
  console.log("strings with content are truthy");
}
```

## Naming rules & convention

- Names use `camelCase` by convention (`userAge`, not `UserAge` or `user_age`).
- Must start with a letter, `_`, or `$` — can't start with a digit.
- Constants that never change are often written `UPPER_SNAKE_CASE`
  (`const MAX_RETRIES = 3;`) by convention, though `const` alone doesn't imply
  this style.

| Keyword | Reassignable? | Scope |
|---------|----------------|-------|
| `const` | No | block |
| `let` | Yes | block |
| `var` | Yes | function (avoid) |

## How It Actually Works

`let` and `const` aren't just "block-scoped var" — they exist because of the **Temporal
Dead Zone (TDZ)**. Every `let`/`const` binding is hoisted to the top of its block during
compilation (V8 allocates the binding slot up front), but it stays uninitialized until
the declaration line actually executes. Reading it before that throws
`ReferenceError: Cannot access 'x' before initialization` — not because the variable
doesn't exist yet, but because the engine is deliberately blocking access to a slot it
already knows about. `var`, by contrast, is hoisted *and* initialized to `undefined`
immediately, which is why `console.log(x); var x = 1;` silently prints `undefined`
instead of throwing.

At the representation level, V8 doesn't store every value the same way. Small integers
get packed into a tagged **Smi** (small integer) representation directly in the pointer
word — no heap allocation at all. Once a number needs a fraction or exceeds the Smi
range, V8 boxes it as a `HeapNumber`. Objects get a **hidden class** (also called a
"map") the moment they're created, which records the order and types of their
properties; two objects created with properties added in the same order share a hidden
class and can reuse the same optimized machine code for property access. This is a
big part of why `{a:1, b:2}` is faster to work with than an object you build by adding
properties in unpredictable order later.
## 🔀 See this in another language

- [TypeScript — Basic Types](https://sigilipelli.github.io/typescript-mastery-path/level-1/02-basic-types/)
- [Ruby — Variables & Types](https://sigilipelli.github.io/ruby-mastery-path/level-1/02-variables-types/)
- [R — Variables & Types](https://sigilipelli.github.io/r-mastery-path/level-1/02-variables-types/)

## Exercise

Write a script that stores a rectangle's `width` and `height`, computes its
area and perimeter, and prints both formatted to 2 decimal places using
`.toFixed(2)`.
