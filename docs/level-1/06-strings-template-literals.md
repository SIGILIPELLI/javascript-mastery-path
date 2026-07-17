# 06 · Strings & Template Literals

## String basics

```javascript
const s = "Hello, World!";

console.log(s.toLowerCase());          // hello, world!
console.log(s.toUpperCase());          // HELLO, WORLD!
console.log(s.replace("World", "JS")); // Hello, JS!
console.log(s.split(", "));             // ['Hello', 'World!']
console.log(["a", "b", "c"].join(" ")); // a b c
console.log(s.trim());                  // removes leading/trailing whitespace
console.log(s.length);                  // 13
console.log(s.slice(7, 12));             // World
```

## Template literals (preferred formatting method)

```javascript
const name = "Ada";
const age = 30;
const pi = 3.14159265;

console.log(`${name} is ${age} years old`);
console.log(`pi rounded: ${pi.toFixed(2)}`); // pi rounded: 3.14
console.log(`${age.toString().padStart(5)}`); // right-pad in width 5
console.log(`${"x".repeat(3)}`);              // expressions work inside templates

function debugValue(label, value) {
  return `${label}=${JSON.stringify(value)}`; // there's no built-in `name=` shorthand like Python's f-string debug
}
console.log(debugValue("name", name)); // name="Ada"
```

## Multi-line strings and template literal tags

```javascript
const paragraph = `
This spans
multiple lines.
`;

// Template literals can also hold arbitrary expressions:
const items = ["a", "b", "c"];
console.log(`Items: ${items.map((i) => i.toUpperCase()).join(", ")}`);
```

## Common string checks

```javascript
"42".match(/^\d+$/) !== null; // true — digit check (see Level 2 · Module 6 for regex)
"hello".startsWith("he");     // true
"hello".endsWith("lo");        // true
"   ".trim().length === 0;      // true — whitespace-only check
"Hello, World!".includes("Hello"); // true — substring test
```

## Immutability

Strings can't be modified in place — every "modification" returns a new
string:

```javascript
let str = "hello";
str.toUpperCase();       // returns "HELLO" but doesn't change str
str = str.toUpperCase();  // you must reassign to keep the result
```

## Template literals cheat sheet

| Need | Syntax |
|------|--------|
| Insert a variable | `` `${value}` `` |
| Multi-line string | `` `line1\nline2` `` or a literal newline inside backticks |
| Format a number | `value.toFixed(2)`, `value.toLocaleString()` |
| Pad a string | `str.padStart(n)`, `str.padEnd(n)` |

## Exercise

Write a function `slugify(title)` that converts `"Hello, World!  "` into
`"hello-world"` — lowercase, punctuation stripped, spaces replaced with
hyphens.
