# 04 · Functions & Scope

## Function declarations

```javascript
function add(a, b) {
  return a + b;
}

function greet(name = "friend") { // default parameter
  return `Hello, ${name}!`;
}

console.log(add(2, 3));   // 5
console.log(greet());      // Hello, friend!
console.log(greet("Ada")); // Hello, Ada!
```

## Function expressions and arrow functions

```javascript
// Function expression: an unnamed function assigned to a variable
const multiply = function (a, b) {
  return a * b;
};

// Arrow function: shorter syntax, does NOT have its own `this`
const subtract = (a, b) => a - b;

// Single parameter: parens optional
const square = (x) => x * x;

// No parameters: empty parens required
const sayHi = () => "hi";

console.log(multiply(3, 4)); // 12
console.log(subtract(10, 4)); // 6
console.log(square(5));       // 25
```

## Rest parameters and the arguments object

```javascript
function total(...numbers) { // rest parameter — gathers args into a real array
  return numbers.reduce((sum, n) => sum + n, 0);
}

function makeProfile(name, ...rest) {
  return { name, extra: rest };
}

console.log(total(1, 2, 3, 4));                 // 10
console.log(makeProfile("Ada", "London", 30));
// { name: 'Ada', extra: [ 'London', 30 ] }
```

## Hoisting

```javascript
console.log(hoisted()); // works — function declarations are fully hoisted
function hoisted() {
  return "I run before my own line in the file";
}

console.log(typeof notYetDefined); // "undefined" — var is hoisted, but not its value
var notYetDefined = 5;

// console.log(letVar); // ReferenceError — let/const are hoisted but not initialized
let letVar = 10;
```

Function declarations are hoisted with their full body; `var` is hoisted but
only the declaration (not the assignment); `let`/`const` are hoisted into a
"temporal dead zone" where referencing them before their declaration throws.

## Scope: block vs. function

```javascript
function demo() {
  if (true) {
    let blockScoped = "only visible inside this block";
    var functionScoped = "visible throughout demo()";
  }
  // console.log(blockScoped);    // ReferenceError
  console.log(functionScoped);    // works — var ignores block boundaries
}

demo();
```

```javascript
let x = "global";

function outer() {
  let x = "enclosing";

  function inner() {
    let x = "local";
    console.log(x); // local
  }

  inner();
  console.log(x); // enclosing
}

outer();
console.log(x); // global
```

Each nested function can read variables from its enclosing scopes (this is
the basis for closures, covered in depth in
[Level 2 · Module 1](../level-2/01-closures-scope.md)).

## Functions are values

```javascript
function squareFn(x) {
  return x * x;
}

const operations = { square: squareFn };
console.log(operations.square(5)); // 25

// Higher-order function: takes a function as an argument
function applyTwice(fn, value) {
  return fn(fn(value));
}

console.log(applyTwice(squareFn, 3)); // 81
```

## Exercise

Write a function `summarize(...values)` that returns an object with the `min`,
`max`, and `average` of `values`, rounding the average to 2 decimal places with
`.toFixed(2)`.
