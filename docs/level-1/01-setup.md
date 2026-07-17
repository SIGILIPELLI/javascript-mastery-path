# 01 · Setup & First Program

## Install Node.js

Download the LTS release from [nodejs.org](https://nodejs.org/) or use your OS
package manager:

```bash
# macOS (Homebrew)
brew install node

# Ubuntu/Debian
sudo apt install nodejs npm

# Windows: use the installer from nodejs.org and check "Add to PATH"
```

Verify the install:

```bash
node --version
# v20.x.x
npm --version
# 10.x.x
```

## The Node REPL

The REPL (Read-Eval-Print Loop) is an interactive JavaScript shell — great for
quick experiments:

```bash
node
> 2 + 2
4
> console.log("hello")
hello
> .exit
```

## The browser console

Every browser ships a JavaScript console too. Open your browser's DevTools
(usually F12 or Cmd+Option+I) and switch to the "Console" tab — it's the same
JavaScript engine that runs the pages you visit, and a quick way to test DOM
code (covered in [Module 7](07-dom-events.md)).

```javascript
// Paste directly into the browser console:
console.log("hello from the browser");
document.title; // reads the current tab's title
```

## Your first script

Create `hello.js`:

```javascript
// hello.js
function greet(name) {
  return `Hello, ${name}!`;
}

console.log(greet("world"));
```

Run it with Node:

```bash
node hello.js
# Hello, world!
```

## Running JavaScript in the browser

Create `index.html` alongside `hello.js`:

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
  <head>
    <title>First Program</title>
  </head>
  <body>
    <script src="hello.js"></script>
  </body>
</html>
```

Open `index.html` in a browser and check the console — `greet` runs there too,
because the same JavaScript works in both environments (as long as it doesn't
use Node-only or browser-only APIs).

## Choosing an editor

Any of these work well for this program: VS Code (free, huge JavaScript/Node
extension ecosystem), WebStorm (free for non-commercial use), or even a plain
text editor plus the terminal. Pick one and move on — the editor matters far
less than practice.

## Exercise

Write a script `greet-many.js` that defines an array of three names and prints
a greeting for each one using the `greet` function above.
