# 05 · Design Patterns in JavaScript

Design patterns are reusable solutions to recurring problems in software
design. JavaScript's flexible object model lets you implement classic
patterns in a few different ways — this module covers four of the most
common: module, singleton, factory, and observer.

## Module pattern

The goal of the module pattern is **encapsulation** — hiding internal state
and exposing only a small public interface. ES modules (`export`/`import`)
give you this for free at the file level, but the pattern also applies
inside a single file using closures.

```javascript
// counter.js — ES module version: file-level encapsulation
let count = 0; // private — not exported, unreachable from outside this file

export function increment() {
  count += 1;
  return count;
}

export function reset() {
  count = 0;
}

export function getCount() {
  return count;
}
```

```javascript
// Closure version: encapsulation without separate files
function createCounter() {
  let count = 0; // private state, captured by the closures below

  return {
    increment() {
      count += 1;
      return count;
    },
    reset() {
      count = 0;
    },
    getCount() {
      return count;
    },
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.getCount());  // 2
// counter.count is not accessible — count only exists inside the closure
```

## Singleton pattern

A singleton ensures a class or module has exactly **one** shared instance,
and provides a global point of access to it — useful for things like a
single database connection pool or a shared configuration object.

```javascript
// connection.js
class DatabaseConnection {
  constructor() {
    if (DatabaseConnection.instance) {
      return DatabaseConnection.instance; // return the existing instance instead of a new one
    }
    this.connectedAt = new Date();
    this.queryCount = 0;
    DatabaseConnection.instance = this;
  }

  query(sql) {
    this.queryCount += 1;
    return `executed: ${sql} (total queries: ${this.queryCount})`;
  }
}

const connectionA = new DatabaseConnection();
const connectionB = new DatabaseConnection();

console.log(connectionA === connectionB); // true — same instance
console.log(connectionA.query("SELECT 1")); // executed: SELECT 1 (total queries: 1)
console.log(connectionB.query("SELECT 2")); // executed: SELECT 2 (total queries: 2) — shared state
```

Because ES modules are cached after their first import, a module that
exports a single created instance is a simpler, idiomatic way to get
singleton behavior in Node:

```javascript
// config.js — the module itself acts as the singleton
const config = { env: "production", version: "1.0.0" };
export default config; // every importer receives the exact same object
```

## Factory pattern

A factory function creates and returns objects without exposing the
`new`/class machinery to the caller — useful when object creation involves
branching logic or you want to hide the concrete type being constructed.

```javascript
function createNotification(type, message) {
  const base = { message, createdAt: new Date() };

  switch (type) {
    case "email":
      return { ...base, kind: "email", send: () => `Emailing: ${message}` };
    case "sms":
      return { ...base, kind: "sms", send: () => `Texting: ${message}` };
    case "push":
      return { ...base, kind: "push", send: () => `Pushing: ${message}` };
    default:
      throw new Error(`unknown notification type: ${type}`);
  }
}

const email = createNotification("email", "Your order shipped");
const sms = createNotification("sms", "Your code is 4821");

console.log(email.send()); // Emailing: Your order shipped
console.log(sms.send());   // Texting: Your code is 4821
console.log(email.kind);   // email
```

Factories are especially handy alongside classes, when you want the
constructor logic centralized but still hide class selection from callers:

```javascript
class EmailNotification {
  constructor(message) { this.message = message; }
  send() { return `Emailing: ${this.message}`; }
}

class SmsNotification {
  constructor(message) { this.message = message; }
  send() { return `Texting: ${this.message}`; }
}

function notificationFactory(type, message) {
  const classes = { email: EmailNotification, sms: SmsNotification };
  const NotificationClass = classes[type];
  if (!NotificationClass) throw new Error(`unknown type: ${type}`);
  return new NotificationClass(message);
}

console.log(notificationFactory("sms", "Reminder: meeting at 3pm").send());
// Texting: Reminder: meeting at 3pm
```

## Observer pattern

The observer pattern lets one object (the **subject**) notify a list of
dependent objects (**observers**) whenever something happens, without the
subject knowing any details about them. This is the foundation of DOM
events, Node's `EventEmitter`, and pub/sub systems in general.

```javascript
class EventBus {
  constructor() {
    this.listeners = {}; // event name -> array of callback functions
  }

  on(event, callback) {
    if (!this.listeners[event]) this.listeners[event] = [];
    this.listeners[event].push(callback);
  }

  off(event, callback) {
    if (!this.listeners[event]) return;
    this.listeners[event] = this.listeners[event].filter((cb) => cb !== callback);
  }

  emit(event, payload) {
    (this.listeners[event] ?? []).forEach((callback) => callback(payload));
  }
}

const bus = new EventBus();

function logOrder(order) {
  console.log(`order placed: #${order.id} for $${order.total}`);
}

function emailReceipt(order) {
  console.log(`emailing receipt for order #${order.id}`);
}

bus.on("order:placed", logOrder);
bus.on("order:placed", emailReceipt);

bus.emit("order:placed", { id: 101, total: 49.99 });
// order placed: #101 for $49.99
// emailing receipt for order #101

bus.off("order:placed", emailReceipt);
bus.emit("order:placed", { id: 102, total: 12.5 });
// order placed: #102 for $12.5
// (emailReceipt no longer fires)
```

Node's built-in `EventEmitter` (from `node:events`) is a ready-made,
battle-tested implementation of exactly this pattern:

```javascript
import { EventEmitter } from "node:events";

const emitter = new EventEmitter();

emitter.on("greet", (name) => console.log(`Hello, ${name}!`));
emitter.emit("greet", "Ada"); // Hello, Ada!
```

## Pattern cheat sheet

| Pattern | Problem it solves | JS building block |
|---------|--------------------|--------------------|
| Module | hide internal state, expose a small API | ES modules / closures |
| Singleton | exactly one shared instance | cached module export / instance check |
| Factory | centralize object creation logic | a function returning different objects/classes |
| Observer | decouple "something happened" from "what to do about it" | listener arrays / `EventEmitter` |

## Exercise

Build a small `TaskQueue` using the observer pattern: it should support
`.on("task:completed", callback)` and `.on("task:failed", callback)`, plus a
method `runTask(fn)` that calls `fn()`, catches any thrown error, and emits
the appropriate event with the result or the error. Then wrap creation of
`TaskQueue` instances in a factory function `createQueue(type)` that returns
either a "verbose" queue (logs every event to the console) or a "silent"
one (no logging), demonstrating factory + observer working together.
