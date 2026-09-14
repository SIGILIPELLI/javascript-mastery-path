# 09 · Modules & npm Basics

## 🎥 Video walkthrough

<iframe width="100%" height="400" style="max-width:720px;aspect-ratio:16/9;height:auto;" src="https://www.youtube.com/embed/yJyHhN0gS9s" title="Video walkthrough" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

## ES modules — export

`shapes.js`:

```javascript
// shapes.js
export function areaCircle(radius) {
  return 3.14159 * radius ** 2;
}

export function areaSquare(side) {
  return side ** 2;
}

export const PI = 3.14159; // named export

export default function areaTriangle(base, height) { // default export — one per file
  return 0.5 * base * height;
}
```

## ES modules — import

`main.js`, in the same folder:

```javascript
import areaTriangle, { areaCircle, areaSquare, PI } from "./shapes.js";

console.log(areaCircle(2));
console.log(areaSquare(3));
console.log(areaTriangle(4, 5));
console.log(PI);

// Import everything under a namespace:
import * as shapes from "./shapes.js";
console.log(shapes.areaCircle(2));
```

## Running ES modules in Node

Node treats `.js` files as CommonJS by default. To use `import`/`export`
directly, either name files `.mjs` or add `"type": "module"` to
`package.json` (shown below).

```bash
node main.js
```

## CommonJS (the older Node module system)

Many existing Node codebases and npm packages still use `require`/
`module.exports` — you'll encounter both systems.

```javascript
// shapes.cjs
function areaCircle(radius) {
  return 3.14159 * radius ** 2;
}

module.exports = { areaCircle };
```

```javascript
// main.cjs
const { areaCircle } = require("./shapes.cjs");
console.log(areaCircle(2));
```

## package.json — describing a project

```bash
npm init -y
```

```json
{
  "name": "myapp",
  "version": "1.0.0",
  "type": "module",
  "main": "main.js",
  "scripts": {
    "start": "node main.js"
  },
  "dependencies": {}
}
```

`"type": "module"` tells Node to treat `.js` files as ES modules, enabling
top-level `import`/`export` without renaming files to `.mjs`.

## Installing third-party packages with npm

```bash
npm install axios
```

```javascript
import axios from "axios";

const response = await axios.get("https://api.github.com");
console.log(response.status); // 200
```

`npm install` adds the package to `node_modules/` and records it in
`package.json`'s `dependencies` — `package-lock.json` pins exact versions so
installs are reproducible across machines.

## npm scripts

```bash
npm run start   # runs the "start" script defined in package.json
npm test         # shorthand for the "test" script
```

## Modules cheat sheet

| Task | ES modules | CommonJS |
|------|------------|----------|
| Export one thing | `export default value` | `module.exports = value` |
| Export several things | `export { a, b }` | `module.exports = { a, b }` |
| Import | `import x from "./file.js"` | `const x = require("./file")` |
| File extension needed? | Yes, in Node | No |

## How It Actually Works

ES modules and CommonJS modules aren't just different syntax — they load and execute
differently. CommonJS (`require`) is **synchronous**: calling `require('./a')` blocks
right there, reads the file, wraps it in a function `(module, exports, require,
__filename, __dirname) => {...}`, executes it top to bottom, and returns
`module.exports`. Node caches the result keyed by resolved file path, so a second
`require` of the same file returns the same object instantly without re-executing it —
this is also why circular `require`s can hand back a *partially filled* `exports`
object if module A requires B while B is still mid-execution requiring A back.

ES modules (`import`/`export`) work in two distinct phases enforced by the engine
itself: first a **linking** phase, where the module graph is parsed and every `export`
binding is connected to every `import` that references it — as *live bindings*, not
copied values, which is why `import { count } from './counter.js'` reflects later
changes to `count` inside that module. Only after the entire graph is linked does
**evaluation** run, top to bottom, per module, exactly once. This static, ahead-of-time
linking is what lets bundlers tree-shake unused exports — the import graph is fully
known before any code runs, unlike CommonJS where `require` calls can be conditional
and dynamic, making static analysis far harder.
## Exercise

Split a script that manages a to-do list into two ES modules: `storage.js`
(load/save the list to a JSON file using Node's `fs` module) and `main.js`
(the logic that imports `storage.js`). This sets up the project for
Module 10.
