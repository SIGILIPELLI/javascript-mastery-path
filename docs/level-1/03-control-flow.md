# 03 · Control Flow

## if / else if / else

```javascript
const score = 82;
let grade;

if (score >= 90) {
  grade = "A";
} else if (score >= 80) {
  grade = "B";
} else if (score >= 70) {
  grade = "C";
} else {
  grade = "F";
}

console.log(grade); // B
```

## The ternary operator

```javascript
const age = 20;
const canVote = age >= 18 ? "yes" : "no";
console.log(canVote); // yes
```

## while and do...while loops

```javascript
let count = 0;
while (count < 5) {
  console.log(count);
  count++;
}

let n = 0;
do {
  console.log("runs at least once:", n);
  n++;
} while (n < 0); // condition is false, but the body already ran once
```

## for loops

```javascript
for (let i = 0; i < 5; i++) {
  console.log(i); // 0, 1, 2, 3, 4
}

const fruits = ["apple", "banana", "cherry"];

// for...of iterates over values — preferred for arrays
for (const fruit of fruits) {
  console.log(fruit);
}

// for...in iterates over keys/indexes — mainly for objects
for (const index in fruits) {
  console.log(index, fruits[index]);
}
```

## break, continue

```javascript
for (let n = 2; n < 20; n++) {
  if (n % 7 === 0) {
    console.log(`first multiple of 7: ${n}`);
    break;
  }
}

for (let n = 0; n < 10; n++) {
  if (n % 2 !== 0) {
    continue; // skip odd numbers
  }
  console.log(n);
}
```

## switch statement

```javascript
function describeDay(day) {
  switch (day) {
    case "Sat":
    case "Sun":
      return "weekend";
    case "Mon":
    case "Tue":
    case "Wed":
    case "Thu":
    case "Fri":
      return "weekday";
    default:
      return "unknown";
  }
}

console.log(describeDay("Sat")); // weekend
console.log(describeDay("Wed")); // weekday
```

`switch` uses strict equality (`===`) to compare and falls through to the next
`case` unless you `break` or `return` — grouping cases without a `break`
(as with `"Sat"`/`"Sun"` above) is a common, intentional pattern.

## Exercise

Write a program that prints FizzBuzz for numbers 1–30: multiples of 3 print
"Fizz", multiples of 5 print "Buzz", multiples of both print "FizzBuzz",
otherwise print the number.
