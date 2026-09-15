---
description: "Strings & Template Literals — Strings in V8 are immutable, but 'immutable' doesn't mean 'one representation.' Short strings are stored inline as…"
---

# 06 · Strings & Template Literals

## 🎥 Video walkthrough

<iframe width="100%" height="400" style="max-width:720px;aspect-ratio:16/9;height:auto;" src="https://www.youtube.com/embed/AIwbmZk-FRA" title="Video walkthrough" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

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

## How It Actually Works

Strings in V8 are immutable, but "immutable" doesn't mean "one representation." Short
strings are stored inline as `SeqOneByteString` or `SeqTwoByteString` (depending on
whether every character fits in Latin-1). Concatenating two strings with `+` or
building one with a template literal doesn't necessarily copy all the characters
immediately — V8 can create a **ConsString**, a lightweight object that just points at
the two original strings and remembers their combined length. The actual character
data is only flattened into one contiguous buffer the first time something needs to
read it linearly (like calling `.charAt()` in a loop or passing it to a regex). This is
why building large strings with many small `+=` operations can look cheap per line but
suddenly pay a large one-time flattening cost the first time you inspect the result.

Template literals (`` `${x}` ``) are handled specially by the parser: a tagged template
like `` tag`a${b}c` `` is compiled so the *same* array of string parts (`["a", "c"]`) is
passed to `tag` on every call at that call site — V8 caches and reuses that array
object rather than rebuilding it, which is what lets libraries rely on referential
identity of the strings array across calls to detect "this is the same literal
template, just with different interpolated values."
## 🔀 See this in another language

- [TypeScript — Arrays & Objects, Typed](https://sigilipelli.github.io/typescript-mastery-path/level-1/06-arrays-objects-typed/)
- [Ruby — Strings & String Methods](https://sigilipelli.github.io/ruby-mastery-path/level-1/06-strings/)
- [R — Data Frames Basics](https://sigilipelli.github.io/r-mastery-path/level-1/06-data-frames-basics/)

## Exercise

Write a function `slugify(title)` that converts `"Hello, World!  "` into
`"hello-world"` — lowercase, punctuation stripped, spaces replaced with
hyphens.
