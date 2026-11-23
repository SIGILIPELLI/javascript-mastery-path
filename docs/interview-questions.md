---
title: "JavaScript Interview Questions and Answers (184 Questions)"
description: "184 JavaScript interview questions with answers and explanations, from beginner to advanced, organised by level. Free, from the JavaScript Mastery Path."
---

# JavaScript Interview Questions and Answers

**184 questions** with the correct answer and a short explanation, organised from beginner (Level 1) to advanced (Level 4). Every question comes from a lesson in the free JavaScript Mastery Path, so you can open the lesson to go deeper.

!!! tip "Practise, don't just read"
    Each lesson also ends with an interactive quiz, and the [final quiz](quiz.md) tests all four levels. Try to answer each question below before reading the answer.

## Level 1 · Entry

### [01 · Setup &amp; First Program](level-1/01-setup.md)

**Q1.** What does REPL stand for, and what is it for in Node?

**Answer:** Read-Eval-Print Loop, an interactive shell for quick experiments

Typing node with no file opens the REPL, where each expression is evaluated and its result printed.

**Q2.** Which command runs a file named hello.js with Node?

**Answer:** node hello.js

Pass the file path to the node binary.

**Q3.** In the lesson's explanation, which engine does Node use to run JavaScript, and what is libuv for?

**Answer:** V8 runs the JavaScript; libuv provides the event loop, thread pool and non-blocking I/O

Node is a C++ shell around V8 plus libuv; V8 runs pure JavaScript while Node adds I/O and timers.

**Q4.** Why can the same hello.js run in Node and in a browser page?

**Answer:** It is the same JavaScript, as long as it avoids Node-only or browser-only APIs

The language is shared; APIs such as the DOM or file system are provided by the host environment.

### [02 · Variables, Data Types &amp; Operators](level-1/02-variables-data-types.md)

**Q5.** Which declaration should you prefer by default?

**Answer:** const

Use const by default, let when the value must change, and avoid var.

**Q6.** What does typeof null return?

**Answer:** "object"

This is a famous long-standing JavaScript quirk.

**Q7.** What is the result of 5 === "5"?

**Answer:** false

Strict equality checks type as well as value; loose == would coerce and return true.

**Q8.** Which value is truthy?

**Answer:** "x" (a non-empty string)

The falsy values are false, 0, "", null, undefined and NaN; everything else is truthy.

### [03 · Control Flow](level-1/03-control-flow.md)

**Q9.** What does a do...while loop guarantee that a while loop does not?

**Answer:** The body runs at least once

do...while checks its condition after the body, so the body runs once even when the condition is false.

**Q10.** Which loop is preferred for iterating over the values of an array?

**Answer:** for...of

for...of gives values; for...in gives keys or indexes and is mainly for objects.

**Q11.** In a switch, what happens if a case has no break or return?

**Answer:** Execution falls through to the next case

Falling through is intentional when grouping cases such as "Sat" and "Sun".

**Q12.** What does `continue` do inside a loop?

**Answer:** Skips the rest of the current iteration and moves to the next

In the lesson it skips odd numbers; break is what exits the loop.

### [04 · Functions &amp; Scope](level-1/04-functions-scope.md)

**Q13.** What does a rest parameter such as ...numbers provide?

**Answer:** A real array containing the extra arguments

Rest parameters gather the remaining arguments into an actual array.

**Q14.** What happens if you reference a let variable before its declaration?

**Answer:** ReferenceError (temporal dead zone)

let and const are hoisted but not initialized, so access before declaration throws.

**Q15.** What does typeof a var variable return when read before its assignment line?

**Answer:** "undefined"

var is hoisted as a declaration only, so its value is undefined until the assignment runs.

**Q16.** Which statement about arrow functions is true, according to the lesson?

**Answer:** They do not have their own this

Arrow functions offer shorter syntax and do not bind their own this.

### [05 · Arrays &amp; Objects](level-1/05-arrays-objects.md)

**Q17.** Which statement about map, filter and reduce is correct?

**Answer:** They return new arrays or values without mutating the original

Prefer them over manual loops for transformations.

**Q18.** Why does the lesson write [...numbers].sort(...) instead of numbers.sort(...)?

**Answer:** sort mutates the array, so sorting a copy protects the original

Array.sort sorts in place; copying first leaves the original unchanged.

**Q19.** What does nested.profile?.social?.twitter return when social is null?

**Answer:** undefined, with no error

Optional chaining short-circuits to undefined; adding ?? "none" supplies a fallback.

**Q20.** How do you read the last element of an array with modern syntax?

**Answer:** fruits.at(-1)

at(-1) is cleaner than fruits[fruits.length - 1].

### [06 · Strings &amp; Template Literals](level-1/06-strings-template-literals.md)

**Q21.** What does `str.toUpperCase();` on its own line do to str?

**Answer:** Nothing to str; it returns a new string you must assign

Strings are immutable, so every modification returns a new string.

**Q22.** Which syntax inserts a variable into a template literal?

**Answer:** `${value}`

Template literals use backticks and ${} for embedded expressions.

**Q23.** What does "hello".startsWith("he") return?

**Answer:** true

startsWith tests whether the string begins with the given text.

**Q24.** Which method pads a string on the left to a given width?

**Answer:** padStart

padStart(n) pads at the start; padEnd(n) pads at the end.

### [07 · DOM Basics &amp; Events](level-1/07-dom-events.md)

**Q25.** Which method returns all elements matching a CSS selector?

**Answer:** document.querySelectorAll()

querySelectorAll returns a NodeList of every match; querySelector returns only the first.

**Q26.** What is event delegation?

**Answer:** Attaching one listener to a parent and checking event.target for the child that was clicked

It scales better, especially when children are added or removed dynamically.

**Q27.** Which event fires as a user types in a form field?

**Answer:** input

input fires whenever the field's value changes; submit fires when the form is submitted.

**Q28.** Why can't the DOM snippets run in Node.js?

**Answer:** Node.js has no DOM; they only apply in the browser

The DOM is provided by the browser, not by the JavaScript language or Node.

### [08 · Error Handling Basics](level-1/08-error-handling.md)

**Q29.** What does a finally block do?

**Answer:** Runs whether or not an error was thrown, for cleanup

The lesson notes it always runs, no matter what.

**Q30.** How do you raise your own error?

**Answer:** throw new Error("message")

throw with an Error object, as in the withdraw example for insufficient funds.

**Q31.** What does JSON.parse("not valid json") do?

**Answer:** Throws an error that a try/catch can handle

The catch block receives the error and can read error.message.

**Q32.** Why is a bare catch that silently swallows everything a problem?

**Answer:** It hides bugs; you should inspect what you catch

JavaScript has no typed catch clauses, so check the error inside the block.

### [09 · Modules &amp; npm Basics](level-1/09-modules-npm.md)

**Q33.** How do you make Node treat .js files as ES modules?

**Answer:** Add "type": "module" to package.json (or name files .mjs)

Node treats .js files as CommonJS by default.

**Q34.** How many default exports can an ES module file have?

**Answer:** One

A file may have many named exports but only one default export.

**Q35.** What is package-lock.json for?

**Answer:** Pinning exact versions so installs are reproducible across machines

npm install records dependencies in package.json and exact versions in the lock file.

**Q36.** What does npm run start do?

**Answer:** Runs the "start" script defined in package.json

npm scripts are named commands defined in package.json.

### [10 · Project — Browser To-Do List App](level-1/10-project-todo-app.md)

**Q37.** Which browser feature does the to-do project use to keep tasks between visits?

**Answer:** localStorage

The project combines DOM, events, error handling and localStorage persistence.

**Q38.** Why does the project use event delegation on the task list?

**Answer:** Tasks are added and removed dynamically, so one parent listener handles clicks

One listener on the parent checking event.target works for children added later.

**Q39.** Which of these is a feature of the to-do app?

**Answer:** Add, mark done, and delete tasks

The app adds tasks from an input, lists them with status, marks them done and deletes them.

**Q40.** Which Level 1 topics does the project combine?

**Answer:** Functions, arrays and objects, strings, DOM, events, error handling and localStorage

It is an end-to-end project tying together everything from Level 1.

## Level 2 · Intermediate

### [01 · Closures &amp; Scope Deep Dive](level-2/01-closures-scope.md)

**Q41.** What is a closure?

**Answer:** A function combined with the lexical environment it was declared in

The inner function keeps a live reference to the outer variables, not a snapshot.

**Q42.** In makeCounter, why can't you read counter.count directly?

**Answer:** count is private state held in the closure, not a property of the returned object

Only the returned methods can reach count; counter.count is undefined.

**Q43.** counterA and counterB come from two calls to makeCounter(). Are their counts shared?

**Answer:** No, each call creates a brand-new independent count

Each call creates a separate closure, so counterA and counterB advance independently.

**Q44.** What determines a function's scope chain?

**Answer:** Where it is written in the source, not where it is called from

JavaScript uses lexical scope.

### [02 · Prototypes &amp; Classes](level-2/02-prototypes-classes.md)

**Q45.** What does Object.create(animal) do?

**Answer:** Makes a new object whose prototype is animal

The new object looks up missing properties on animal through its prototype.

**Q46.** Why put methods on Person.prototype rather than inside the constructor?

**Answer:** Every instance shares one copy of the function in memory

ada.greet === grace.greet is true because both use the same prototype function.

**Q47.** What does typeof Animal return for a class Animal?

**Answer:** "function"

class is syntactic sugar; classes are functions under the hood.

**Q48.** In a subclass constructor, what must happen before using this?

**Answer:** super(...) must be called

super calls the parent constructor and must run before this is used.

### [03 · Asynchronous JavaScript I](level-2/03-async-javascript-i.md)

**Q49.** In the lesson's setTimeout(..., 0) example, when does the callback run?

**Answer:** After all synchronous code finishes, once the call stack is empty

Even with a delay of 0, the callback waits for the call stack to empty.

**Q50.** What is a callback?

**Answer:** A function passed as an argument to be called later

Callbacks are often called after an operation such as a timer or request finishes.

**Q51.** Which queue does the event loop fully drain before the task queue?

**Answer:** The microtask queue (Promise callbacks)

Promise callbacks and async/await continuations go into the microtask queue.

**Q52.** What does the call stack do with function calls?

**Answer:** Pushes each call on and pops it off when it returns

JavaScript runs one thing at a time; each call waits for the ones it invokes to finish.

### [04 · Promises &amp; Async/Await](level-2/04-promises-async-await.md)

**Q53.** Which are the three states of a Promise?

**Answer:** pending, fulfilled, rejected

Once it settles (fulfilled or rejected) it never changes again.

**Q54.** What does each .then return?

**Answer:** A new Promise, enabling chaining

Chaining replaces the nested callback pyramid.

**Q55.** What does an async function always return?

**Answer:** A Promise

async/await is syntax sugar over Promises.

**Q56.** What does await do inside an async function?

**Answer:** Pauses that function until the Promise settles, without blocking the rest of the program

Execution inside the function resumes once the awaited Promise settles.

### [05 · Working with JSON &amp; Fetch API](level-2/05-json-fetch.md)

**Q57.** What does JSON.stringify do with a function-valued or undefined property?

**Answer:** Omits it from the output

Functions and undefined keys are silently dropped; null is preserved and Dates become ISO strings.

**Q58.** Why wrap JSON.parse in try/catch for outside data?

**Answer:** It throws a SyntaxError on invalid input

Data from a network response, user or file may be malformed.

**Q59.** What does fetch return, and when does it resolve?

**Answer:** A Promise that resolves to a Response once the headers arrive; the body is read separately

You then call methods such as response.json(), which also returns a Promise.

**Q60.** What does JSON.stringify(user, null, 2) produce?

**Answer:** A pretty-printed JSON string with 2-space indentation

The extra arguments handle filtering and pretty-printing.

### [06 · Regular Expressions in JS](level-2/06-regular-expressions.md)

**Q61.** What does the g flag do?

**Answer:** Finds all matches, not just the first

The i flag ignores case; g finds every match.

**Q62.** Which pattern matches exactly 5 digits across the whole string?

**Answer:** /^\d{5}$/

^ and $ anchor to the start and end, and {5} requires exactly five repetitions.

**Q63.** What does \w match?

**Answer:** A word character: letters, digits or underscore

\d is a digit, \s whitespace and . any character except a newline.

**Q64.** How does exec() differ from test()?

**Answer:** exec returns match details (or null); test returns a boolean

With the g flag, exec can be called repeatedly to step through all matches.

### [07 · Error Handling Advanced](level-2/07-error-handling-advanced.md)

**Q65.** Why create custom Error subclasses?

**Answer:** So calling code can distinguish failure kinds by type instead of parsing messages

Subclasses like ValidationError can carry extra context such as a field.

**Q66.** What does error instanceof ValidationError let a catch block do?

**Answer:** Handle different failures differently

Branch on type, and re-throw errors you don't know how to handle.

**Q67.** In the signup example, why does the catch block re-throw unknown errors?

**Answer:** It doesn't know how to handle them

Swallowing unknown errors would hide bugs.

**Q68.** What does try/catch around await catch?

**Answer:** Both synchronous throws and rejected Promises

That is a main reason async/await is easier to reason about than raw .then() chains.

### [08 · Testing Basics with Jest](level-2/08-testing-jest.md)

**Q69.** Which file names does Jest look for by default?

**Answer:** *.test.js or files inside a __tests__ folder

Each test is a test() or it() call with a description and assertions.

**Q70.** Which matcher checks deep equality of objects?

**Answer:** toEqual

toBe is strict equality (===); toEqual compares objects and arrays deeply.

**Q71.** What does describe do?

**Answer:** Groups related tests for readable output and shared setup

It organizes tests into named groups.

**Q72.** How should you test async code in Jest?

**Answer:** Return or await the promise inside the test

Jest waits for the promise before deciding pass/fail, so async bugs aren't silently ignored.

### [09 · Build Tooling Intro](level-2/09-build-tooling.md)

**Q73.** What does a bundler do?

**Answer:** Follows imports from an entry file and combines everything into a few output files

It resolves the dependency graph so the browser needn't make dozens of separate requests.

**Q74.** What is tree-shaking?

**Answer:** Removing exported code that is never imported anywhere

Minifying strips whitespace and shortens names; tree-shaking drops unused exports.

**Q75.** In "verify": "npm run lint &amp;&amp; npm run test &amp;&amp; npm run build", what happens if lint fails?

**Answer:** It stops at the first failure and does not run the rest

&amp;&amp; only continues when the previous command succeeds.

**Q76.** Which npm script names can run without the word run?

**Answer:** test and start

npm test and npm start are special shortcuts; others need npm run &lt;name&gt;.

### [10 · Project — Weather Dashboard](level-2/10-project-weather-dashboard.md)

**Q77.** Which API does the weather dashboard project use, and does it need a key?

**Answer:** Open-Meteo, with no API key required

The project uses the free Open-Meteo API.

**Q78.** Why does weather.js contain no DOM code or fetch calls?

**Answer:** So its data transformation can be unit tested with Jest without real network calls

A pure data-transformation module is easy to test.

**Q79.** How should the app behave when the API fails?

**Answer:** Show a readable error instead of a blank page

Graceful failure handling is a stated requirement.

**Q80.** Why do latitude and longitude default to London?

**Answer:** So the form works immediately without a geocoding step

The defaults remove the need for a geocoding lookup.

## Level 3 · Advanced

### [01 · Advanced Async Patterns](level-3/01-advanced-async-patterns.md)

**Q81.** What does Promise.all do if one promise rejects?

**Answer:** Rejects immediately with that error

The other promises keep running, but their results are discarded.

**Q82.** What does Promise.allSettled return?

**Answer:** The outcome of every promise, fulfilled or rejected, never short-circuiting

Use it when partial results are useful, such as checking which services are up.

**Q83.** In loadDashboard, why is the total wait about 50ms rather than 80ms?

**Answer:** Both requests run concurrently via Promise.all

Concurrent execution takes as long as the slowest request.

**Q84.** In what order does Promise.all return results?

**Answer:** The same order the promises were passed in

The array of results matches input order regardless of which finishes first.

### [02 · Node.js Fundamentals](level-3/02-nodejs-fundamentals.md)

**Q85.** Which is true about require() and ES modules?

**Answer:** You can't require() an ES module, but ESM can import most CommonJS packages

The lesson's "mixing systems" section covers this.

**Q86.** How do you get __dirname in an ES module?

**Answer:** Derive it from import.meta.url using fileURLToPath and dirname

__dirname and __filename exist only in CommonJS.

**Q87.** Is the file extension required in ESM imports in Node?

**Answer:** Yes, e.g. ./math.js

In CommonJS the extension is optional; in ES modules it is required.

**Q88.** Why was ESM introduced, per the lesson?

**Answer:** CommonJS require is synchronous and can be called anywhere, whereas ESM has stricter static import rules

require calls can happen conditionally anywhere in a file.

### [03 · Building a REST API with Express](level-3/03-rest-api-express.md)

**Q89.** What does app.use(express.json()) do?

**Answer:** Parses JSON request bodies into req.body

Without it, req.body would not hold the parsed JSON body.

**Q90.** Which HTTP status code does the lesson return when a new note is created?

**Answer:** 201 Created

The POST route responds with res.status(201).json(note).

**Q91.** For the route /notes/:id, how do you read the id?

**Answer:** req.params.id

Route params use req.params; query strings use req.query; JSON bodies use req.body.

**Q92.** A client POSTs to /notes without a text field. What does the lesson's route return?

**Answer:** 400 with an error that text is required

The handler validates input and returns res.status(400).json({ error: "text is required" }).

### [04 · Working with Databases](level-3/04-databases.md)

**Q93.** Why use prepared statements with ? placeholders?

**Answer:** They are the standard defense against SQL injection

Never build SQL strings from user input with template literals.

**Q94.** What is true of better-sqlite3 according to the lesson?

**Answer:** It is synchronous, so no promises are needed

It is also very fast for typical app workloads.

**Q95.** What does an in-memory array lack compared with SQLite?

**Answer:** It does not survive a restart

SQLite stores data in a file on disk, so it persists.

**Q96.** How does MongoDB differ from SQLite in structure?

**Answer:** Flexible documents (schema-optional) rather than fixed columns per table

MongoDB suits rapidly changing or nested data and uses a JSON-like query language.

### [05 · Design Patterns in JavaScript](level-3/05-design-patterns.md)

**Q97.** What is the goal of the module pattern?

**Answer:** Encapsulation: hide internal state and expose a small public interface

ES modules give file-level encapsulation, and closures give it within a file.

**Q98.** What does a singleton guarantee?

**Answer:** Exactly one shared instance with a global access point

In the lesson, connectionA === connectionB is true for the database connection.

**Q99.** Which four patterns does the module cover?

**Answer:** Module, singleton, factory and observer

These are described as four of the most common patterns.

**Q100.** In a module, why is a non-exported variable like count private?

**Answer:** It is unreachable from outside that file

Only exported functions can access or change it.

### [06 · Introduction to TypeScript](level-3/06-intro-typescript.md)

**Q101.** What does TypeScript compile to?

**Answer:** Plain JavaScript

Types only help catch mistakes before running the code.

**Q102.** Why is unknown safer than any?

**Answer:** It must be narrowed (e.g. with typeof) before use

any disables type checking and should be a last resort.

**Q103.** What does ? mean in isAdmin?: boolean?

**Answer:** The property is optional

Optional properties may be omitted without a compile error.

**Q104.** What does an interface describe?

**Answer:** The shape of an object: required properties and their types

TypeScript checks that values used as that type match the shape.

### [07 · Testing Frameworks Deep Dive](level-3/07-testing-deep-dive.md)

**Q105.** Why mock a dependency such as an email service in a test?

**Answer:** Tests stay fast and don't depend on network or external state

A mock replaces the real dependency with a fake version you control.

**Q106.** What does jest.spyOn(logger, "info") do?

**Answer:** Wraps the method so you can assert on calls, optionally keeping the real behavior

Call mockRestore() afterward to restore the original implementation.

**Q107.** Which assertion checks a mock was called with specific arguments?

**Answer:** toHaveBeenCalledWith

toHaveBeenCalledTimes(1) checks how many times it was called.

**Q108.** What is supertest used for in this module?

**Answer:** Integration tests that exercise a real Express API

The module goes beyond unit tests to integration tests against an Express app.

### [08 · Performance &amp; Memory](level-3/08-performance-memory.md)

**Q109.** What is the first rule of performance work?

**Answer:** Measure, don't guess

console.time gives quick rough timings; profilers show real bottlenecks.

**Q110.** What does a memory leak mean in a Node process?

**Answer:** References to objects are kept alive unintentionally, so the garbage collector can't reclaim them

Memory grows over the process's life until it crashes.

**Q111.** Which command lets Chrome DevTools attach for flame charts and heap snapshots?

**Answer:** node --inspect server.js

Then open chrome://inspect and click inspect under Remote Target.

**Q112.** What can high GC time in a profile signal?

**Answer:** Too many short-lived objects being allocated

It points to a memory-usage problem rather than a raw CPU one.

### [09 · Security Basics](level-3/09-security-basics.md)

**Q113.** What is the safe way to insert untrusted text into the page?

**Answer:** Set textContent, which is treated as plain text

textContent is never parsed as HTML, avoiding XSS.

**Q114.** What does CSRF do?

**Answer:** Tricks a logged-in user's browser into submitting a request they didn't intend, using their session cookie

For example, a hidden auto-submitting form on an attacker's page pointed at POST /transfer.

**Q115.** In EJS, which tag is the dangerous unescaped one?

**Answer:** &lt;%- value %&gt;

&lt;%= escapes by default; the raw &lt;%- helper on untrusted input is the danger.

**Q116.** What does npm audit help with?

**Answer:** Catching known-vulnerable dependencies

It is one of the practices covered alongside input sanitization.

### [10 · Build &amp; Bundling Deep Dive](level-3/10-build-bundling.md)

**Q117.** What does Vite use during development vs. production?

**Answer:** Native ES modules in development and Rollup for the production bundle

This gives instant startup in dev and an optimized bundle for production.

**Q118.** Why use [contenthash] in webpack's output filename?

**Answer:** The hash changes when content changes, busting the browser cache

New content gets a new filename, so browsers fetch the update.

**Q119.** What does the Vite server.proxy option for "/api" do?

**Answer:** Forwards API calls to a backend during dev

In the example it forwards to http://localhost:3000.

**Q120.** What is code splitting for?

**Answer:** So users only download what a given page needs

It is one of the techniques that most affects real-world load performance.

### [11 · Project — REST API + Database](level-3/11-project-rest-api-db.md)

**Q121.** Why does the project use an in-memory database (:memory:) for tests?

**Answer:** Tests never touch catalog.db on disk while the same schema and logic are exercised

A real file is used by the running app, :memory: by the tests.

**Q122.** Which route updates an existing book?

**Answer:** PUT /books/:id

The API offers full CRUD: list, get one, create, update and delete.

**Q123.** What does the centralized error-handling middleware achieve?

**Answer:** Errors are handled in one place so nothing crashes the server

The project lists it as a requirement alongside validation.

**Q124.** Which test libraries cover every route in the project?

**Answer:** Jest and supertest

The suite uses Jest with supertest against the Express app.

## Level 4 · Master

### [01 · Full-Stack Architecture Patterns](level-4/01-fullstack-architecture.md)

**Q125.** In the layered architecture, which layer is the only one that knows about SQL or an ORM?

**Answer:** repositories

Routes handle HTTP, services hold business logic, and repositories handle data access.

**Q126.** Why is the users service easy to unit test?

**Answer:** It depends on an abstract repository, so a fake can replace Postgres (dependency injection)

Business logic stays independent of HTTP and SQL.

**Q127.** What is a monorepo?

**Answer:** Multiple related packages in one repository with one history

A polyrepo splits each package into its own repository; neither is universally right.

**Q128.** What should controllers do in this structure?

**Answer:** Act as thin glue translating HTTP to service calls

The controller calls the service and maps results or errors to HTTP responses.

### [02 · Advanced TypeScript](level-4/02-advanced-typescript.md)

**Q129.** What is the benefit of function first&lt;T&gt;(arr: T[]): T | undefined over using any?

**Answer:** The return type is tied to the input type

first([1,2,3]) is number | undefined, while an any version loses type information.

**Q130.** What does Partial&lt;Product&gt; produce?

**Answer:** A type where every field is optional

It is useful for PATCH-style update payloads.

**Q131.** What does Omit&lt;Product, "price"&gt; produce?

**Answer:** Product without the price field

Pick selects named fields; Omit removes them.

**Q132.** What does &lt;T extends HasId&gt; do?

**Answer:** Restricts T to types that have an id

It is a generic constraint that guarantees a known shape.

### [03 · Microservices &amp; Serverless Patterns](level-4/03-microservices-serverless.md)

**Q133.** What is the most common microservices mistake?

**Answer:** Splitting by technical layer instead of business capability

Good boundaries are business capabilities, such as orders, inventory and notifications.

**Q134.** What is a distributed monolith?

**Answer:** Services that can't deploy independently, combining microservices' operational cost with a monolith's coupling

If another team's release can block you, it isn't really an independent service.

**Q135.** What is the trade-off of asynchronous messaging between services?

**Answer:** It decouples availability but adds eventual consistency

Synchronous HTTP/gRPC is simpler but couples availability.

**Q136.** What does serverless mean in this lesson?

**Answer:** A cloud platform runs individual functions on demand

It is one of two paths when a monolith outgrows a single deployable unit.

### [04 · Advanced Security Practices](level-4/04-advanced-security.md)

**Q137.** What is the difference between authentication and authorization?

**Answer:** Authentication verifies who you are; authorization decides what you may do

A valid token doesn't automatically mean the request should be allowed.

**Q138.** Which status code does the lesson's requireRole return when the role doesn't match?

**Answer:** 403 forbidden

401 is used for a missing or invalid token; 403 for lacking permission.

**Q139.** Why are access tokens short-lived (15m)?

**Answer:** JWTs are hard to revoke early, so a short life limits exposure

A JWT can be verified without a database lookup, but can't easily be revoked before expiry.

**Q140.** Where should the JWT signing secret come from?

**Answer:** An environment variable, never hardcoded

The code reads process.env.JWT_ACCESS_SECRET.

### [05 · Testing at Scale](level-4/05-testing-at-scale.md)

**Q141.** What shape does the testing pyramid recommend?

**Answer:** Many unit tests, fewer integration tests and a small number of high-value E2E tests

Covering every click path would be slow and brittle.

**Q142.** What can E2E tests catch that unit tests structurally cannot?

**Answer:** CSS hiding a button or a bug only in Safari

Playwright drives a real browser, so it sees rendered output.

**Q143.** Which tool drives Chromium, Firefox and WebKit in this lesson?

**Answer:** Playwright

Cypress is mentioned as another E2E option.

**Q144.** Why does the config set retries only when process.env.CI is set?

**Answer:** Flaky E2E tests get automatic retries in CI

retries: process.env.CI ? 2 : 0.

### [06 · Scaling Node.js Applications](level-4/06-scaling-nodejs.md)

**Q145.** What does CPU-heavy synchronous work do in Node?

**Answer:** Blocks the entire event loop and stalls every other request

Offload it to a worker thread to keep the event loop free.

**Q146.** What does the cluster module do?

**Answer:** Forks multiple copies of the process, one per CPU core, sharing a port

A crashed worker doesn't take down the whole app.

**Q147.** In the cluster example, what happens when a worker dies?

**Answer:** The primary logs it and forks a replacement

cluster.fork() keeps the pool at full strength.

**Q148.** Why does Node handle many concurrent I/O requests well on one thread?

**Answer:** I/O such as database and HTTP calls is non-blocking

Scaling beyond one core needs clustering, caching and load balancing.

### [07 · Real-Time Applications](level-4/07-realtime-applications.md)

**Q149.** Why are WebSockets better than polling for live features?

**Answer:** One long-lived two-way connection pushes data the instant it's available

Polling wastes requests and adds latency equal to the interval.

**Q150.** In the counter example, why does the server call broadcast?

**Answer:** To push the new value to every connected client, not just the sender

The server loops over all clients whose readyState is OPEN.

**Q151.** What does Socket.IO add on top of WebSockets?

**Answer:** Fallback to HTTP long-polling and extras such as rooms

It falls back to long-polling when needed.

**Q152.** Why does the server check client.readyState === client.OPEN before sending?

**Answer:** To avoid sending to connections that aren't open

Sending on closed sockets would fail.

### [08 · CI/CD for JavaScript Projects](level-4/08-cicd.md)

**Q153.** What is the main difference between Continuous Delivery and Continuous Deployment?

**Answer:** Delivery needs a human to approve the deploy; Deployment ships automatically on merge

Most teams use continuous delivery for production and full deployment for staging or preview.

**Q154.** What does the matrix strategy in the test job do?

**Answer:** Runs tests against multiple Node versions (18, 20, 22)

It verifies the code on each supported Node version.

**Q155.** What does needs: lint do on the test job?

**Answer:** The test job waits for the lint job to succeed first

Jobs form a lint → test → build → deploy chain.

**Q156.** Why does the deploy job have if: github.ref == 'refs/heads/main'?

**Answer:** Only deploy from main, never from PR branches

The production environment can also require manual approval.

### [09 · Observability](level-4/09-observability.md)

**Q157.** Why use structured (JSON) logs instead of free-form text?

**Answer:** Log aggregators can filter and query them reliably

Examples of aggregators are Datadog, CloudWatch and Elastic.

**Q158.** How does winston differ from pino in the lesson?

**Answer:** More configurable, with multiple transports, at the cost of more overhead

pino is very low-overhead; winston can write to Console and File and other transports at once.

**Q159.** What does req.log.info({ orderId }, "fetching order") demonstrate?

**Answer:** Logging structured fields instead of string concatenation

pino-http attaches request-scoped logging.

**Q160.** Why can't you rely on a debugger in production across multiple instances?

**Answer:** You can't attach and step through a running system, so you use logs, metrics and health checks

Observability explains what a running system is doing without interrupting it.

### [10 · Capstone Project](level-4/10-capstone-project.md)

**Q161.** What is the capstone project?

**Answer:** A Task Tracker API with JWT-protected REST routes backed by SQLite

Users register, log in and manage their own tasks.

**Q162.** Why can the database be swapped for Postgres or MySQL without changing business logic?

**Answer:** The repository layer is the only place that knows about SQL

The layering keeps business logic independent of the data store.

**Q163.** How does config.js choose the database path under test?

**Answer:** Uses :memory: when NODE_ENV is test

Otherwise it uses ./data/tasks.db unless DB_PATH is set.

**Q164.** Which pieces does the capstone bring together?

**Answer:** REST API, data layer, JWT auth, tests, container image and CI pipeline

It is the final production-shaped project of the whole JavaScript course.

## Mixed questions across all levels

**Q165.** What does this print?

```
console.log(typeof null, typeof undefined, 5 === "5");
```

**Answer:** object undefined false

typeof null is the long-standing quirk "object"; === checks type, so 5 === "5" is false.

**Q166.** What does this print?

```
const o = { a: 1 };
const { a, b = 5 } = o;
console.log(a, b);
```

**Answer:** 1 5

Destructuring takes a from the object and falls back to the default 5 for the missing b.

**Q167.** What does this print?

```
function mk() { let c = 0; return () => ++c; }
const f = mk(); f();
console.log(f());
```

**Answer:** 2

The returned function closes over c, so the second call returns 2.

**Q168.** What does this print?

```
console.log([1,2,3,4,5].filter(n => n % 2).reduce((s, n) => s + n, 0));
```

**Answer:** 9

filter keeps 1, 3, 5, and reduce sums them to 9.

**Q169.** What do Number("42px") and parseInt("42px") return?

**Answer:** NaN and 42

Number must parse the whole string; parseInt reads the leading digits and ignores the rest.

**Q170.** What is the order of the output?

```
setTimeout(() => console.log("t"), 0);
Promise.resolve().then(() => console.log("p"));
console.log("s");
```

**Answer:** s, p, t

Synchronous code first, then the microtask queue (promise callbacks), then the task queue (timers).

**Q171.** What does this print?

```
console.log(null ?? "d", 0 ?? "d", 0 || "d");
```

**Answer:** d 0 d

?? only falls back for null or undefined, so 0 is kept; || falls back for any falsy value, including 0.

**Q172.** What does [10, 9, 1].sort() return?

**Answer:** [1, 10, 9]

With no comparison function, sort compares elements as strings, so "1" &lt; "10" &lt; "9".

**Q173.** What does this print?

```
class A { speak() { return "a"; } }
class B extends A { speak() { return super.speak() + "b"; } }
console.log(new B().speak());
```

**Answer:** ab

super.speak() calls the parent's method, then the subclass adds "b".

**Q174.** What does JSON.stringify({ x: undefined, y: () =&gt; 1, z: null, d: 1 }) return?

**Answer:** {"z":null,"d":1}

Functions and undefined values are omitted; null is kept.

**Q175.** What do these two loops print?

```
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i), 0);
for (let j = 0; j < 3; j++) setTimeout(() => console.log(j), 0);
```

**Answer:** 3 3 3 then 0 1 2

var is function-scoped, so all callbacks see the final i (3); let creates a new j for each iteration.

**Q176.** You need to run three independent API calls and use all results. Which is best?

**Answer:** await Promise.all([a(), b(), c()])

Promise.all runs them concurrently, so the total wait is the slowest call, not the sum.

**Q177.** A service has to report which of five health checks failed without stopping at the first failure. Which should you use?

**Answer:** Promise.allSettled

allSettled waits for everything and reports each outcome; all rejects on the first failure.

**Q178.** A user types in a comment box and you insert their text into the page. Which is the safe way?

**Answer:** element.textContent = comment

textContent treats the text as plain text, which prevents XSS; innerHTML would parse injected markup.

**Q179.** An Express route receives POST /notes with no `text` field in the body. What should a well-designed handler return?

**Answer:** 400 with an error message

Validate input and return a 400 Bad Request; 201 is for successful creation.

**Q180.** Why do the lessons use prepared statements with ? placeholders for database queries?

**Answer:** To prevent SQL injection

Values are passed as data, never concatenated into the SQL string.

**Q181.** Your Node server freezes every other request while one endpoint runs a 10-second CPU-heavy loop. What is the cause and the fix?

**Answer:** It blocks the single-threaded event loop; offload it to a worker thread

CPU-bound work blocks the event loop; worker_threads (or clustering) keeps it free.

**Q182.** A JWT access token is short-lived (15 minutes) on purpose. Why?

**Answer:** JWTs are hard to revoke, so a short life limits the damage if one leaks

A JWT is verified without a database lookup, so you can't easily revoke it before it expires.

**Q183.** In a Jest test, how do you verify that a function was called with specific arguments?

**Answer:** expect(mockFn).toHaveBeenCalledWith(...)

toHaveBeenCalledWith checks a mock's call arguments; toHaveBeenCalledTimes checks the count.

**Q184.** Which package.json setting lets plain .js files use import/export in Node?

**Answer:** "type": "module"

Otherwise .js files are CommonJS by default unless you rename to .mjs.

## Practical coding questions

### Group words by length

Write `groupByLength(words)` that returns an object whose keys are word lengths and whose values are arrays of the words with that length, in input order. Print the result with `JSON.stringify`.

Starter code:

```
function groupByLength(words) {
  // your code here
}
console.log(JSON.stringify(groupByLength(["to", "be", "or", "not", "bee"])));
```

Expected output:

```text
{"2":["to","be","or"],"3":["not","bee"]}
```

??? success "Show a solution"

    ```
    function groupByLength(words) {
      const out = {};
      for (const w of words) (out[w.length] ??= []).push(w);
      return out;
    }
    console.log(JSON.stringify(groupByLength(["to", "be", "or", "not", "bee"])));
    ```

    `??=` creates the array the first time a length is seen, then push adds the word. Object keys are strings, so the lengths print as "2" and "3".

### Run promises in parallel and handle an error

Use `Promise.all` to wait for three delayed values (`a`, `b`, `c`) and print them joined by commas in the original order, then show that `await` on a rejected promise can be caught with try/catch.

Starter code:

```
const delay = (ms, v) => new Promise((r) => setTimeout(() => r(v), ms));
async function main() {
  // 1. await Promise.all([...]) with delays 30, 10, 20ms for "a", "b", "c"; print the results joined with ","
  // 2. try { await Promise.reject(new Error("boom")); } catch ... print "caught boom"
}
main();
```

Expected output:

```text
a,b,c
caught boom
```

??? success "Show a solution"

    ```
    const delay = (ms, v) => new Promise((r) => setTimeout(() => r(v), ms));
    async function main() {
      const results = await Promise.all([delay(30, "a"), delay(10, "b"), delay(20, "c")]);
      console.log(results.join(","));
      try { await Promise.reject(new Error("boom")); } catch (e) { console.log("caught", e.message); }
    }
    main();
    ```

    Promise.all keeps results in the order the promises were passed, even though b finishes first. try/catch around await catches rejected promises.

---

More free practice: the [JavaScript Mastery Path](index.md) curriculum, the [Interview Prep Pack](https://learnmasterypath.com/interview-pack/) and the [free resume ATS check](https://learnmasterypath.com/resume/).
