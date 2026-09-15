---
description: "DOM Basics & Events — Attaching one listener to a parent, then checking event.target, scales better than attaching a listener to every child individually…"
---

# 07 · DOM Basics & Events

## 🎥 Video walkthrough

<iframe width="100%" height="400" style="max-width:720px;aspect-ratio:16/9;height:auto;" src="https://www.youtube.com/embed/pMXonwnLuGI" title="Video walkthrough" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

This module only applies in the browser — Node.js has no DOM. Try these
snippets by pasting them into the browser console on any page, or wiring them
up in an `index.html` + `.js` file pair.

## Selecting elements

```javascript
// Given: <div id="app"><p class="note">Hello</p></div>

const app = document.getElementById("app");
const note = document.querySelector(".note");     // first match
const allNotes = document.querySelectorAll(".note"); // NodeList of all matches

console.log(note.textContent); // Hello
```

## Creating and modifying elements

```javascript
const heading = document.createElement("h1");
heading.textContent = "Welcome";
heading.classList.add("title"); // add a CSS class

document.body.appendChild(heading); // insert into the page

note.textContent = "Updated text";
note.style.color = "blue";
note.setAttribute("data-status", "read");
console.log(note.getAttribute("data-status")); // read
```

## Removing elements

```javascript
const toRemove = document.querySelector(".note");
toRemove.remove(); // modern, direct removal

// Older pattern, still common in codebases:
// toRemove.parentNode.removeChild(toRemove);
```

## Handling events

```javascript
const button = document.createElement("button");
button.textContent = "Click me";
document.body.appendChild(button);

button.addEventListener("click", () => {
  console.log("Button was clicked!");
});

// The event object carries details about what happened
button.addEventListener("click", (event) => {
  console.log("Target:", event.target.textContent);
});
```

## Common events

| Event | Fires when |
|-------|------------|
| `click` | an element is clicked |
| `input` | a form field's value changes as the user types |
| `submit` | a form is submitted |
| `keydown` / `keyup` | a key is pressed / released |
| `DOMContentLoaded` | the HTML has finished loading (before images/styles) |
| `load` | the whole page (including assets) has finished loading |

## A small interactive example

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
  <body>
    <input id="name-input" placeholder="Type your name" />
    <p id="greeting"></p>
    <script src="app.js"></script>
  </body>
</html>
```

```javascript
// app.js
const input = document.getElementById("name-input");
const greeting = document.getElementById("greeting");

input.addEventListener("input", (event) => {
  const value = event.target.value;
  greeting.textContent = value ? `Hello, ${value}!` : "";
});
```

## Event delegation

Attaching one listener to a parent, then checking `event.target`, scales
better than attaching a listener to every child individually — especially
when children are added/removed dynamically (as in the to-do app in
[Module 10](10-project-todo-app.md)).

```javascript
const list = document.getElementById("list");

list.addEventListener("click", (event) => {
  if (event.target.matches("button.delete")) {
    event.target.closest("li").remove();
  }
});
```

## How It Actually Works

The DOM and JavaScript are two separate worlds connected by bindings: the DOM tree
itself lives in the browser's C++ rendering engine, and every `element.addEventListener`
call registers your callback in the engine's event-dispatch tables, not in V8's memory
directly. When a click happens, the browser doesn't run your handler immediately as
part of the click — it constructs an `Event` object and schedules a **task** on the
main thread's task queue. The event loop picks up that task, and only then does it call
into V8 to run your handler synchronously to completion before picking up the next
task. This is why a slow event handler blocks all rendering and other events: there is
only one thread, and tasks run one at a time to completion.

Event dispatch itself follows three phases you can hook into: **capture** (root down to
target, via the `{capture: true}` option), **target**, and **bubble** (target back up
to root, the default). `event.stopPropagation()` halts this walk but doesn't cancel the
browser's own default behavior (scrolling, following a link) — for that you need
`event.preventDefault()`, which just sets a flag the browser checks after your handler
returns. Because listeners are stored as references, closures you create inside
`addEventListener` keep their entire enclosing scope alive for as long as the listener
is attached — a common, hard-to-spot memory leak source when elements are removed from
the DOM without calling `removeEventListener` first.
## 🔀 See this in another language

- [TypeScript — Classes Basics](https://sigilipelli.github.io/typescript-mastery-path/level-1/07-classes-basics/)
- [Ruby — Classes & Objects Basics](https://sigilipelli.github.io/ruby-mastery-path/level-1/07-classes-objects/)
- [R — Reading Data](https://sigilipelli.github.io/r-mastery-path/level-1/07-reading-data/)

## Exercise

Build a page with a text input and a button. When the button is clicked,
append the input's current value as a new `<li>` to a `<ul>` on the page, then
clear the input. Use event delegation so clicking any list item toggles a
`done` CSS class on it.
