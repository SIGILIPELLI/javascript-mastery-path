# 08 · Testing Basics with Jest

## Why automated tests

Manually re-checking behavior after every change doesn't scale. A test
suite encodes expected behavior as runnable code, so you get an instant
pass/fail signal whenever anything changes.

## Installing Jest

Jest is a popular JavaScript testing framework that includes a test runner,
assertion library, and mocking tools in one package.

```bash
npm init -y
npm install --save-dev jest
```

Add a script to `package.json` so `npm test` runs the suite:

```json
{
  "scripts": {
    "test": "jest"
  }
}
```

## Writing your first test

Jest looks for files named `*.test.js` (or inside a `__tests__` folder).
Each test is a `test()` (or its alias `it()`) call with a description and a
function containing one or more assertions.

```javascript
// math.js
function add(a, b) {
  return a + b;
}
module.exports = { add };
```

```javascript
// math.test.js
const { add } = require("./math");

test("adds two positive numbers", () => {
  expect(add(2, 3)).toBe(5);
});

test("adds negative numbers", () => {
  expect(add(-2, -3)).toBe(-5);
});
```

Running `npm test` executes both and reports:

```text
PASS  ./math.test.js
  ✓ adds two positive numbers
  ✓ adds negative numbers
```

## `describe` blocks for grouping

`describe` groups related tests together, producing more readable output
and letting you share setup between them.

```javascript
describe("add()", () => {
  test("adds two positive numbers", () => {
    expect(add(2, 3)).toBe(5);
  });

  test("adds zero", () => {
    expect(add(5, 0)).toBe(5);
  });
});
```

## Common `expect` matchers

`expect(value)` returns an object with matcher methods describing what the
value should look like.

```javascript
test("common matchers", () => {
  expect(2 + 2).toBe(4);                     // strict equality (===)
  expect({ name: "Ada" }).toEqual({ name: "Ada" }); // deep equality for objects/arrays
  expect([1, 2, 3]).toContain(2);             // array/string membership
  expect("hello world").toMatch(/world/);     // regex match
  expect(null).toBeNull();
  expect(undefined).toBeUndefined();
  expect(5).toBeGreaterThan(3);
  expect(() => {
    throw new Error("bad input");
  }).toThrow("bad input"); // asserts a function throws (matching message)
});
```

## Testing async code

Return (or `await`) the promise inside the test — Jest waits for it before
deciding pass/fail, so async bugs aren't silently ignored.

```javascript
// userService.js
async function fetchUserName(id) {
  if (id <= 0) throw new Error("invalid id");
  return `user-${id}`;
}
module.exports = { fetchUserName };
```

```javascript
// userService.test.js
const { fetchUserName } = require("./userService");

test("resolves with a formatted name", async () => {
  const name = await fetchUserName(7);
  expect(name).toBe("user-7");
});

test("rejects for an invalid id", async () => {
  await expect(fetchUserName(-1)).rejects.toThrow("invalid id");
});
```

## Mocking with `jest.fn()` and `jest.mock()`

A mock replaces a real dependency with a fake one you can inspect — useful
for testing code that calls `fetch`, a database, or any function you don't
want to actually run during a test.

```javascript
test("jest.fn() tracks how a function was called", () => {
  const callback = jest.fn((x) => x * 2);

  callback(5);
  callback(10);

  expect(callback).toHaveBeenCalledTimes(2);
  expect(callback).toHaveBeenCalledWith(10);
  expect(callback.mock.results[0].value).toBe(10); // return value of first call
});
```

```javascript
// Mocking global fetch so no real network call happens
test("processes a mocked API response", async () => {
  global.fetch = jest.fn(() =>
    Promise.resolve({
      ok: true,
      json: () => Promise.resolve({ id: 1, name: "Ada" }),
    })
  );

  const response = await fetch("https://example.com/user/1");
  const data = await response.json();

  expect(data.name).toBe("Ada");
  expect(fetch).toHaveBeenCalledWith("https://example.com/user/1");
});
```

## Setup and teardown hooks

`beforeEach`/`afterEach` (and their `*All` counterparts) run shared setup or
cleanup code around every test in a `describe` block.

```javascript
describe("counter", () => {
  let count;

  beforeEach(() => {
    count = 0; // fresh state before every test, avoiding cross-test leakage
  });

  test("starts at zero", () => {
    expect(count).toBe(0);
  });

  test("can be incremented", () => {
    count += 1;
    expect(count).toBe(1);
  });
});
```

## Jest matcher cheat sheet

| Matcher | Use for |
|---------|---------|
| `.toBe(value)` | primitives, strict `===` equality |
| `.toEqual(value)` | deep equality on objects/arrays |
| `.toContain(item)` | array or string membership |
| `.toThrow(messageOrPattern)` | a function throws synchronously |
| `.rejects.toThrow(...)` | a promise rejects (with `await expect(...)`) |
| `.toHaveBeenCalledWith(...)` | a `jest.fn()` mock's call arguments |
| `.toBeNull()` / `.toBeUndefined()` | absence checks |

## Exercise

Create `stringUtils.js` exporting a function `truncate(str, maxLength)` that
returns `str` unchanged if it's short enough, or the first `maxLength`
characters plus `"..."` if it's longer. Then write `stringUtils.test.js`
with at least three tests: a string under the limit, a string over the
limit, and an edge case of `maxLength === 0`.
