# 09 · Build Tooling Intro

## Why build tools exist

Modern JavaScript is usually written as many small modules, sometimes with
syntax browsers don't natively support (JSX, TypeScript). Build tools take
that source code and produce optimized files a browser can actually load
quickly — bundling, transpiling, and minifying along the way.

## npm scripts

`package.json`'s `scripts` field turns arbitrary shell commands into short,
memorable names — the same interface regardless of what's running
underneath.

```json
{
  "name": "my-app",
  "scripts": {
    "start": "node server.js",
    "dev": "vite",
    "build": "vite build",
    "test": "jest",
    "lint": "eslint ."
  }
}
```

```bash
npm run dev     # starts the Vite dev server
npm run build   # produces an optimized production build
npm test        # "test" is special — runs without needing "run"
npm start       # "start" is also special — same shortcut
```

## Chaining and composing scripts

Scripts can call other scripts, letting you build up a pipeline instead of
one giant command.

```json
{
  "scripts": {
    "lint": "eslint .",
    "test": "jest",
    "build": "vite build",
    "verify": "npm run lint && npm run test && npm run build"
  }
}
```

```bash
npm run verify
# runs lint, then test, then build — stops at the first failure (&&)
```

## What a bundler actually does

A bundler starts from an entry file, follows every `import`/`require` it
finds, and combines everything into a small number of output files —
resolving the dependency graph so the browser doesn't need to make dozens of
separate requests.

```javascript
// math.js
export function add(a, b) {
  return a + b;
}
```

```javascript
// main.js
import { add } from "./math.js";
console.log(add(2, 3));
```

A bundler turns these two files into a single output file (roughly):

```javascript
// dist/main.bundled.js (simplified illustration of the output)
function add(a, b) {
  return a + b;
}
console.log(add(2, 3));
```

Along the way, bundlers typically also **minify** (strip whitespace/comments
and shorten variable names) and **tree-shake** (remove exported code that's
never actually imported anywhere).

## Vite — the fast, modern default

Vite serves source files directly during development using native browser
ES modules (near-instant startup, no bundling needed), then switches to a
production bundler (Rollup) only for the final build.

```bash
npm create vite@latest my-app -- --template vanilla
cd my-app
npm install
npm run dev    # instant dev server with hot reload
npm run build  # bundled, minified output in dist/
```

## webpack — the established, highly configurable option

webpack bundles for both development and production using a config file
that describes entry points, output location, and "loaders" that teach it
how to handle non-JS file types (CSS, images, TypeScript, etc.).

```javascript
// webpack.config.js
const path = require("path");

module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
  module: {
    rules: [
      { test: /\.css$/, use: ["style-loader", "css-loader"] },
    ],
  },
  mode: "production", // enables minification and optimizations
};
```

```bash
npm install --save-dev webpack webpack-cli
npx webpack
```

## Vite vs. webpack at a glance

| Aspect | Vite | webpack |
|--------|------|---------|
| Dev server startup | near-instant (native ES modules) | slower (bundles everything upfront) |
| Configuration | minimal, sensible defaults | explicit, highly configurable |
| Production build | uses Rollup under the hood | uses its own bundler |
| Ecosystem maturity | newer, rapidly adopted | very mature, huge plugin ecosystem |
| Best for | most new projects | projects needing deep, custom control |

## Environment-specific config and dev vs. prod

Both tools distinguish a development build (fast rebuilds, readable output,
source maps) from a production build (minified, optimized, no dev-only
code) — always check which mode you're running before debugging "why is
this so slow/large."

```bash
# Vite automatically sets import.meta.env.MODE
npm run dev   # "development"
npm run build # "production"
```

```javascript
if (import.meta.env.MODE === "development") {
  console.log("verbose debug logging enabled");
}
```

## Exercise

Initialize a new project with `npm init -y`, add `"dev"`, `"build"`, and
`"test"` scripts to `package.json` (they can point to placeholder commands
like `echo "building..."` if you don't have Vite/webpack installed), and add
a `"verify"` script that chains `lint`, `test`, and `build` together with
`&&`. Run `npm run verify` and confirm it executes each step in order.
