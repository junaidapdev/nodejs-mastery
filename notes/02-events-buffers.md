# Section 2: Events and Buffers in Node.js

**Status:** ✅ Done | **Time:** 1hr 25min | **Lectures:** 5

---

## 🎯 Keywords (quick definitions)

- **Event** — a "something happened" signal in your app (user joined, file ready, error occurred)
- **Event-driven architecture** — your code waits silently and reacts only when events fire (vs constantly checking for things)
- **EventEmitter** — Node's built-in class that lets you create, fire, and listen to events
- **Emit** — to fire/trigger an event ("hey, this just happened!")
- **Listener** — a function that runs when a specific event is fired
- **Pub/Sub (Publisher–Subscriber) model** — one part publishes events, others subscribe to react
- **`.on()`** — listen to an event every time it fires
- **`.once()`** — listen to an event ONLY the first time it fires
- **`.emit()`** — fire an event (optionally with data)
- **`.removeListener()`** — stop listening to an event
- **Buffer** — a chunk of raw memory used to handle binary data (files, network packets, images)
- **Binary data** — raw 0s and 1s; the actual format computers store everything in
- **Encoding** — the rule for turning numbers into characters (e.g. UTF-8, UTF-16, hex)
- **UTF-8 / UTF-16** — character encoding standards; UTF-8 is most common
- **Hex (hexadecimal)** — base-16 number system used as a shorter way to display binary data
- **`Buffer.alloc(n)`** — create a buffer of n bytes, all zeroed out (safe)
- **`Buffer.allocUnsafe(n)`** — create a buffer of n bytes, NOT cleaned (faster, dangerous)
- **`Buffer.from(string)`** — create a buffer directly from a string

---

## 🧠 Core Concepts

### 1. What is an Event? (the Amazon delivery analogy)

**Real-life analogy:** You ordered something from Amazon. Two ways to know when it arrives:

| Approach | What you do |
|---|---|
| ❌ **Polling** | Keep running to the door every 30 seconds: "is it here yet? is it here yet?" Exhausting. |
| ✅ **Event-driven** | Live your life. When the doorbell rings, *then* you go check. |

**That doorbell = an event.** Your reaction = a listener.

This is how Node works. Your app doesn't constantly check things. It quietly waits, and when something happens (file ready, request arrives, user joins), the right function fires automatically.

**Why this matters:** events don't waste CPU. Code only runs when there's a reason to run.

---

### 2. The EventEmitter — Node's doorbell system

`EventEmitter` is the built-in class that gives you events. Every event-driven thing in Node (HTTP, streams, file system) is built on top of it.

**Real-life analogy:** EventEmitter is like a radio station.
- The station **broadcasts** (emits) on a frequency named "greet"
- Anyone with a radio tuned to "greet" (listeners) hears it
- One broadcast → many listeners can react

```js
const EventEmitter = require('events');     // import the class
const emitter = new EventEmitter();         // create your radio station

// Tune in (add a listener)
emitter.on('greet', (name) => {
  console.log(`Hello ${name}!`);
});

// Broadcast (emit the event)
emitter.emit('greet', 'Hitesh');   // → "Hello Hitesh!"
```

**The flow:**
1. Create an emitter
2. Use `.on('eventName', callback)` to listen
3. Use `.emit('eventName', data)` to fire

**Naming rule:** event names are just strings. You can name them anything — `greet`, `userJoined`, `superman`, `payment-completed`. In real apps, store these names in a `constants.js` file to avoid typos.

---

### 3. `.on()` vs `.once()` — listening forever vs one-time

| Method | Behavior | Real-life example |
|---|---|---|
| **`.on('event', fn)`** | Runs every time the event fires | Doorbell — rings every time someone comes |
| **`.once('event', fn)`** | Runs ONLY the first time, then auto-removes | "Welcome" message when a user joins a chat — only shown once |

```js
emitter.once('userJoined', (user) => {
  console.log(`Welcome ${user}, this only fires once!`);
});

emitter.emit('userJoined', 'Alice');  // ✅ fires
emitter.emit('userJoined', 'Bob');    // ❌ ignored (already fired once)
```

**Use `.once()` for:** welcome messages, "first time only" tracking, one-shot setup events.

---

### 4. Removing listeners — `.removeListener()`

You can stop listening at any time. Useful when a user logs out, a chat closes, etc.

```js
const myFn = () => console.log('I am listening');

emitter.on('test', myFn);
emitter.emit('test');  // fires myFn

emitter.removeListener('test', myFn);
emitter.emit('test');  // nothing happens
```

**Important:** to remove a listener, you must pass the **same function reference** you used in `.on()`. That's why you store it in a variable first.

---

### 5. Multiple listeners on the same event

One event can have many listeners. They all fire when the event happens.

**Real-life analogy:** A fire alarm goes off — the fire department, the building manager, and the sprinklers all react.

```js
emitter.on('greet', (name) => console.log(`Hello ${name}`));
emitter.on('greet', (name) => console.log(`Hola ${name}`));

emitter.emit('greet', 'Hitesh');
// → "Hello Hitesh"
// → "Hola Hitesh"
```

Useful methods:
- `emitter.listenerCount('event')` — how many listeners are on this event?
- `emitter.listeners('event')` — get the array of listener functions

---

### 6. The class-based pattern (the cleaner way)

Instead of creating an emitter directly, you usually **extend `EventEmitter`** in your own class. This is how production code looks.

**Real-life analogy:** Building your own custom radio station. You inherit all the broadcasting features, then add your own buttons and channels on top.

```js
class ChatRoom extends EventEmitter {
  sendMessage(msg) {
    console.log(`Sent: ${msg}`);
    this.emit('messageReceived', msg);   // emit from inside the class
  }
}

const chat = new ChatRoom();

chat.on('messageReceived', (msg) => {
  console.log(`New message: ${msg}`);
});

chat.sendMessage('Hello!');
```

**Why this is better:**
- Methods (`sendMessage`) live with the data they manage
- The class itself is both the emitter AND has business logic
- Reusable — create multiple chat rooms with `new ChatRoom()`

---

### 7. Error handling with events ⚠️ important

If an event named `'error'` fires and **no listener is attached, Node CRASHES the entire app**. This is intentional — Node forces you to handle errors.

```js
emitter.on('error', (err) => {
  console.error('Error occurred:', err.message);
});

emitter.emit('error', new Error('Something broke!'));
// ✅ caught, app keeps running

// If you forget the listener:
emitter.emit('error', new Error('Something broke!'));
// 🚨 Node crashes
```

**Rule:** any time you build something with EventEmitter, add an `'error'` listener. Always.

---

### 8. Why use events at all?

| Benefit | Why it matters |
|---|---|
| Doesn't choke CPU | Code runs only when needed (vs constantly checking) |
| Async without callback hell | Cleaner than nested callbacks for "wait for X then do Y" |
| Real-time apps | Chat, notifications, live dashboards all use events |
| Foundation of Node modules | `fs`, `http`, `streams` — all built on EventEmitter |

**Once you understand events, every other Node module makes more sense.** ⭐

---

### 9. What is a Buffer? (and why it exists)

**Real-life analogy:** A buffer is a **temporary parking lot for raw data**.

You can't drive a car directly into a building — you park it first, then move stuff inside. Same with files and network data: Node doesn't read directly from disk to your variable. It loads chunks into a buffer (parking lot), then your code reads from it.

**Why we need this:**
- JavaScript was built for the browser, where you mostly handle text
- But servers deal with files, images, videos, network packets — raw binary
- JS strings can't represent raw bytes efficiently
- **Buffer = a special box for handling raw binary data**

**Where buffers show up:**
- Reading files (`fs` module)
- Network requests (HTTP, TCP)
- Streams (uploads, video, large data)
- Image/PDF/file processing

---

### 10. How data is stored — bits, bytes, and encoding

**Real-life analogy:** Imagine an alphabet system where every letter is a number. The number `72` means `H`. The number `128512` means 😀.

This is exactly how computers work — every character/emoji has a unique number, and that number is stored as binary (0s and 1s).

```
"H" → number 72 → binary 01001000
```

**Three ways to view the same data:**

| Format | Example | Use case |
|---|---|---|
| **Binary** | `01001000` | What's actually in memory |
| **Hex** | `0x48` | Shorter way to display binary (most common) |
| **Decimal** | `72` | Human-readable number |
| **UTF-8 char** | `H` | What we actually want to read |

**Encoding** is the rule that says: "this number = this character." UTF-8 is the most popular standard. Same number in a different encoding could mean a different character.

---

### 11. Creating buffers — three ways

```js
const { Buffer } = require('node:buffer');

// 1. Allocate empty space (SAFE — initialized to zeros)
const buf1 = Buffer.alloc(4);
console.log(buf1);  // <Buffer 00 00 00 00>

// 2. Create from a string (most common)
const buf2 = Buffer.from('Hello');
console.log(buf2);              // <Buffer 48 65 6c 6c 6f>
console.log(buf2.toString());   // "Hello"

// 3. Allocate UNSAFE (fast but dangerous — uninitialized memory)
const buf3 = Buffer.allocUnsafe(10);
console.log(buf3);  // could contain old garbage data!
```

**Real-life analogy for `alloc` vs `allocUnsafe`:**
- `alloc` = renting a hotel room that's been cleaned ✅
- `allocUnsafe` = renting a hotel room nobody cleaned — last guest's stuff might still be there ⚠️

**Rule:** use `alloc` by default. Only use `allocUnsafe` when you're 100% sure you're going to overwrite every byte immediately, and speed matters.

---

### 12. Reading and writing buffers

```js
// Write to a buffer
const buf = Buffer.alloc(10);
buf.write('Hello');
console.log(buf.toString());  // "Hello"

// Read with encoding & range (start, end)
const buf2 = Buffer.from('chai or code');
console.log(buf2.toString('utf-8', 0, 4));  // "chai"

// Modify a single byte
const buf3 = Buffer.from('chai');
console.log(buf3);              // <Buffer 63 68 61 69>
buf3[0] = 0x4a;                 // change first byte to "J"
console.log(buf3.toString());   // "Jhai"

// Concatenate buffers
const a = Buffer.from('chai ');
const b = Buffer.from('code');
const merged = Buffer.concat([a, b]);
console.log(merged.toString());  // "chai code"

// Get length in bytes
console.log(merged.length);  // 9
```

**Key thing to remember:** when you `console.log(buffer)`, you see hex values. To see actual text, use `.toString()`. Many devs forget this and get confused. ⭐

---

## ⚡ "Aha" moments

- ⭐ **Events = doorbell, not polling.** This single mental shift makes async Node make sense.
- ⭐ **Every Node module is event-driven under the hood.** `fs`, `http`, streams — all extend EventEmitter. Master events, master Node.
- ⭐ **Buffer = parking lot for raw bytes.** Files don't come as strings, they come as binary chunks. Buffer is where they sit before you process them.
- **Forgetting an `error` listener = crash.** Node does this on purpose to force you to handle failures.
- **`allocUnsafe` literally gives you whatever was in that RAM before.** Could be empty, could be the last app's leftover data. Wild.

---

## 🔧 Patterns → for CLAUDE.md

| Rule | Why |
|---|---|
| ALWAYS attach an `'error'` listener to every EventEmitter | Unhandled error events crash the entire Node process |
| ALWAYS store event names in a `constants.js` file | Prevents typo bugs (`'userJoind'` vs `'userJoined'`) |
| PREFER extending `EventEmitter` in a class over standalone emitters | Keeps related methods + events together; cleaner architecture |
| ALWAYS use `Buffer.alloc()` over `Buffer.allocUnsafe()` unless performance is proven critical | Unsafe version may leak old memory data |
| ALWAYS call `.toString()` when reading a buffer as text | Otherwise you get hex output |
| PASS function references (not inline arrows) to `.on()` if you might need to remove them later | `.removeListener()` requires the same reference |

---

## ⚠️ Anti-patterns → NEVER do

| Don't | Why it's bad |
|---|---|
| Emit an `'error'` event without a listener | Crashes the whole app |
| Use `Buffer.allocUnsafe()` without immediately overwriting all bytes | May contain sensitive leftover data from RAM |
| Hardcode event name strings everywhere in the codebase | Typos = silent bugs (event never fires, no error) |
| Forget `.toString()` when reading buffer text data | You'll see hex, get confused, waste time debugging |
| Attach the same listener inside a loop | Memory leak — listeners pile up forever |

---

## 🔐 Security implications

- **`Buffer.allocUnsafe()` can leak data.** If you don't fully overwrite the buffer before sending it somewhere (response, file, log), you could expose memory contents from previous requests — including passwords, tokens, other users' data. **This is a real CVE class.**
- **Unhandled `error` events crash the server** → DoS vector if attacker can trigger an error-emitting code path you forgot to listen to.
- **Buffers and user input:** if you take user input and put it into a buffer, watch the size. An attacker sending a 10GB payload can OOM (out-of-memory) crash your server. Always set size limits.
- **Memory leaks from forgotten listeners:** every `.on()` without a matching `.removeListener()` (when no longer needed) holds memory. In long-running servers this builds up.

---

## ❓ Open questions

- When should I prefer `EventEmitter` vs Promises vs async/await? (Probably: events for "happens N times," promises for "happens once.")
- How does `EventEmitter` work under the hood — is it just an array of callbacks?

---

## 🔗 Connects to

- **Section 1 (Node core)** → events ARE the async architecture. Now I see what async actually looks like.
- **Section 3 (HTTP module)** → `req` and `res` are EventEmitters. `req.on('data', ...)` is exactly this pattern.
- **Section 6 (Database)** → DB connections emit events (`connect`, `error`, `disconnect`).
- **Streams (later)** → entirely built on events. `.on('data')`, `.on('end')`, `.on('error')`.
- **File system** → `fs` reads/writes are buffer-based under the hood.

---

## ⭐ ONE thing to remember

> **Events = doorbell pattern (don't poll, just listen). EventEmitter is the doorbell system every Node module is built on. Always handle the `error` event or your app crashes. Buffers are parking lots for raw binary data — needed because JS strings can't handle files, images, or network packets natively.**

---

## ✏️ Mini-project recap: Chat Room simulation

Built a `ChatRoom` class that extends `EventEmitter`:
- `join(user)` → emits `'join'` event
- `sendMessage(user, msg)` → emits `'message'` event (only if user is in active set)
- `leave(user)` → emits `'leave'` event
- Validates user is in active set before sending — silent rejection otherwise

Driver code listens to all three events and prints messages.

**Why this matters for real apps:** this is the exact pattern behind Socket.io, real-time chat, notifications, multiplayer games. The transport changes (WebSocket vs in-memory), but the event pattern is identical.

**Code lives in:** `code-along/week-1-chat-room/` (when you build it)