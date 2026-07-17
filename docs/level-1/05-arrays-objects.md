# 05 · Arrays & Objects

## Arrays — ordered, mutable lists

```javascript
const fruits = ["apple", "banana", "cherry"];

fruits.push("date");        // add to end
fruits.unshift("avocado");  // add to start
fruits.splice(2, 1);         // remove 1 item starting at index 2
const last = fruits.pop();   // remove & return last item

console.log(fruits[0]);       // first item
console.log(fruits.at(-1));   // last item (modern, cleaner than fruits[fruits.length - 1])
console.log(fruits.slice(1, 3)); // shallow copy of a range, doesn't mutate
console.log(fruits.length);
```

## Common array methods

```javascript
const numbers = [1, 2, 3, 4, 5];

const doubled = numbers.map((n) => n * 2);          // [2, 4, 6, 8, 10]
const evens = numbers.filter((n) => n % 2 === 0);    // [2, 4]
const sum = numbers.reduce((total, n) => total + n, 0); // 15
const found = numbers.find((n) => n > 3);             // 4
const hasNegative = numbers.some((n) => n < 0);        // false
const allPositive = numbers.every((n) => n > 0);       // true

numbers.forEach((n) => console.log(n)); // side-effect iteration, no return value

console.log([...numbers].sort((a, b) => b - a)); // [5, 4, 3, 2, 1] — sort a copy!
console.log(numbers.includes(3)); // true
console.log(numbers.join(", ")); // "1, 2, 3, 4, 5"
```

`map`, `filter`, and `reduce` all return **new** arrays or values without
mutating the original — prefer them over manual loops for transformations.

## Destructuring and the spread operator

```javascript
const [first, second, ...rest] = fruits;
console.log(first, second, rest); // avocado banana [ 'cherry', 'date' ]

const combined = [...fruits, "elderberry"]; // spread into a new array
```

## Objects — key/value pairs

```javascript
const person = { name: "Ada", age: 30 };

person.email = "ada@example.com";  // add/update via dot notation
person["age"] = 31;                 // bracket notation — needed for dynamic keys
delete person.age;                   // remove a key

for (const key in person) {
  console.log(key, person[key]);
}

for (const [key, value] of Object.entries(person)) {
  console.log(key, value);
}

console.log(Object.keys(person));   // ['name', 'email']
console.log(Object.values(person)); // ['Ada', 'ada@example.com']
```

## Object destructuring and shorthand

```javascript
const { name, email } = person;
console.log(name, email); // Ada ada@example.com

const city = "London";
const country = "UK";
const address = { city, country }; // shorthand: same as { city: city, country: country }

function describe({ name, age = 0 }) { // destructure directly in parameters
  return `${name} (${age})`;
}
console.log(describe({ name: "Grace" })); // Grace (0)
```

## Spread and optional chaining

```javascript
const base = { role: "user" };
const admin = { ...base, role: "admin", canDelete: true }; // spread + override

const nested = { profile: { social: null } };
console.log(nested.profile?.social?.twitter); // undefined, no error — optional chaining
console.log(nested.profile?.social?.twitter ?? "none"); // "none" — nullish coalescing
```

## Choosing the right structure

| Need | Use |
|------|-----|
| Ordered list, index-based access | `Array` |
| Key/value pairs, known field names | plain `Object` |
| Frequent additions/removals by key, guaranteed insertion order | `Map` |
| Unique values, fast membership tests | `Set` |

```javascript
const uniqueTags = new Set(["js", "web", "js"]);
console.log(uniqueTags); // Set(2) { 'js', 'web' }

const scoreboard = new Map();
scoreboard.set("Ada", 10);
scoreboard.set("Grace", 15);
console.log(scoreboard.get("Ada")); // 10
```

## Exercise

Given an array of words, use `.reduce()` to build an object mapping each
unique word to its length, then use a `Set` to find which words appear more
than once in the original array.
