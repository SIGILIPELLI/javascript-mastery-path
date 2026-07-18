# 02 · Prototypes & Classes

## Every object has a prototype

JavaScript objects link to another object called their prototype, and look
up missing properties there. This chain is how methods like `.toString()`
"just work" on objects you never defined them on.

```javascript
const animal = {
  eats: true,
  describe() {
    return "an animal that eats";
  },
};

const rabbit = Object.create(animal); // rabbit's prototype is `animal`
rabbit.jumps = true;

console.log(rabbit.jumps);        // true — own property
console.log(rabbit.eats);         // true — found on the prototype
console.log(rabbit.describe());   // an animal that eats — inherited method
console.log(Object.getPrototypeOf(rabbit) === animal); // true
```

## Constructor functions (the pre-`class` way)

Before `class` syntax, functions combined with `new` and `.prototype` were
the standard way to create many objects that share methods.

```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
}

// Methods go on the prototype so every instance shares ONE copy in memory
Person.prototype.greet = function () {
  return `Hi, I'm ${this.name}`;
};

const ada = new Person("Ada", 30);
const grace = new Person("Grace", 45);

console.log(ada.greet());   // Hi, I'm Ada
console.log(grace.greet()); // Hi, I'm Grace
console.log(ada.greet === grace.greet); // true — same function, shared via prototype
```

## `class` syntax

`class` is syntactic sugar over the same prototype mechanism — cleaner to
read, but it compiles down to functions and prototypes under the hood.

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    return `${this.name} makes a sound.`;
  }
}

const dog = new Animal("Rex");
console.log(dog.speak()); // Rex makes a sound.
console.log(dog instanceof Animal); // true
console.log(typeof Animal); // function — classes are functions under the hood
```

## Inheritance with `extends` and `super`

`extends` sets up the prototype chain between two classes; `super` calls the
parent class's constructor or methods.

```javascript
class Dog extends Animal {
  constructor(name, breed) {
    super(name); // must run before using `this` — calls Animal's constructor
    this.breed = breed;
  }

  speak() {
    const base = super.speak(); // call the parent version, then extend it
    return `${base} Specifically, a bark! (${this.breed})`;
  }
}

const rex = new Dog("Rex", "Labrador");
console.log(rex.speak());
// Rex makes a sound. Specifically, a bark! (Labrador)
console.log(rex instanceof Dog);    // true
console.log(rex instanceof Animal); // true — Dog's chain includes Animal
```

## Getters, setters, and private fields

Classes support computed properties via `get`/`set`, and true private state
using a `#` prefix (not accessible from outside the class at all).

```javascript
class BankAccount {
  #balance; // private field — a syntax error to access as account.#balance outside

  constructor(initialBalance = 0) {
    this.#balance = initialBalance;
  }

  get balance() {
    return this.#balance;
  }

  deposit(amount) {
    if (amount <= 0) throw new Error("deposit must be positive");
    this.#balance += amount;
  }
}

const account = new BankAccount(100);
account.deposit(50);
console.log(account.balance); // 150 — read through the getter
// console.log(account.#balance); // SyntaxError if written outside the class
```

## Static members

`static` properties and methods belong to the class itself, not to
instances — useful for factory methods or shared utilities.

```javascript
class Point {
  static origin = new Point(0, 0); // shared "constant" instance

  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  static distance(a, b) {
    return Math.sqrt((a.x - b.x) ** 2 + (a.y - b.y) ** 2);
  }
}

const p1 = new Point(0, 0);
const p2 = new Point(3, 4);

console.log(Point.distance(p1, p2)); // 5
console.log(Point.origin.x, Point.origin.y); // 0 0
```

## Prototype chain vs. class syntax

| Concept | Constructor function style | `class` style |
|---------|------------------------------|----------------|
| Create instance | `new Person(...)` | `new Person(...)` |
| Shared methods | `Person.prototype.method = ...` | defined inside the class body |
| Inheritance | manual `Object.create`/`.call()` wiring | `extends` + `super` |
| Private state | closures or naming conventions (`_balance`) | `#field` (enforced by the engine) |
| Underlying mechanism | prototype chain | same prototype chain, nicer syntax |

## Exercise

Create a `Shape` base class with a constructor field `name` and a method
`describe()` returning `"<name> has an area of <area>"`. Then create
`Rectangle` and `Circle` subclasses that `extend Shape`, each implementing
its own `get area()` getter (Rectangle: `width * height`; Circle:
`Math.PI * radius ** 2`, rounded to 2 decimals). Verify `describe()` works
correctly for both.
