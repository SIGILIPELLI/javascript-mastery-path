# 06 · Introduction to TypeScript

TypeScript is JavaScript with an optional static type system layered on
top. It compiles down to plain JavaScript, so anything TypeScript can
express, the browser or Node ultimately runs as regular JS — the types only
help you catch mistakes *before* running the code.

## Installing and running TypeScript

```bash
npm install --save-dev typescript
npx tsc --init   # generates tsconfig.json
```

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "strict": true,
    "outDir": "dist",
    "esModuleInterop": true
  }
}
```

```bash
npx tsc            # compiles all .ts files according to tsconfig.json
node dist/main.js   # run the compiled JavaScript output
```

## Basic types

```typescript
let age: number = 30;
let name: string = "Ada";
let isActive: boolean = true;
let tags: string[] = ["admin", "editor"];   // array of strings
let scores: number[] = [10, 20, 30];

// Tuple: a fixed-length array with known types per position
let point: [number, number] = [3, 4];

// any disables type checking — avoid it; escape hatch of last resort
let anything: any = "could be anything";

// unknown is the safer version of any — must be narrowed before use
let value: unknown = "hello";
if (typeof value === "string") {
  console.log(value.toUpperCase()); // OK — TypeScript knows it's a string here
}
```

## Function types

```typescript
function add(a: number, b: number): number {
  return a + b;
}

// Optional parameter with `?`, default parameter, and a void return type
function greet(name: string, greeting?: string): void {
  console.log(`${greeting ?? "Hello"}, ${name}!`);
}

greet("Ada");           // Hello, Ada!
greet("Ada", "Welcome"); // Welcome, Ada!

// add("2", 3); // Compile error: Argument of type 'string' is not assignable to type 'number'
```

## Interfaces

An `interface` describes the **shape** of an object — which properties it
must have and their types. TypeScript checks that any value used as that
type actually matches the shape.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  isAdmin?: boolean; // optional property
}

function printUser(user: User): string {
  return `${user.name} <${user.email}>${user.isAdmin ? " (admin)" : ""}`;
}

const ada: User = { id: 1, name: "Ada", email: "ada@example.com" };
console.log(printUser(ada)); // Ada <ada@example.com>

// const bad: User = { id: 2, name: "Missing Email" };
// Compile error: Property 'email' is missing in type '{ id: number; name: string; }'
```

## Interfaces for function shapes and extension

```typescript
interface Comparator {
  (a: number, b: number): number; // callable shape: a function taking two numbers
}

const ascending: Comparator = (a, b) => a - b;
console.log([3, 1, 2].sort(ascending)); // [1, 2, 3]

interface Animal {
  name: string;
}

interface Dog extends Animal { // extend: Dog has everything Animal has, plus breed
  breed: string;
}

const rex: Dog = { name: "Rex", breed: "Labrador" };
console.log(rex.name, rex.breed); // Rex Labrador
```

## type aliases vs. interfaces

```typescript
type ID = number | string;         // union type: either a number or a string
type Status = "pending" | "active" | "closed"; // string literal union — an enum-like set

function findById(id: ID): void {
  console.log(`looking up ${id}`);
}

findById(42);     // OK
findById("abc");  // OK — ID allows both
// findById(true); // Compile error

let orderStatus: Status = "pending";
// orderStatus = "cancelled"; // Compile error: not assignable to type 'Status'
```

| | `interface` | `type` |
|---|---|---|
| Object shapes | yes | yes |
| Unions (`A \| B`) | no | yes |
| Extending/merging | `extends`, declaration merging | `&` intersections |
| Typical use | public object/class shapes | unions, aliases, utility types |

## Migrating a JavaScript file to TypeScript

`cart.js` — the plain JavaScript version:

```javascript
// cart.js
function createCart() {
  const items = [];

  function addItem(name, price, quantity = 1) {
    items.push({ name, price, quantity });
  }

  function total() {
    return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }

  return { addItem, total, items };
}

const cart = createCart();
cart.addItem("Book", 12.99, 2);
console.log(cart.total()); // 25.98
```

`cart.ts` — the same logic with types added incrementally:

```typescript
// cart.ts
interface CartItem {
  name: string;
  price: number;
  quantity: number;
}

interface Cart {
  addItem(name: string, price: number, quantity?: number): void;
  total(): number;
  items: CartItem[];
}

function createCart(): Cart {
  const items: CartItem[] = [];

  function addItem(name: string, price: number, quantity: number = 1): void {
    items.push({ name, price, quantity });
  }

  function total(): number {
    return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }

  return { addItem, total, items };
}

const cart: Cart = createCart();
cart.addItem("Book", 12.99, 2);
console.log(cart.total()); // 25.98

// cart.addItem("Pen", "1.50"); // Compile error: '"1.50"' is not assignable to parameter of type 'number'
```

Notice what changed: an `interface` for each shape (`CartItem`, `Cart`), a
return type on each function, and parameter types — the runtime behavior is
identical, but typos or wrong argument types are now caught by `tsc` before
the code ever runs.

## Exercise

Take a plain JavaScript file that manages a simple `Library` (an array of
`{ title, author, available }` book objects with `addBook`, `checkOut`, and
`returnBook` functions) and rewrite it as TypeScript: define a `Book`
interface and a `Library` interface describing the object's shape, add
parameter/return types to every function, and introduce a `Status` union
type (`"available" | "checked-out"`) used instead of the boolean
`available` flag.
