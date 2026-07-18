# 02 · Advanced TypeScript

TypeScript's type system is Turing-complete in practice — you can express
remarkably precise constraints about your data and catch entire classes of
bugs before running a single test. This module covers the features that show
up constantly in production TypeScript codebases.

## Generics

Generics let a function, type, or class work with any type while preserving
the relationship between its inputs and outputs.

```typescript
// Without generics — loses type information
function firstUnsafe(arr: any[]): any {
  return arr[0];
}

// With generics — the return type is tied to the input type
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

const n = first([1, 2, 3]);       // n: number | undefined
const s = first(["a", "b"]);       // s: string | undefined
```

```typescript
// Generic constraints — restrict T to types with a known shape
interface HasId {
  id: string;
}

function findById<T extends HasId>(items: T[], id: string): T | undefined {
  return items.find((item) => item.id === id);
}

interface User extends HasId {
  name: string;
}

const users: User[] = [{ id: "1", name: "Ada" }];
const found = findById(users, "1"); // found: User | undefined
```

```typescript
// Generic classes — a type-safe cache keyed by string
class Cache<T> {
  private store = new Map<string, T>();

  set(key: string, value: T): void {
    this.store.set(key, value);
  }

  get(key: string): T | undefined {
    return this.store.get(key);
  }
}

const userCache = new Cache<User>();
userCache.set("1", { id: "1", name: "Ada" });
```

## Utility types

TypeScript ships built-in utility types that transform existing types instead
of redeclaring them.

```typescript
interface Product {
  id: string;
  name: string;
  price: number;
  description?: string;
}

type ProductPreview = Pick<Product, "id" | "name">;
// { id: string; name: string }

type ProductWithoutPrice = Omit<Product, "price">;
// { id: string; name: string; description?: string }

type PartialProduct = Partial<Product>;
// every field optional — useful for PATCH-style update payloads

type RequiredProduct = Required<Product>;
// every field mandatory — description is no longer optional

type ReadonlyProduct = Readonly<Product>;
// all fields cannot be reassigned after creation

type ProductPriceMap = Record<string, number>;
// { [key: string]: number } — e.g. { "sku-1": 19.99, "sku-2": 24.99 }
```

```typescript
function updateProduct(id: string, changes: Partial<Product>): Product {
  const existing = getProductById(id); // pretend this exists
  return { ...existing, ...changes };
}

updateProduct("p1", { price: 29.99 }); // only price needs to be supplied
```

## Conditional types

Conditional types choose between two types based on a check, similar to a
ternary operator but evaluated at the type level.

```typescript
type IsString<T> = T extends string ? "yes" : "no";

type A = IsString<string>; // "yes"
type B = IsString<number>; // "no"
```

```typescript
// Extracting the return type of a function, without calling it
type ReturnOf<Fn> = Fn extends (...args: any[]) => infer R ? R : never;

function getUser() {
  return { id: "1", name: "Ada" };
}

type User = ReturnOf<typeof getUser>; // { id: string; name: string }
```

```typescript
// A conditional type that unwraps a Promise
type Awaited2<T> = T extends Promise<infer V> ? V : T;

type A1 = Awaited2<Promise<number>>; // number
type A2 = Awaited2<string>;          // string (unchanged, not a Promise)
```

`infer` introduces a new type variable inside a conditional type — it lets you
pull a piece out of a more complex type (a return type, a Promise's resolved
value, an array's element type) instead of restating it manually.

## Mapped types

Mapped types build a new object type by iterating over the keys of an
existing one — this is how `Partial`, `Required`, and `Readonly` are actually
implemented under the hood.

```typescript
// A simplified version of what lib.es5.d.ts defines for Partial<T>
type MyPartial<T> = {
  [K in keyof T]?: T[K];
};

// Mapped type with a transformation: wrap every field in a validator function
type Validators<T> = {
  [K in keyof T]: (value: T[K]) => boolean;
};

interface Form {
  email: string;
  age: number;
}

const formValidators: Validators<Form> = {
  email: (value) => value.includes("@"),
  age: (value) => value >= 0,
};
```

```typescript
// Key remapping with `as` — rename keys while mapping (TS 4.1+)
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type ProductGetters = Getters<Product>;
// { getId: () => string; getName: () => string; getPrice: () => number; ... }
```

## Discriminated unions and exhaustiveness checks

A discriminated union uses a shared literal field (often called `type` or
`kind`) so TypeScript can narrow which shape you're working with — and the
compiler can force you to handle every case.

```typescript
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "rectangle"; width: number; height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "rectangle":
      return shape.width * shape.height;
    default:
      // if a new Shape variant is added and not handled above,
      // this line fails to compile — `shape` can't be `never`
      const exhaustive: never = shape;
      throw new Error(`unhandled shape: ${exhaustive}`);
  }
}
```

## Template literal types

Template literal types build string types out of other types, useful for
typed route paths, CSS-in-JS keys, or event names.

```typescript
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type Resource = "users" | "orders";

type Route = `${HttpMethod} /${Resource}`;
// "GET /users" | "GET /orders" | "POST /users" | "POST /orders" | ...

function request(route: Route) {
  /* ... */
}

request("GET /users");   // OK
// request("PATCH /users"); // Error: "PATCH /users" is not assignable to Route
```

## Utility types cheat sheet

| Utility | Purpose |
|---|---|
| `Partial<T>` | Every field becomes optional |
| `Required<T>` | Every field becomes mandatory |
| `Pick<T, K>` | Keep only keys `K` |
| `Omit<T, K>` | Remove keys `K` |
| `Record<K, V>` | Object type with keys `K` mapped to values `V` |
| `ReturnType<Fn>` | The return type of a function type |
| `Awaited<T>` | The resolved value type of a Promise |
| `NonNullable<T>` | Removes `null`/`undefined` from a union |

## Exercise

Given this base type:

```typescript
interface Task {
  id: string;
  title: string;
  status: "todo" | "in-progress" | "done";
  assigneeId?: string;
}
```

Write: (1) a `CreateTaskInput` type derived from `Task` via utility types that
omits `id` and makes `status` optional (defaulting elsewhere to `"todo"`);
(2) a generic `Result<T>` discriminated union type with two variants,
`{ ok: true; value: T }` and `{ ok: false; error: string }`; and (3) a function
`updateTaskStatus(task: Task, status: Task["status"]): Result<Task>` that
returns an error result if attempting to move a `"done"` task back to
`"todo"`.
