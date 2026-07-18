# 10 · Build & Bundling Deep Dive

Level 2 introduced build tooling at a high level. This module goes deeper
into two widely-used bundlers — Vite and webpack — and the two techniques
that most affect real-world load performance: code splitting and
tree-shaking.

## Why bundle at all?

Bundlers combine many source modules into a small number of optimized
output files, resolve `import`/`require` graphs, transpile modern syntax for
older browsers, and can split code so users only download what a given page
actually needs.

## Vite — a fast, modern bundler

Vite serves source files over native ES modules during development (no
bundling needed — instant startup) and uses Rollup under the hood to
produce an optimized bundle for production.

```bash
npm create vite@latest my-app -- --template vanilla
cd my-app
npm install
npm run dev     # instant dev server with hot module replacement
npm run build    # produces an optimized dist/ folder
```

```javascript
// vite.config.js
import { defineConfig } from "vite";

export default defineConfig({
  build: {
    outDir: "dist",
    sourcemap: true, // ship source maps for easier production debugging
    rollupOptions: {
      output: {
        // group all node_modules code into a separate "vendor" chunk
        manualChunks: {
          vendor: ["lodash", "axios"],
        },
      },
    },
  },
  server: {
    port: 5173,
    proxy: {
      "/api": "http://localhost:3000", // forward API calls to a backend during dev
    },
  },
});
```

## webpack — the long-standing configurable bundler

webpack predates Vite and is still extremely common in existing codebases.
Its behavior is driven entirely by an explicit configuration object.

```bash
npm install --save-dev webpack webpack-cli
```

```javascript
// webpack.config.js
import path from "node:path";

export default {
  mode: "production", // "development" for faster, unminified builds
  entry: "./src/index.js",
  output: {
    filename: "bundle.[contenthash].js", // hash busts the browser cache when content changes
    path: path.resolve("dist"),
    clean: true, // wipe dist/ before each build
  },
  module: {
    rules: [
      { test: /\.css$/, use: ["style-loader", "css-loader"] },
      { test: /\.(png|jpg|svg)$/, type: "asset/resource" },
    ],
  },
  devServer: {
    static: "./dist",
    port: 8080,
    hot: true,
  },
};
```

```bash
npx webpack          # build once, using the config above
npx webpack serve     # start a dev server with hot reloading
```

## Vite vs. webpack

| | Vite | webpack |
|---|---|---|
| Dev server startup | near-instant (native ESM, no bundling) | bundles everything upfront (slower on large apps) |
| Config style | sensible defaults, small config | fully explicit, more boilerplate |
| Production bundler | Rollup (under the hood) | its own bundler |
| Ecosystem age | newer, growing fast | mature, huge plugin ecosystem |
| Good fit | most new projects | existing large configs, niche loaders/plugins |

## Code splitting

Code splitting breaks a bundle into multiple chunks so the browser only
downloads what's needed for the current page, deferring the rest until it's
actually used.

```javascript
// Without splitting: everything is in one bundle, loaded up front
import { renderDashboard } from "./dashboard.js";
import { renderSettings } from "./settings.js";

// With splitting: a dynamic import() creates a separate chunk,
// only fetched when this code path actually runs
async function loadSettingsPage() {
  const { renderSettings } = await import("./settings.js");
  renderSettings();
}

document.getElementById("settings-link").addEventListener("click", loadSettingsPage);
// dashboard.js is in the main bundle; settings.js is its own chunk,
// downloaded only when a user clicks "Settings"
```

Both Vite and webpack recognize dynamic `import()` automatically and split
the target module into its own output chunk — no extra configuration
needed beyond writing the `import()` call.

```javascript
// Route-based splitting in a simple router
const routes = {
  "/dashboard": () => import("./pages/dashboard.js"),
  "/settings": () => import("./pages/settings.js"),
  "/reports": () => import("./pages/reports.js"),
};

async function navigate(path) {
  const pageModule = await routes[path](); // only this route's chunk is fetched
  pageModule.render();
}
```

## Tree-shaking

Tree-shaking removes exported code that's never actually imported anywhere,
shrinking the final bundle. It relies on ES module `import`/`export`
being **static** (not conditional or computed), which lets the bundler
statically trace what's actually used.

```javascript
// mathUtils.js
export function add(a, b) {
  return a + b;
}

export function multiply(a, b) {
  return a * b;
}

export function unusedHelper() { // never imported anywhere in the app
  return "this will be tree-shaken out of the production bundle";
}
```

```javascript
// main.js
import { add } from "./mathUtils.js"; // only `add` is imported

console.log(add(2, 3)); // 5
// `multiply` and `unusedHelper` are never referenced, so a production
// build (Vite/webpack in "production" mode) drops them from the output bundle.
```

```json
// package.json — mark side-effect-free files so bundlers can tree-shake more aggressively
{
  "name": "my-app",
  "sideEffects": false
}
```

`"sideEffects": false` tells the bundler that importing any file in this
package purely for its side effects (like `import "./polyfill.js"` with no
named imports) never happens — so unused exports can be safely dropped.
Set it to an array of specific file paths instead of `false` if some files
genuinely do have side effects (e.g. a CSS import or a global polyfill).

## Why CommonJS resists tree-shaking

```javascript
// CommonJS: module.exports is just an object, computed at runtime —
// a bundler can't statically know which properties will actually be used.
module.exports = {
  add(a, b) { return a + b; },
  multiply(a, b) { return a * b; },
};

// Even if only `add` is destructured here, dead-code elimination for
// CommonJS is much harder for a bundler to prove safe:
const { add } = require("./mathUtils.cjs");
```

This is one of the practical reasons modern libraries ship ES module
(`"module"` field in `package.json`) builds alongside CommonJS ones — ESM's
static `import`/`export` is what makes reliable tree-shaking possible.

## Build & bundling cheat sheet

| Technique | What it does | Enabled by |
|-----------|----------------|------------|
| Code splitting | defer loading unused code until needed | dynamic `import()` |
| Tree-shaking | remove exports that are never imported | static ES module `import`/`export` |
| Minification | shrink code size (short names, no whitespace) | production build mode |
| Content-hash filenames | safe long-term browser caching | `[contenthash]` in output filename |

## Exercise

Take a small multi-page app with `dashboard.js`, `settings.js`, and
`reports.js` modules (stub each with a `render()` function that just
`console.log`s a page name) and configure either Vite or webpack so that
each page is its own dynamically-imported chunk, loaded only when a
corresponding nav link is clicked. Then add one exported function to
`dashboard.js` that nothing ever imports, build for production, and confirm
in the output bundle (or build report) that the unused function was
tree-shaken out.
