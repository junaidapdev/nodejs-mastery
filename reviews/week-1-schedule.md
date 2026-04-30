# Week 1 Schedule — Foundations

> **Goal:** Master Node.js core (modules, file system, event loop), event-driven patterns (EventEmitter, Buffers), and the native HTTP module. By the end of this week, I should be able to build a working HTTP server WITHOUT Express.
>
> **Total course time this week:** ~3 hr 52 min of video. With my 2hr/day plan including notes + type-along, this is comfortably 5 days.

---

## Daily structure (every weekday)

| Block | Time | What |
|---|---|---|
| Watch | ~60 min | Course at 1.25–1.5x |
| Type-along | ~25 min | Re-do compounding parts |
| Notes | ~20 min | Fill 3-layer template |
| Harvest | ~5 min | Update CLAUDE.md if anything new |

---

## Day 1 (Mon) — Node.js: What, Why, How

**Watch (Section 1, lectures 1–4):**
- What is Node.js and Why Use It for Server-Side Development (11:23)
- Installing Node.js and Setting Up a Local Development Environment (4:35)
- Your First Node.js Script: Writing and Running Hello World (7:17)
- Node.js vs Browser JavaScript Engines: Key Differences (5:14)

**Type-along:**
- Skip the install lecture (just do it)
- Type the first script yourself, then experiment: read a file, print to console, run with different flags

**Notes target:** open `notes/01-nodejs-core.md`, fill the **Core Concepts** section for "What is Node.js" and "Node.js vs browser." This is the foundation — invest in writing it clearly in your own words.

**Vocabulary to capture:** V8, runtime, REPL, libuv, blocking, non-blocking

**Commit:** `git commit -m "Day 1: Node.js intro and environment"`

---

## Day 2 (Tue) — Modules, npm, Package Management

**Watch (Section 1, lectures 5–6):**
- Understanding Node.js Modules and Their Role in Structuring Code (16:22)
- Working with Third-Party Modules using npm and Package.json (10:47)

**Type-along:**
- Create a small project with 2–3 modules of your own (a `math.js` you import, a `greet.js`, etc.)
- Initialize a real `package.json`, install lodash or dayjs, use it
- Try both CommonJS (`require`) and ES Modules (`import`) — note the differences

**Notes target:** fill **Core Concepts** for modules and npm. Capture the `require` vs `import` distinction. Add an entry to `_index.md` for: module, npm, package.json, dependency, devDependency, peerDependency.

**Compounding code to type:** the module patterns. You'll use these every day forever.

**Commit:** `git commit -m "Day 2: Modules and npm"`

---

## Day 3 (Wed) — File System & Sync vs Async

**Watch (Section 1, lectures 7–8):**
- Using the File System (FS) Module to Read and Write Files (7:38)
- Understanding Blocking vs Non-Blocking Code Execution in Node.js (8:57)

**Type-along (definitely type this — it compounds):**
- Read a file synchronously, then asynchronously, then with promises, then with async/await
- Write a small script that reads multiple files in parallel vs sequentially — feel the difference
- Cause an actual blocking situation (a `for` loop with millions of iterations) and watch your server stall

**Notes target:** fill **Core Concepts** for sync vs async, blocking vs non-blocking. This is the conceptual heart of Node — if you don't get this, you don't get Node. Spend extra time here.

**🔧 Pattern to capture:** "Always prefer async I/O. Sync APIs only acceptable for one-time startup operations (e.g. reading config)."

**Commit:** `git commit -m "Day 3: File system and async fundamentals"`

---

## Day 4 (Thu) — The Event Loop & Internal Architecture

**Watch (Section 1, lecture 9):**
- The Internal Architecture of Node.js: Event Loop and Thread Pool (8:32)

**Plus Section 2, lectures 1–2:**
- What Are Events in Node.js and Why Are They Important? (11:03)
- Creating and Using Event Emitters in Node.js (23:29)

**Type-along (compounding — type this!):**
- Write your own EventEmitter usage — emit custom events, listen with `.on()`, handle errors
- Build a tiny pub/sub: one module emits, two others listen

**Notes target:** finish `01-nodejs-core.md`, start `02-events-buffers.md`. The event loop diagram should be in your notes (draw it ASCII or just describe it in your words). Add to `_index.md`: event loop, call stack, callback queue, microtask queue, EventEmitter.

**🔧 Pattern to capture:** "Use EventEmitter to decouple modules. Emitter doesn't know who's listening."

**Commit:** `git commit -m "Day 4: Event loop and EventEmitter basics"`

---

## Day 5 (Fri) — Real-Time Patterns & Buffers

**Watch (Section 2, lectures 3–5):**
- Building a Real-Time Chat App with Custom Events in Node.js (16:26)
- Introduction to Buffers in Node.js: What and Why (17:33)
- Working with Buffers in Node.js Using Practical Code Examples (15:59)

**Type-along:**
- Build the chat app pattern yourself. This is your first taste of real-time architecture.
- Create and manipulate Buffers: convert strings to buffers and back, slice them, concat them.

**Notes target:** finish `02-events-buffers.md`. Capture WHY Buffers exist (JS strings can't represent arbitrary binary data efficiently).

**Friday recall test (15 min):** close all notes. Write down everything you remember from this week into a blank file `reviews/week-1-recall.md`. Then compare to your notes. Whatever you missed is exactly what to re-study tomorrow.

**Commit:** `git commit -m "Day 5: Custom events pattern and Buffers"`

---

## Saturday — Build day (no new lectures)

**Challenge:** Build a basic HTTP server using ONLY Node's `http` module (no Express). Requirements:
- Routes: `GET /`, `GET /users`, `POST /users`, `GET /users/:id`
- In-memory user store
- JSON request/response
- Proper status codes
- Catches its own errors

**Don't watch lectures during this.** This is recall practice. If you get stuck, re-watch only the specific bit you need.

When done, save it to `code-along/week-1-http-server/`.

**Commit:** `git commit -m "Day 6: Self-built HTTP server, no framework"`

---

## Sunday — Review day (30 min total)

1. **Re-read** `notes/01-nodejs-core.md` and `notes/02-events-buffers.md`
2. **Update** `notes/_index.md` glossary with new terms from the week
3. **Harvest** patterns/anti-patterns into `standards/CLAUDE.md`. Examples that probably belong there now:
   - "ALWAYS prefer async I/O over sync APIs except for one-time startup reads"
   - "ALWAYS handle the `error` event on EventEmitters — unhandled error events crash the process"
   - "NEVER block the event loop with synchronous CPU-heavy work"
4. **Write** `reviews/week-1.md` retrospective (template provided in that folder)
5. **Push** everything to GitHub
6. **Plan** Week 2 — preview Section 3 (HTTP module) and Section 4 (Express). I'll have your Week 2 schedule ready when you ask.

**Commit:** `git commit -m "Week 1 complete: foundations locked in"`

---

## Week 1 success criteria

By Sunday night, I should be able to:
- [ ] Explain what Node.js is to a non-technical client in 2 sentences
- [ ] Write a Node.js module and import it from another file (both CommonJS and ESM)
- [ ] Read/write files asynchronously using fs.promises
- [ ] Explain why `setTimeout(fn, 0)` doesn't run immediately
- [ ] Build a basic HTTP server without Express
- [ ] Use EventEmitter to decouple two parts of an app
- [ ] Manipulate a Buffer

If any of these feels shaky, that's a Week 2 morning warm-up topic.

---

## Notes for the future me

- Events and Buffers will feel abstract this week. That's fine. They click in Week 3 when you see streams and DB drivers using them under the hood.
- Don't get lost in the weeds of "is `require` better than `import`." Use ESM for new projects, accept CommonJS in legacy code, move on.
- The event loop topic (Day 4) is the single most important concept in Node. If anything is unclear, search Philip Roberts' "What the heck is the event loop" talk on YouTube — 25 min, worth every second.
