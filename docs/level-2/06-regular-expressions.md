# 06 · Regular Expressions in JS

## RegExp literals

A regular expression is a pattern for matching text. JavaScript has a
built-in literal syntax using slashes, plus flags that change how the
pattern behaves.

```javascript
const hasDigit = /\d/;       // matches any single digit
const wordChars = /\w+/;      // matches one or more word characters
const caseInsensitive = /hello/i; // 'i' flag ignores case
const global = /a/g;           // 'g' flag finds ALL matches, not just the first

console.log(hasDigit.test("abc123")); // true
console.log(hasDigit.test("abc"));    // false
console.log(caseInsensitive.test("HELLO world")); // true
```

## Common pattern building blocks

| Syntax | Meaning |
|--------|---------|
| `\d` | a digit (0-9) |
| `\w` | a word character (letters, digits, underscore) |
| `\s` | whitespace (space, tab, newline) |
| `.` | any character except a newline |
| `+` | one or more of the previous token |
| `*` | zero or more of the previous token |
| `?` | zero or one of the previous token (also marks "lazy" matching) |
| `^` / `$` | start / end of the string (or line, with the `m` flag) |
| `[abc]` | any one of `a`, `b`, or `c` |
| `(...)` | a capture group |
| `{2,4}` | between 2 and 4 repetitions |

```javascript
const zipCode = /^\d{5}$/;          // exactly 5 digits, whole string
const simpleEmail = /^[\w.-]+@[\w.-]+\.\w+$/; // basic (not RFC-complete) email shape

console.log(zipCode.test("94107"));      // true
console.log(zipCode.test("9410"));       // false — only 4 digits
console.log(simpleEmail.test("ada@example.com")); // true
console.log(simpleEmail.test("not-an-email"));    // false
```

## `test` and `exec`

`.test()` returns a boolean. `.exec()` returns match details (or `null`),
and with the `g` flag can be called repeatedly to step through all matches.

```javascript
const pattern = /\d+/g;
const text = "Order 12 has 3 items, order 45 has 7 items";

let match;
while ((match = pattern.exec(text)) !== null) {
  console.log(match[0], "at index", match.index);
}
// 12 at index 6
// 3 at index 13
// 45 at index 30
// 7 at index 40
```

## String methods that accept a RegExp: `match`, `matchAll`, `replace`

Strings themselves expose regex-powered methods, often more convenient than
`exec` for one-off extraction or substitution.

```javascript
const sentence = "cats, dogs, and cats again";

console.log(sentence.match(/cats/));    // first match only (no 'g' flag): ['cats', index: 0, ...]
console.log(sentence.match(/cats/g));   // ['cats', 'cats'] — every match, as plain strings

console.log(sentence.replace("cats", "birds"));  // string arg replaces only the first occurrence
console.log(sentence.replace(/cats/g, "birds")); // regex + 'g' replaces every occurrence
// birds, dogs, and birds again

const allMatches = [...sentence.matchAll(/cats/g)]; // full match objects, spreadable
console.log(allMatches.length); // 2
```

## Capture groups

Parentheses create capture groups, letting you pull specific pieces out of
a match instead of the whole thing — named groups make the result even more
readable.

```javascript
const logLine = "2024-03-15 ERROR: disk full";
const logPattern = /^(\d{4})-(\d{2})-(\d{2}) (\w+): (.+)$/;

const [, year, month, day, level, message] = logLine.match(logPattern);
console.log(year, month, day, level, message);
// 2024 03 15 ERROR disk full
```

```javascript
// Named capture groups — clearer than positional destructuring
const namedPattern = /^(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const result = "2024-03-15 ERROR: disk full".match(namedPattern);

console.log(result.groups.year);  // 2024
console.log(result.groups.month); // 03
```

## Using capture groups in `replace`

Reference captured groups inside the replacement string with `$1`, `$2`,
etc. — useful for reformatting text without manually rebuilding it.

```javascript
const rawDate = "15/03/2024"; // DD/MM/YYYY
const isoDate = rawDate.replace(/(\d{2})\/(\d{2})\/(\d{4})/, "$3-$2-$1");
console.log(isoDate); // 2024-03-15
```

## `test`/`exec`/`match`/`replace` cheat sheet

| Method | Called on | Returns | Best for |
|--------|-----------|---------|----------|
| `regex.test(str)` | RegExp | boolean | quick yes/no validation |
| `regex.exec(str)` | RegExp | match array or `null`, stateful with `g` | stepping through matches one at a time |
| `str.match(regex)` | String | first match, or all matches (with `g`) | extracting matched text |
| `str.matchAll(regex)` | String | iterator of full match objects (requires `g`) | extracting matches with capture groups |
| `str.replace(regex, ...)` | String | new string | substituting or reformatting matched text |

## Exercise

Write a function `extractHashtags(text)` that returns an array of all
hashtags in a string (e.g. `"loving #javascript and #webdev"` →
`["#javascript", "#webdev"]`) using a regular expression with the global
flag.
