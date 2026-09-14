# 02 · Node.js Fundamentals

## 🎥 Video walkthrough

<iframe width="100%" height="400" style="max-width:720px;aspect-ratio:16/9;height:auto;" src="https://www.youtube.com/embed/RuuMhtb45Rc" title="Video walkthrough" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Level 3 moves off the browser and onto the server. Node.js runs JavaScript
outside a browser, giving you access to the file system, network sockets,
and the OS — none of which exist in browser JavaScript. This module covers
Node's two module systems, reading/writing files, and streams.

## CommonJS modules

CommonJS (`require`/`module.exports`) is Node's original module system and
still the default for plain `.js` files.

```javascript
// math.cjs
function add(a, b) {
  return a + b;
}

function multiply(a, b) {
  return a * b;
}

module.exports = { add, multiply };
```

```javascript
// main.cjs
const { add, multiply } = require("./math.cjs");

console.log(add(2, 3));      // 5
console.log(multiply(4, 5)); // 20
```

```bash
node main.cjs
```

CommonJS `require` calls are synchronous and can happen anywhere in a file
(even conditionally), which is one reason the newer ESM system was
introduced with stricter, more static import rules.

## ES modules in Node

To use `import`/`export` syntax directly, either name files `.mjs` or set
`"type": "module"` in `package.json` — then plain `.js` files are treated as
ES modules.

```json
{
  "name": "node-fundamentals-demo",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node main.js"
  }
}
```

```javascript
// math.js
export function add(a, b) {
  return a + b;
}

export function multiply(a, b) {
  return a * b;
}
```

```javascript
// main.js
import { add, multiply } from "./math.js"; // note the required .js extension

console.log(add(2, 3));      // 5
console.log(multiply(4, 5)); // 20
```

## Mixing systems: what to know

You can't `require()` an ES module, but ESM code can `import` most CommonJS
packages, and Node exposes a handful of globals differently between the two
systems (`__dirname`/`__filename` exist only in CommonJS).

```javascript
// ESM equivalent of __dirname / __filename
import { fileURLToPath } from "node:url";
import { dirname } from "node:path";

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

console.log(__dirname);  // absolute path to this file's folder
console.log(__filename); // absolute path to this file
```

| Feature | CommonJS | ES Modules |
|---------|----------|------------|
| Import | `require("./file")` | `import x from "./file.js"` |
| Export | `module.exports = ...` | `export`/`export default` |
| File extension in imports | optional | required |
| `__dirname` / `__filename` | built in | derive via `import.meta.url` |
| Enabled by | default for `.js` | `"type": "module"` or `.mjs` |
| Loading | synchronous | can be async (top-level `await`) |

## The file system module — callbacks vs. promises

Node's original `fs` module is callback-based. `fs/promises` wraps the same
operations in promises, which is almost always the better fit for modern
`async`/`await` code.

```javascript
// Old style: callback-based fs
import fs from "node:fs";

fs.readFile("notes.txt", "utf8", (error, data) => {
  if (error) {
    console.error("read failed:", error.message);
    return;
  }
  console.log("contents:", data);
});
```

```javascript
// Modern style: fs/promises
import { readFile, writeFile, appendFile, mkdir } from "node:fs/promises";

async function saveNote(text) {
  await mkdir("data", { recursive: true }); // create the folder if missing
  await writeFile("data/notes.txt", text, "utf8");
}

async function readNote() {
  try {
    const contents = await readFile("data/notes.txt", "utf8");
    console.log("note contents:", contents);
  } catch (error) {
    if (error.code === "ENOENT") {
      console.log("no note saved yet");
    } else {
      throw error;
    }
  }
}

await saveNote("Buy milk\n");
await appendFile("data/notes.txt", "Walk the dog\n");
await readNote();
// note contents: Buy milk
// Walk the dog
```

## Working with JSON files

A very common Node task is treating a JSON file as lightweight storage —
read it, parse it, mutate the in-memory data, then write it back.

```javascript
import { readFile, writeFile } from "node:fs/promises";

const DB_PATH = "data/users.json";

async function loadUsers() {
  try {
    const raw = await readFile(DB_PATH, "utf8");
    return JSON.parse(raw);
  } catch (error) {
    if (error.code === "ENOENT") return []; // first run — file doesn't exist yet
    throw error;
  }
}

async function saveUsers(users) {
  await writeFile(DB_PATH, JSON.stringify(users, null, 2), "utf8");
}

const users = await loadUsers();
users.push({ id: users.length + 1, name: "Grace" });
await saveUsers(users);
console.log(users); // [ { id: 1, name: 'Grace' } ]
```

## Streams — processing data in chunks

Reading a whole file into memory with `readFile` is fine for small files,
but large files (or network data) should be processed as a **stream** —
data arrives in chunks so memory usage stays flat regardless of file size.

```javascript
import { createReadStream, createWriteStream } from "node:fs";

const readStream = createReadStream("data/notes.txt", { encoding: "utf8" });
const writeStream = createWriteStream("data/notes-upper.txt");

readStream.on("data", (chunk) => {
  writeStream.write(chunk.toUpperCase()); // transform each chunk as it arrives
});

readStream.on("end", () => {
  writeStream.end();
  console.log("done transforming file");
});

readStream.on("error", (error) => {
  console.error("stream failed:", error.message);
});
```

## Piping streams

`.pipe()` connects a readable stream to a writable one and handles
backpressure automatically (pausing the source if the destination can't keep
up) — almost always preferable to manual `.on("data")` handling.

```javascript
import { createReadStream, createWriteStream } from "node:fs";
import { createGzip } from "node:zlib";

createReadStream("data/notes.txt")
  .pipe(createGzip()) // Transform stream: compresses chunks as they pass through
  .pipe(createWriteStream("data/notes.txt.gz"))
  .on("finish", () => console.log("compressed notes.txt -> notes.txt.gz"));
```

## Stream types

| Type | Direction | Example |
|------|-----------|---------|
| Readable | data flows out | `fs.createReadStream`, an incoming HTTP request body |
| Writable | data flows in | `fs.createWriteStream`, an HTTP response |
| Duplex | both | a TCP socket |
| Transform | in, modified, out | `zlib.createGzip()`, a CSV parser |

## How It Actually Works

Node's event loop, implemented by libuv, isn't a single queue — it's a sequence of
distinct **phases** that run in a fixed order every tick: timers (due `setTimeout`/
`setInterval` callbacks), pending callbacks, idle/prepare (internal), poll (retrieve new
I/O events, execute I/O callbacks — this phase can block here waiting for events if
there's nothing else to do), check (`setImmediate` callbacks specifically live here),
and close callbacks. Microtasks (promise `.then`, `queueMicrotask`) and
`process.nextTick` callbacks are **not** a phase — Node drains the `nextTick` queue and
then the microtask queue completely after *every single callback*, not just once per
loop iteration, which is why `process.nextTick` callbacks can starve I/O entirely if
they keep scheduling more of themselves.

The reason Node can do non-blocking file I/O in a single-threaded language is that
CPU/blocking-style operations (`fs.readFile`, DNS lookups, some crypto) are handed off
to libuv's **thread pool** (default size 4, tunable via `UV_THREADPOOL_SIZE`) — actual
OS threads do the blocking read, and when it completes, libuv queues the JS callback to
run on the single main thread during the poll phase. Network I/O (sockets, HTTP) on
Linux doesn't even need the thread pool — it uses the OS's native async I/O
notification (`epoll`), so `fetch`/`http.get` calls scale to far more concurrent
connections than thread-pool-bound file operations, because there's no fixed pool size
limiting them.
## Exercise

Write a small Node script (ESM, `"type": "module"`) that reads a directory
path from `process.argv`, lists all `.txt` files in it using `fs/promises`,
and streams the concatenated contents of every file into a single
`combined.txt` using `createReadStream`/`createWriteStream` and `.pipe()`.
Handle the case where the directory doesn't exist with a clear error message
instead of a crash.
