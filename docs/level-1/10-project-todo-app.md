# 10 · Project — Browser To-Do List App

A small end-to-end project combining everything from Level 1: functions,
arrays/objects, strings, DOM manipulation, events, error handling, and
`localStorage` persistence.

## What you'll build

A browser-based to-do list that:

- Adds tasks from a text input
- Lists tasks (with done/pending status)
- Marks tasks done by clicking them
- Deletes tasks
- Persists everything to `localStorage` between page reloads

## Project layout

```text
todo-app/
    index.html
    style.css
    app.js
```

## index.html — page structure

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>To-Do List</title>
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <h1>To-Do List</h1>

    <form id="task-form">
      <input id="task-input" placeholder="What needs doing?" required />
      <button type="submit">Add</button>
    </form>

    <ul id="task-list"></ul>

    <script src="app.js"></script>
  </body>
</html>
```

## style.css — minimal styling

```css
li.done span {
  text-decoration: line-through;
  color: gray;
}

li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.4rem 0;
}
```

## app.js — storage layer

```javascript
// app.js — storage layer
const STORAGE_KEY = "todo-tasks";

function loadTasks() {
  const raw = localStorage.getItem(STORAGE_KEY);
  if (!raw) return [];
  try {
    return JSON.parse(raw);
  } catch (error) {
    console.error("Corrupted task data, resetting:", error);
    return [];
  }
}

function saveTasks(tasks) {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(tasks));
}
```

## app.js — application logic

```javascript
// app.js — application logic (continued)
let tasks = loadTasks();

const form = document.getElementById("task-form");
const input = document.getElementById("task-input");
const list = document.getElementById("task-list");

function render() {
  list.innerHTML = ""; // clear and redraw — simple approach for a small list

  if (tasks.length === 0) {
    const empty = document.createElement("li");
    empty.textContent = "No tasks yet.";
    list.appendChild(empty);
    return;
  }

  tasks.forEach((task, index) => {
    const item = document.createElement("li");
    item.className = task.done ? "done" : "";
    item.dataset.index = index;

    const label = document.createElement("span");
    label.textContent = task.description;

    const deleteButton = document.createElement("button");
    deleteButton.textContent = "Delete";
    deleteButton.className = "delete";

    item.appendChild(label);
    item.appendChild(deleteButton);
    list.appendChild(item);
  });
}

function addTask(description) {
  tasks.push({ description, done: false });
  saveTasks(tasks);
  render();
}

function toggleTask(index) {
  tasks[index].done = !tasks[index].done;
  saveTasks(tasks);
  render();
}

function deleteTask(index) {
  tasks.splice(index, 1);
  saveTasks(tasks);
  render();
}

form.addEventListener("submit", (event) => {
  event.preventDefault(); // stop the page from reloading on submit
  const description = input.value.trim();
  if (!description) return;
  addTask(description);
  input.value = "";
});

// Event delegation: one listener handles clicks on any current or future <li>
list.addEventListener("click", (event) => {
  const item = event.target.closest("li");
  if (!item || !item.dataset.index) return;
  const index = Number(item.dataset.index);

  if (event.target.classList.contains("delete")) {
    deleteTask(index);
  } else {
    toggleTask(index);
  }
});

render(); // initial paint from whatever was loaded from localStorage
```

## Running it

Open `index.html` directly in a browser (double-click it, or use a simple
local server like `npx serve`). Try:

- Typing a task and clicking "Add" (or pressing Enter)
- Clicking a task's text to toggle it done/pending
- Clicking "Delete" to remove a task
- Reloading the page — tasks persist because they're read from and written to
  `localStorage`

## Stretch goals

- Add due dates and sort tasks by them.
- Add a `priority` field (`low`/`medium`/`high`) and color-code list items with
  CSS classes.
- Add a "Clear completed" button that removes all `done` tasks at once.
- Move the storage/rendering logic into separate ES modules
  (`storage.js`, `render.js`) as practiced in
  [Module 9](09-modules-npm.md).

Completing this project means you're ready for **Level 2 · Intermediate**.
