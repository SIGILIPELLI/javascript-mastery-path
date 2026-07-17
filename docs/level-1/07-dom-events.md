# 07 · DOM Basics & Events

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

## Exercise

Build a page with a text input and a button. When the button is clicked,
append the input's current value as a new `<li>` to a `<ul>` on the page, then
clear the input. Use event delegation so clicking any list item toggles a
`done` CSS class on it.
