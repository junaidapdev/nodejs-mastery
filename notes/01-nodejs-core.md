# Section 1: Getting Started with Node.js

**Status:** ✅ Done | **Time:** 1hr 21min

---

## 🎯 Keywords (quick definitions)

- **Node.js** — JavaScript that runs on the server, not just in the browser
- **V8** — the engine (made by Google) that actually executes JS code
- **Runtime** — engine + the tools around it (file access, networking, etc.)
- **Module** — a file with code you can import into other files
- **`require()`** — the function that imports a module
- **`fs`** — built-in module for reading/writing files
- **package.json** — the project's ID card (name, dependencies, scripts)
- **package-lock.json** — locks exact versions so your team gets the same install
- **node_modules** — folder where installed packages live (never edit, never commit)
- **Sync (blocking)** — code waits for the line to finish before moving on
- **Async (non-blocking)** — code starts the work, moves on, comes back when ready
- **Event loop** — the thing that decides what runs next
- **Thread pool** — background workers that do heavy I/O while your main code keeps moving
- **libuv** — the C library that makes the event loop and thread pool work

---

## 🧠 Core Concepts

### 1. What is Node.js?

**Real-life analogy:** V8 is like a car engine. Powerful, but useless on its own — no wheels, no steering. **Node.js is the full car** — V8 engine + everything around it (wheels = file access, steering = networking) so it can actually go somewhere.

- **JavaScript** = the language (works anywhere)
- **V8** = Google's engine that runs JS (originally built for Chrome)
- **Node.js** = V8 + extra tools (built by Ryan Dahl in 2009) so JS can run on a server

**Java analogy:** Java has JVM + JRE → runs on any computer. JavaScript has V8 + Node → runs on any server. Same idea.

---

### 2. Why `alert()` works in browser but not in Node

**Real-life analogy:** Same chef (JavaScript), different kitchens.
- **Browser kitchen** has popup-makers, screens, mouse — so `alert()`, `document`, `window` work
- **Node kitchen** has files, network cables, server stuff — so `fs`, `http` work

The chef (JS) is the same. The tools available depend on which kitchen they're in.

| In browser, you get | In Node, you get |
|---|---|
| `window`, `document` | `fs`, `http` |
| `alert`, `localStorage` | `process`, `Buffer` |
| `fetch` (built-in) | `require()`, `__dirname` |

**Lesson:** code from a browser tutorial won't always work in Node, and vice versa.

---

### 3. JavaScript Modules — three types

**Real-life analogy:** Building a house.

| Type | Analogy | Example |
|---|---|---|
| **Built-in** | Tools that come in your toolbox | `fs`, `http`, `path` |
| **Third-party** | Tools you buy from the hardware store (npm) | `express`, `lodash` |
| **Custom** | Tools you make yourself in your workshop | `./utils.js`, `./config.js` |

```js
const fs = require('fs');               // built-in
const express = require('express');     // third-party (npm install)
const myUtils = require('./utils.js');  // custom (your file)
```

---

### 4. How `require()` actually works (the magic)

**Real-life analogy:** Every file in Node is like a sealed envelope. Outside the envelope, Node automatically gives you 5 tools: `require`, `module`, `exports`, `__filename`, `__dirname`. That's why you can use them anywhere without importing.

When you do `module.exports = something`, you're putting that "something" inside the envelope. When another file does `require('./that-file')`, Node opens the envelope and gives you what's inside.

**Why this matters:** variables in one file don't leak into other files. Each file is its own sealed scope. ⭐

---

### 5. `fs` — reading and writing files

```js
const fs = require('fs');

// SYNC (blocking) — names end with "Sync"
fs.writeFileSync('hello.txt', 'Hello');
const data = fs.readFileSync('hello.txt', 'utf-8');

// ASYNC (non-blocking) — uses callback
fs.readFile('hello.txt', 'utf-8', (err, data) => {
  if (err) return console.error(err);
  console.log(data);
});
```

Common operations: `readFile`, `writeFile`, `appendFile`, `mkdir`, `unlink` (delete).

Each has both a sync and async version. **Which to use? See next concept.**

---

### 6. Blocking vs Non-Blocking — the most important concept

**Real-life analogy:** A restaurant with one waiter (Node = single thread).

**Blocking (sync) waiter:**
- Takes Table 1's order
- Walks to kitchen, **stands there waiting** for the food
- Brings food to Table 1
- *Only now* takes Table 2's order
- Tables 2, 3, 4, 5... are starving 🍽️

**Non-blocking (async) waiter:**
- Takes Table 1's order, hands it to the kitchen, walks away
- Takes Table 2's order, hands it to the kitchen, walks away
- Takes Table 3's order...
- When kitchen rings the bell, picks up whichever food is ready

Same one waiter, but everyone gets served fast. **This is Node.**

```js
// 🚨 DISASTER — blocks every other user
app.get('/users', (req, res) => {
  const data = fs.readFileSync('huge.json', 'utf-8'); // server FROZEN
  res.send(data);
});

// ✅ GOOD — server keeps serving while file reads
app.get('/users', (req, res) => {
  fs.readFile('huge.json', 'utf-8', (err, data) => {
    res.send(data);
  });
});
```

**Rule:** sync APIs are OK only at startup (loading config). NEVER in code that handles user requests.

---

### 7. Event Loop & Thread Pool — the architecture

**Real-life analogy:** Coffee shop with one barista (your main thread) and helpers in the back (thread pool).

1. Customer orders a complex drink → barista hands the ticket to a back helper
2. Barista immediately serves the next customer (doesn't wait)
3. Back helper finishes the drink → puts it on the "ready" counter
4. **Event loop** = the barista glancing at the "ready" counter between customers, calling out names

```
Your code → Node hands I/O work to libuv
         → libuv uses thread pool (4 workers) to do the work
         → Main thread keeps running JS, NEVER blocked
         → Work finishes → callback queued in event loop
         → Event loop says: "main thread free? run this callback"
```

**Three things to remember:**
1. **Event loop** = the coordinator, decides what runs next
2. **Thread pool** = background workers (default: 4) for I/O work
3. **Your JS still runs single-threaded** — heavy CPU work (math, image processing) blocks Node, because there's no helper for *that*

---

### 8. `package.json` and friends

**Real-life analogy:** Cooking a recipe.

| File | What it is |
|---|---|
| **package.json** | The recipe card — name of dish, ingredients (dependencies), how to cook (scripts) |
| **package-lock.json** | The exact brand and quantity of every ingredient — so your friend's version tastes identical |
| **node_modules** | The pantry where all ingredients are stored — huge, auto-managed, never touch by hand |

```json
{
  "name": "my-app",
  "scripts": { "start": "node index.js" },
  "dependencies": { "express": "^4.18.2" },     // needed in production
  "devDependencies": { "nodemon": "^3.0.1" }    // only for development
}
```

**`^4.18.2`** = "any version 4.x.x ≥ 4.18.2" → can drift over time, that's why we need the lock file.

**Always commit `package-lock.json`. Never commit `node_modules`.**

---

## ⚡ "Aha" moments

- ⭐ **Node isn't a language. It's a runtime.** JavaScript is the language. Node is just the kitchen it runs in.
- ⭐ **Single-threaded ≠ slow.** By NOT making a thread per user (like old servers did), Node avoids the overhead. Async I/O does the magic.
- **`require()` isn't magic** — every file is wrapped in a hidden function that gives it `require`, `module`, etc. as parameters.

---

## 🔧 Patterns → for CLAUDE.md

| Rule | Why |
|---|---|
| ALWAYS use async `fs` APIs in request paths | Sync APIs freeze the server for everyone |
| PREFER built-in modules before installing npm packages | Less attack surface, smaller installs |
| ALWAYS commit `package-lock.json` | Without it, teammates get different installs |

---

## ⚠️ Anti-patterns → NEVER do

| Don't | Why it's bad |
|---|---|
| Use `fs.readFileSync` inside a route handler | Freezes the entire server during the read |
| Edit files inside `node_modules` | Changes vanish on next `npm install` |
| Commit `node_modules` to git | Repo bloat, slow clones, merge conflicts |
| `require(userInput)` with dynamic paths | Attacker can load any file on your system |

---

## 🔐 Security implications

- **Sync APIs = DoS vector.** One attacker triggering a slow sync read can freeze your whole server.
- **Every npm package runs with full Node privileges** — it can read your files, env vars, send data anywhere. `package-lock.json` prevents a malicious package update from sneaking in.
- **`fs` + user input = path traversal danger** (`../../etc/passwd`). Always validate file paths from users.

---

## ❓ Open questions

- How exactly does the event loop prioritize different callbacks (timers vs I/O vs `setImmediate` vs `process.nextTick`)? Worth learning when I hit a real bug.

---

## 🔗 Connects to

- **Section 2 (Events & Buffers)** → builds on the same async/callback foundation
- **Section 3 (HTTP module)** → every request flows through the event loop
- **Section 6 (Postgres + Drizzle)** → DB queries are I/O, same model

---

## ⭐ ONE thing to remember

> **Node = JavaScript engine (V8) + a kitchen full of server tools (file system, network). Its superpower is the single-thread + event loop + thread pool combo — one waiter serving many tables fast, BUT only if you use async APIs.**