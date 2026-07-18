# 05 · Working with JSON & Fetch API

## JSON: JavaScript Object Notation

JSON is a text format for representing data — the universal language APIs
use to send and receive structured data over HTTP.

```javascript
const user = { name: "Ada", age: 30, tags: ["admin", "beta"] };

const json = JSON.stringify(user); // convert a JS value to a JSON string
console.log(json); // {"name":"Ada","age":30,"tags":["admin","beta"]}

const parsed = JSON.parse(json); // convert a JSON string back to a JS value
console.log(parsed.name); // Ada
console.log(parsed.tags[0]); // admin
```

`JSON.stringify` accepts extra arguments for pretty-printing and filtering
keys — handy for logging or writing config files.

```javascript
console.log(JSON.stringify(user, null, 2));
// {
//   "name": "Ada",
//   "age": 30,
//   "tags": [
//     "admin",
//     "beta"
//   ]
// }

console.log(JSON.stringify(user, ["name", "age"])); // only these keys
// {"name":"Ada","age":30}
```

## What JSON can't represent

JSON only supports plain data — some JavaScript values are silently dropped
or converted when stringified.

```javascript
const tricky = {
  fn: () => "hi",       // functions are omitted entirely
  missing: undefined,    // undefined keys are omitted entirely
  when: new Date(2024, 0, 1), // Dates become ISO strings
  empty: null,           // null is preserved as-is
};

console.log(JSON.stringify(tricky));
// {"when":"2024-01-01T00:00:00.000Z","empty":null}
```

## Handling malformed JSON

`JSON.parse` throws a `SyntaxError` on invalid input — always wrap it in
`try`/`catch` when the data comes from outside your program (a network
response, user input, a file).

```javascript
function safeParseJSON(text) {
  try {
    return { data: JSON.parse(text), error: null };
  } catch (error) {
    return { data: null, error: error.message };
  }
}

console.log(safeParseJSON('{"ok": true}')); // { data: { ok: true }, error: null }
console.log(safeParseJSON("not json"));
// { data: null, error: 'Unexpected token o in JSON at position 1' }
```

## The Fetch API

`fetch` makes an HTTP request and returns a Promise that resolves to a
`Response` object once the headers arrive — the body itself is read
separately and asynchronously, via its own methods.

```javascript
async function getTodo(id) {
  const response = await fetch(`https://jsonplaceholder.typicode.com/todos/${id}`);
  const todo = await response.json(); // parses the body as JSON, returns a Promise
  return todo;
}

getTodo(1).then((todo) => console.log(todo));
// { userId: 1, id: 1, title: 'delectus aut autem', completed: false }
```

## Checking for HTTP errors

A key `fetch` gotcha: the returned Promise only rejects on network failure
(DNS error, no connection). A `404` or `500` response still resolves
successfully — you must check `response.ok` yourself.

```javascript
async function getTodoSafely(id) {
  try {
    const response = await fetch(`https://jsonplaceholder.typicode.com/todos/${id}`);

    if (!response.ok) {
      // response.ok is false for any status outside 200-299
      throw new Error(`request failed with status ${response.status}`);
    }

    return await response.json();
  } catch (error) {
    console.log("could not load todo:", error.message);
    return null;
  }
}

getTodoSafely(999999).then((todo) => console.log(todo));
```

## Sending data with `fetch`

For anything beyond a `GET`, pass an options object with the method, a JSON
body, and the matching `Content-Type` header.

```javascript
async function createPost(title, body) {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ title, body, userId: 1 }),
  });

  if (!response.ok) {
    throw new Error(`create failed: ${response.status}`);
  }

  return response.json();
}

createPost("Hello", "World").then((created) => console.log(created.id));
```

## Fetch in the browser vs. Node.js

Modern Node.js (18+) ships a global `fetch` that behaves the same as the
browser's, so the same code generally runs in both environments unmodified.

| Environment | `fetch` availability | Common gotchas |
|-------------|------------------------|------------------|
| Browser | global, built-in | CORS restrictions on cross-origin requests |
| Node.js 18+ | global, built-in | no CORS concept, but no DOM either |
| Node.js < 18 | not built-in | needs a package like `node-fetch` |

## Fetch cheat sheet

| Task | Code |
|------|------|
| GET and parse JSON | `const data = await (await fetch(url)).json();` |
| Check for HTTP errors | `if (!response.ok) throw new Error(response.status);` |
| POST JSON | `fetch(url, { method: "POST", headers: {...}, body: JSON.stringify(data) })` |
| Parse JSON string | `JSON.parse(text)` |
| Serialize to JSON string | `JSON.stringify(value, null, 2)` |

## Exercise

Write an `async` function `getUserPosts(userId)` that fetches
`https://jsonplaceholder.typicode.com/posts?userId=<userId>`, throws a
descriptive error if the response isn't OK, and otherwise returns an array
of just the post titles (use `.map()` on the parsed JSON).
