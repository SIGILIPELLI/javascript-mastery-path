---
title: "JavaScript Cheat Sheet: Quick Reference, Beginner to Advanced"
description: "A free JavaScript cheat sheet: 15 quick-reference tables from every level of the JavaScript Mastery Path, each linked to its full lesson."
---

# JavaScript Cheat Sheet

Quick-reference tables from **15 lessons** of the free JavaScript Mastery Path, in course order from beginner to advanced. Each section links to the full lesson with explanations and runnable examples.

## Level 1 · Entry

### [06 · Strings &amp; Template Literals](level-1/06-strings-template-literals.md)

| Need | Syntax |
|------|--------|
| Insert a variable | `` `${value}` `` |
| Multi-line string | `` `line1\nline2` `` or a literal newline inside backticks |
| Format a number | `value.toFixed(2)`, `value.toLocaleString()` |
| Pad a string | `str.padStart(n)`, `str.padEnd(n)` |

### [09 · Modules &amp; npm Basics](level-1/09-modules-npm.md)

| Task | ES modules | CommonJS |
|------|------------|----------|
| Export one thing | `export default value` | `module.exports = value` |
| Export several things | `export { a, b }` | `module.exports = { a, b }` |
| Import | `import x from "./file.js"` | `const x = require("./file")` |
| File extension needed? | Yes, in Node | No |

## Level 2 · Intermediate

### [01 · Closures &amp; Scope Deep Dive](level-2/01-closures-scope.md)

| Concept | What it means |
|---------|----------------|
| Lexical scope | Where a variable is accessible, decided by source-code position |
| Block scope | `let`/`const` are confined to the nearest `{ }` |
| Function scope | `var` is confined to the nearest function, ignoring blocks |
| Closure | A function plus the scope it was created in, kept alive after the outer function returns |
| Module pattern | Using a closure (often an IIFE) to expose a public API while hiding private state |

### [03 · Asynchronous JavaScript I](level-2/03-async-javascript-i.md)

| Source | Queue | Runs relative to sync code | Runs relative to other queue |
|--------|-------|------------------------------|-------------------------------|
| `console.log`, loops, direct calls | call stack | immediately | — |
| `Promise.then`/`catch`/`finally`, `async` continuations | microtask queue | after current sync code finishes | before any pending macrotask |
| `setTimeout`, `setInterval`, DOM events | task (macro) queue | after current sync code finishes | after all pending microtasks drain |

### [04 · Promises &amp; Async/Await](level-2/04-promises-async-await.md)

| Method | Resolves when | Rejects when |
|--------|----------------|----------------|
| `Promise.all` | every promise fulfills | any single promise rejects (fails fast) |
| `Promise.allSettled` | every promise settles (fulfilled or rejected) | never — always fulfills with a status report |
| `Promise.race` | the first promise to settle, settles | the first promise to settle rejects |
| `Promise.any` | the first promise to fulfill, fulfills | only if *all* promises reject |

### [05 · Working with JSON &amp; Fetch API](level-2/05-json-fetch.md)

| Task | Code |
|------|------|
| GET and parse JSON | `const data = await (await fetch(url)).json();` |
| Check for HTTP errors | `if (!response.ok) throw new Error(response.status);` |
| POST JSON | `fetch(url, { method: "POST", headers: {...}, body: JSON.stringify(data) })` |
| Parse JSON string | `JSON.parse(text)` |
| Serialize to JSON string | `JSON.stringify(value, null, 2)` |

### [06 · Regular Expressions in JS](level-2/06-regular-expressions.md)

| Method | Called on | Returns | Best for |
|--------|-----------|---------|----------|
| `regex.test(str)` | RegExp | boolean | quick yes/no validation |
| `regex.exec(str)` | RegExp | match array or `null`, stateful with `g` | stepping through matches one at a time |
| `str.match(regex)` | String | first match, or all matches (with `g`) | extracting matched text |
| `str.matchAll(regex)` | String | iterator of full match objects (requires `g`) | extracting matches with capture groups |
| `str.replace(regex, ...)` | String | new string | substituting or reformatting matched text |

### [07 · Error Handling Advanced](level-2/07-error-handling-advanced.md)

| Situation | Strategy |
|-----------|----------|
| Expected, recoverable failure (bad input) | throw a specific `Error` subclass, `catch` and handle it |
| Unexpected failure inside `async` code | `try`/`catch` around `await`, or `.catch()` on the promise chain |
| Failure crossing a module boundary | wrap with `{ cause: originalError }` for traceability |
| Promise created but never awaited/caught | always attach `.catch()`, or listen for `unhandledrejection`/`unhandledRejection` as a safety net |
| Truly unknown/corrupted state | log and crash/restart (`uncaughtException`) rather than continue silently |

### [08 · Testing Basics with Jest](level-2/08-testing-jest.md)

| Matcher | Use for |
|---------|---------|
| `.toBe(value)` | primitives, strict `===` equality |
| `.toEqual(value)` | deep equality on objects/arrays |
| `.toContain(item)` | array or string membership |
| `.toThrow(messageOrPattern)` | a function throws synchronously |
| `.rejects.toThrow(...)` | a promise rejects (with `await expect(...)`) |
| `.toHaveBeenCalledWith(...)` | a `jest.fn()` mock's call arguments |
| `.toBeNull()` / `.toBeUndefined()` | absence checks |

## Level 3 · Advanced

### [01 · Advanced Async Patterns](level-3/01-advanced-async-patterns.md)

| Method | Settles when | Rejects when |
|--------|---------------|--------------|
| `Promise.all` | all fulfill | any one rejects (immediately) |
| `Promise.allSettled` | all settle (fulfilled or rejected) | never — always fulfills |
| `Promise.race` | the first one settles (either way) | first settled one is a rejection |
| `Promise.any` | the first one fulfills | all reject (`AggregateError`) |

### [02 · Node.js Fundamentals &amp; REST APIs with Express](level-3/02-nodejs-express-rest-api.md)

*Part 2 · Building a REST API with Express*

| Code | Meaning | Typical use |
|------|---------|--------------|
| 200 | OK | successful GET/PUT |
| 201 | Created | successful POST that creates a resource |
| 204 | No Content | successful DELETE, nothing to return |
| 400 | Bad Request | validation failure |
| 401 | Unauthorized | missing/invalid credentials |
| 404 | Not Found | resource doesn't exist |
| 500 | Internal Server Error | unhandled exception |

### [04 · Design Patterns in JavaScript](level-3/04-design-patterns.md)

| Pattern | Problem it solves | JS building block |
|---------|--------------------|--------------------|
| Module | hide internal state, expose a small API | ES modules / closures |
| Singleton | exactly one shared instance | cached module export / instance check |
| Factory | centralize object creation logic | a function returning different objects/classes |
| Observer | decouple "something happened" from "what to do about it" | listener arrays / `EventEmitter` |

### [08 · Security Basics](level-3/08-security-basics.md)

| Threat | Defense |
|--------|---------|
| XSS | escape/sanitize output, use `textContent`, rely on templating auto-escaping |
| CSRF | CSRF tokens, `SameSite` cookies, verify `Origin`/`Referer` |
| SQL injection | parameterized queries / prepared statements, never string-concatenate SQL |
| Malformed/malicious input | validate shape and type at the boundary (schema libraries like `zod`) |
| Vulnerable dependencies | `npm audit`, keep packages updated, minimize dependency count |

### [09 · Build &amp; Bundling Deep Dive](level-3/09-build-bundling.md)

| Technique | What it does | Enabled by |
|-----------|----------------|------------|
| Code splitting | defer loading unused code until needed | dynamic `import()` |
| Tree-shaking | remove exports that are never imported | static ES module `import`/`export` |
| Minification | shrink code size (short names, no whitespace) | production build mode |
| Content-hash filenames | safe long-term browser caching | `[contenthash]` in output filename |

## Level 4 · Master

### [02 · Advanced TypeScript](level-4/02-advanced-typescript.md)

| Utility | Purpose |
|---|---|
| `Partial<T>` | Every field becomes optional |
| `Required<T>` | Every field becomes mandatory |
| `Pick<T, K>` | Keep only keys `K` |
| `Omit<T, K>` | Remove keys `K` |
| `Record<K, V>` | Object type with keys `K` mapped to values `V` |
| `ReturnType<Fn>` | The return type of a function type |
| `Awaited<T>` | The resolved value type of a Promise |
| `NonNullable<T>` | Removes `null`/`undefined` from a union |

---

Go deeper with the full [JavaScript Mastery Path](index.md) curriculum or test yourself with the [interview questions](interview-questions.md).
