---
title: "Final Quiz"
description: "Test what you have learned across all four levels of JavaScript Mastery Path with a 20-question final quiz."
---

# Final Quiz

20 questions drawn from all four levels of this course. You need **70% (14 out of 20)** to pass —
you can retake it as many times as you like. Your best result is saved in this browser.

<div id="mp-quiz-root"></div>

<script type="application/json" id="mp-quiz-data">
[
  {
    "question": "What does this print?\n```\nconsole.log(typeof null, typeof undefined, 5 === \"5\");\n```",
    "options": [
      "object undefined false",
      "null undefined true",
      "object object false",
      "undefined undefined false"
    ],
    "correctIndex": 0,
    "explanation": "typeof null is the long-standing quirk \"object\"; === checks type, so 5 === \"5\" is false."
  },
  {
    "question": "What does this print?\n```\nconst o = { a: 1 };\nconst { a, b = 5 } = o;\nconsole.log(a, b);\n```",
    "options": [
      "undefined 5",
      "5 1",
      "1 5",
      "1 undefined"
    ],
    "correctIndex": 2,
    "explanation": "Destructuring takes a from the object and falls back to the default 5 for the missing b."
  },
  {
    "question": "What does this print?\n```\nfunction mk() { let c = 0; return () => ++c; }\nconst f = mk(); f();\nconsole.log(f());\n```",
    "options": [
      "0",
      "2",
      "1",
      "undefined"
    ],
    "correctIndex": 1,
    "explanation": "The returned function closes over c, so the second call returns 2."
  },
  {
    "question": "What does this print?\n```\nconsole.log([1,2,3,4,5].filter(n => n % 2).reduce((s, n) => s + n, 0));\n```",
    "options": [
      "9",
      "6",
      "3",
      "15"
    ],
    "correctIndex": 0,
    "explanation": "filter keeps 1, 3, 5, and reduce sums them to 9."
  },
  {
    "question": "What do Number(\"42px\") and parseInt(\"42px\") return?",
    "options": [
      "NaN and NaN",
      "42 and 42",
      "NaN and 42",
      "42 and NaN"
    ],
    "correctIndex": 2,
    "explanation": "Number must parse the whole string; parseInt reads the leading digits and ignores the rest."
  },
  {
    "question": "What is the order of the output?\n```\nsetTimeout(() => console.log(\"t\"), 0);\nPromise.resolve().then(() => console.log(\"p\"));\nconsole.log(\"s\");\n```",
    "options": [
      "s, p, t",
      "t, p, s",
      "s, t, p",
      "p, s, t"
    ],
    "correctIndex": 0,
    "explanation": "Synchronous code first, then the microtask queue (promise callbacks), then the task queue (timers)."
  },
  {
    "question": "What does this print?\n```\nconsole.log(null ?? \"d\", 0 ?? \"d\", 0 || \"d\");\n```",
    "options": [
      "d d d",
      "null 0 0",
      "d 0 0",
      "d 0 d"
    ],
    "correctIndex": 3,
    "explanation": "?? only falls back for null or undefined, so 0 is kept; || falls back for any falsy value, including 0."
  },
  {
    "question": "What does [10, 9, 1].sort() return?",
    "options": [
      "[9, 10, 1]",
      "[1, 10, 9]",
      "[10, 9, 1]",
      "[1, 9, 10]"
    ],
    "correctIndex": 1,
    "explanation": "With no comparison function, sort compares elements as strings, so \"1\" < \"10\" < \"9\"."
  },
  {
    "question": "What does this print?\n```\nclass A { speak() { return \"a\"; } }\nclass B extends A { speak() { return super.speak() + \"b\"; } }\nconsole.log(new B().speak());\n```",
    "options": [
      "b",
      "a",
      "ba",
      "ab"
    ],
    "correctIndex": 3,
    "explanation": "super.speak() calls the parent's method, then the subclass adds \"b\"."
  },
  {
    "question": "What does JSON.stringify({ x: undefined, y: () => 1, z: null, d: 1 }) return?",
    "options": [
      "{\"x\":null,\"y\":null,\"z\":null,\"d\":1}",
      "An error",
      "{\"z\":null,\"d\":1}",
      "{\"x\":undefined,\"z\":null,\"d\":1}"
    ],
    "correctIndex": 2,
    "explanation": "Functions and undefined values are omitted; null is kept."
  },
  {
    "question": "What do these two loops print?\n```\nfor (var i = 0; i < 3; i++) setTimeout(() => console.log(i), 0);\nfor (let j = 0; j < 3; j++) setTimeout(() => console.log(j), 0);\n```",
    "options": [
      "3 3 3 then 0 1 2",
      "3 3 3 then 3 3 3",
      "0 1 2 then 3 3 3",
      "0 1 2 then 0 1 2"
    ],
    "correctIndex": 0,
    "explanation": "var is function-scoped, so all callbacks see the final i (3); let creates a new j for each iteration."
  },
  {
    "question": "You need to run three independent API calls and use all results. Which is best?",
    "options": [
      "Use setTimeout for each",
      "await Promise.all([a(), b(), c()])",
      "await a(); await b(); await c();",
      "Nest three callbacks"
    ],
    "correctIndex": 1,
    "explanation": "Promise.all runs them concurrently, so the total wait is the slowest call, not the sum."
  },
  {
    "question": "A service has to report which of five health checks failed without stopping at the first failure. Which should you use?",
    "options": [
      "Promise.allSettled",
      "Promise.race",
      "Promise.all",
      "A for loop with break"
    ],
    "correctIndex": 0,
    "explanation": "allSettled waits for everything and reports each outcome; all rejects on the first failure."
  },
  {
    "question": "A user types in a comment box and you insert their text into the page. Which is the safe way?",
    "options": [
      "element.textContent = comment",
      "element.innerHTML += comment",
      "document.write(comment)",
      "eval(comment)"
    ],
    "correctIndex": 0,
    "explanation": "textContent treats the text as plain text, which prevents XSS; innerHTML would parse injected markup."
  },
  {
    "question": "An Express route receives POST /notes with no `text` field in the body. What should a well-designed handler return?",
    "options": [
      "It should crash",
      "201 with an empty note",
      "400 with an error message",
      "200 with null"
    ],
    "correctIndex": 2,
    "explanation": "Validate input and return a 400 Bad Request; 201 is for successful creation."
  },
  {
    "question": "Why do the lessons use prepared statements with ? placeholders for database queries?",
    "options": [
      "To avoid needing a schema",
      "To enable transactions",
      "To prevent SQL injection",
      "To make queries shorter"
    ],
    "correctIndex": 2,
    "explanation": "Values are passed as data, never concatenated into the SQL string."
  },
  {
    "question": "Your Node server freezes every other request while one endpoint runs a 10-second CPU-heavy loop. What is the cause and the fix?",
    "options": [
      "Too many routes; split the file",
      "It blocks the single-threaded event loop; offload it to a worker thread",
      "Express is outdated; upgrade it",
      "The database is slow; add an index"
    ],
    "correctIndex": 1,
    "explanation": "CPU-bound work blocks the event loop; worker_threads (or clustering) keeps it free."
  },
  {
    "question": "A JWT access token is short-lived (15 minutes) on purpose. Why?",
    "options": [
      "Browsers delete long tokens",
      "JWTs are hard to revoke, so a short life limits the damage if one leaks",
      "bcrypt requires it",
      "Short tokens are smaller"
    ],
    "correctIndex": 1,
    "explanation": "A JWT is verified without a database lookup, so you can't easily revoke it before it expires."
  },
  {
    "question": "In a Jest test, how do you verify that a function was called with specific arguments?",
    "options": [
      "expect(mockFn).toBe(...)",
      "expect(mockFn).toHaveBeenCalledWith(...)",
      "expect(mockFn).toMatch(...)",
      "expect(mockFn).toEqual(...)"
    ],
    "correctIndex": 1,
    "explanation": "toHaveBeenCalledWith checks a mock's call arguments; toHaveBeenCalledTimes checks the count."
  },
  {
    "question": "Which package.json setting lets plain .js files use import/export in Node?",
    "options": [
      "\"main\": \"module\"",
      "\"imports\": \"all\"",
      "\"type\": \"module\"",
      "\"esm\": true"
    ],
    "correctIndex": 2,
    "explanation": "Otherwise .js files are CommonJS by default unless you rename to .mjs."
  }
]
</script>

## Coding challenges

Try these on your own computer, then compare your output. Nothing you write is sent anywhere.

### Group words by length

Write `groupByLength(words)` that returns an object whose keys are word lengths and whose values are arrays of the words with that length, in input order. Print the result with `JSON.stringify`.

**Starter code**

```javascript
function groupByLength(words) {
  // your code here
}
console.log(JSON.stringify(groupByLength(["to", "be", "or", "not", "bee"])));
```

**Expected output**

```text
{"2":["to","be","or"],"3":["not","bee"]}
```

<details><summary>Show solution and explanation</summary>

<pre><code>function groupByLength(words) {
  const out = {};
  for (const w of words) (out[w.length] ??= []).push(w);
  return out;
}
console.log(JSON.stringify(groupByLength([&quot;to&quot;, &quot;be&quot;, &quot;or&quot;, &quot;not&quot;, &quot;bee&quot;])));</code></pre>

<p>`??=` creates the array the first time a length is seen, then push adds the word. Object keys are strings, so the lengths print as &quot;2&quot; and &quot;3&quot;.</p>
</details>

### Run promises in parallel and handle an error

Use `Promise.all` to wait for three delayed values (`a`, `b`, `c`) and print them joined by commas in the original order, then show that `await` on a rejected promise can be caught with try/catch.

**Starter code**

```javascript
const delay = (ms, v) => new Promise((r) => setTimeout(() => r(v), ms));
async function main() {
  // 1. await Promise.all([...]) with delays 30, 10, 20ms for "a", "b", "c"; print the results joined with ","
  // 2. try { await Promise.reject(new Error("boom")); } catch ... print "caught boom"
}
main();
```

**Expected output**

```text
a,b,c
caught boom
```

<details><summary>Show solution and explanation</summary>

<pre><code>const delay = (ms, v) =&gt; new Promise((r) =&gt; setTimeout(() =&gt; r(v), ms));
async function main() {
  const results = await Promise.all([delay(30, &quot;a&quot;), delay(10, &quot;b&quot;), delay(20, &quot;c&quot;)]);
  console.log(results.join(&quot;,&quot;));
  try { await Promise.reject(new Error(&quot;boom&quot;)); } catch (e) { console.log(&quot;caught&quot;, e.message); }
}
main();</code></pre>

<p>Promise.all keeps results in the order the promises were passed, even though b finishes first. try/catch around await catches rejected promises.</p>
</details>
