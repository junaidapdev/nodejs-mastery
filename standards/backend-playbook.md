# Backend Engineering Playbook

> **Purpose:** Human-readable reference I read when reviewing what Claude Code (or anyone else) wrote.
> Every entry explains the **what**, the **why**, and the **trade-offs**. This is the "thinking" version of `CLAUDE.md`.

---

## Table of contents

- [Project Architecture](#project-architecture)
- [Express Application Setup](#express-application-setup)
- [Error Handling Philosophy](#error-handling-philosophy)
- [Authentication Strategies](#authentication-strategies)
- [Database Design](#database-design)
- [API Design Principles](#api-design-principles)
- [Security Mindset](#security-mindset)
- [Performance & Scaling](#performance--scaling)
- [Deployment & Operations](#deployment--operations)

---

## Project Architecture

### Why MVC (and what it really means in Node)

In Node/Express, "MVC" is more accurately **Routes → Controllers → Services → Models**:

- **Routes** map URLs to controllers. Just routing, no logic.
- **Controllers** handle the HTTP layer: parse `req`, call services, send `res`. They should be thin.
- **Services** contain business logic. They don't know about HTTP. You can call them from anywhere — a route, a cron job, a CLI script.
- **Models / Schemas** define the data shape and how it persists.

**Why this separation matters:** when a client says "we need to also send this data to a Slack webhook when a user signs up," you don't want to find that logic buried in a route handler. It belongs in a service that the route, a future cron, and a future admin tool can all call.

### The `app.js` vs `index.js` split

- `app.js` builds the Express application: middleware, routes, error handler. Exports the app.
- `index.js` (or `server.js`) imports `app`, connects to the DB, starts the server.

**Why:** testability. You can import `app` into a test file and run requests against it without ever starting a real server or hitting a real DB.

---

## Express Application Setup

### Middleware order matters

Middleware runs in the order you register it. The standard order is:

1. Security middleware (`helmet`, `cors`)
2. Body parsers (`express.json()`, `express.urlencoded()`)
3. Logging middleware
4. Rate limiters (route-specific)
5. Routes (`/api/v1/...`)
6. 404 handler
7. **Centralized error handler (LAST)**

The error handler must be last because Express identifies error middleware by its 4-argument signature `(err, req, res, next)`, and it only catches errors from middleware/routes registered *before* it.

### Why use `asyncHandler`

By default, if you `throw` inside an async route handler, Express doesn't catch it — your app crashes or the request hangs. `asyncHandler` is a 5-line utility that wraps async functions and forwards errors to `next()`:

```js
const asyncHandler = (fn) => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);
```

Now every error — thrown, rejected, anything — flows to your central error handler. Without this, every controller needs its own try/catch.

---

## Error Handling Philosophy

### Two classes that solve 90% of error problems

**`ApiError`** — a custom error class for known, expected failures:
```js
class ApiError extends Error {
  constructor(statusCode, message, errors = []) {
    super(message);
    this.statusCode = statusCode;
    this.errors = errors;
    Error.captureStackTrace(this, this.constructor);
  }
}
```

**`ApiResponse`** — a consistent success response shape:
```js
class ApiResponse {
  constructor(statusCode, data, message = 'Success') {
    this.statusCode = statusCode;
    this.data = data;
    this.message = message;
    this.success = statusCode < 400;
  }
}
```

**Why this matters:** every endpoint returns the same shape. Frontend devs (and you, debugging at 2am) always know what `response.data` looks like. Errors and successes are distinguishable by the `success` flag.

### Process-level handlers

```js
process.on('unhandledRejection', (reason) => { /* log, exit */ });
process.on('uncaughtException', (err) => { /* log, exit */ });
```

These catch errors that escape your normal flow. Best practice: log and exit (let your process manager — pm2, Docker — restart). Trying to "recover" from an uncaught exception leaves the process in an unknown state.

---

## Authentication Strategies

### Sessions vs JWT — the real trade-off

| | Sessions | JWT |
|---|---|---|
| State | Server stores session in DB/Redis | Stateless, server stores nothing |
| Revocation | Easy: delete session record | Hard: token is valid until expiry |
| Scaling | Need shared session store across servers | Trivial — just verify signature |
| Size | Small cookie, just session ID | Larger token with payload |
| Best for | Traditional web apps, easy logout | APIs, microservices, mobile clients |

**The pragmatic answer:** for most production apps with a frontend, use **httpOnly cookies containing a JWT**. You get JWT's stateless verification and the cookie's XSS protection. For instant revocation, keep a small "revoked tokens" set in Redis (only checked for sensitive operations).

### Why bcrypt and not SHA-256

SHA-256 is fast — billions of hashes per second on a GPU. That's a *bad* property for password hashing. bcrypt is intentionally slow (configurable cost factor) and includes a salt. Even if your DB leaks, attackers can't easily brute-force passwords.

Cost factor: 10 is the modern minimum, 12 is safer. Each increment doubles the time.

### RBAC done right

Don't put role checks in controllers:

```js
// BAD
async function deleteUser(req, res) {
  if (req.user.role !== 'admin') return res.status(403).send();
  // ...
}
```

Put them in middleware:

```js
// GOOD
router.delete('/users/:id', requireAuth, requireRole('admin'), deleteUser);
```

Why: the route definition becomes self-documenting. Anyone reading the routes file knows immediately what permissions each route requires.

---

## Database Design

### Indexes — the most underrated 5 minutes you'll spend

A query without an index does a full table scan. With an index, it's a B-tree lookup. The difference at 1M rows is ~1000x.

**Always index:**
- Foreign keys
- Columns in `WHERE` clauses you run often
- Columns in `ORDER BY`
- Composite indexes for multi-column queries (order matters: most selective first)

**Don't index:**
- Columns that change very frequently (write penalty)
- Tables with very few rows
- Boolean columns (low selectivity)

### Migrations are non-negotiable

Never edit a production database manually. Never push schema changes by re-running an init script. Use migrations — versioned, reviewable, rollback-able files that describe schema changes.

Drizzle Kit and Mongoose both support this. Use them from day one, not after your first incident.

### When to denormalize (NoSQL specifically)

In Mongo, joining is expensive. If you read user + their last 5 orders together 1000x/day, consider embedding a cached `lastOrders` array on the user doc. Trade-off: writes get more complex (you update in two places). Rule: optimize for reads if reads >> writes.

---

## API Design Principles

### REST is a vocabulary, not a religion

The point of REST: predictable URLs and verbs. Once a frontend dev has hit one of your endpoints, they should be able to guess the rest.

- `GET /api/v1/books` — list
- `GET /api/v1/books/:id` — read one
- `POST /api/v1/books` — create
- `PATCH /api/v1/books/:id` — partial update
- `PUT /api/v1/books/:id` — full replace (rarely used)
- `DELETE /api/v1/books/:id` — remove

For things that don't fit (e.g. "send a password reset email"), it's fine to use action-style routes: `POST /api/v1/auth/reset-password`. Don't contort REST to fit every operation.

### Status codes that matter

- **200** OK — success with body
- **201** Created — POST that created a resource
- **204** No Content — DELETE that succeeded
- **400** Bad Request — malformed input
- **401** Unauthorized — not authenticated
- **403** Forbidden — authenticated but not allowed
- **404** Not Found — resource doesn't exist
- **409** Conflict — duplicate, version conflict
- **422** Unprocessable Entity — validation failed (use this OR 400, pick one and stick with it)
- **500** Internal Server Error — bug on your end

### Pagination — always, from day one

A list endpoint without pagination is a time bomb. The day a user has 10,000 orders, your app dies. Two patterns:

- **Offset pagination:** `?page=2&limit=20` — easy to implement, breaks when data shifts
- **Cursor pagination:** `?after=<id>&limit=20` — stable, scales infinitely, harder to skip to "page 47"

For most CRUD apps, offset is fine. For feeds and infinite scroll, use cursors.

---

## Security Mindset

### The OWASP Top 10 in plain English

1. **Broken Access Control** — most common. User A can access User B's data. Fix: check ownership in every route that touches user data.
2. **Cryptographic Failures** — passwords in plaintext, weak hashing, secrets in code.
3. **Injection** — SQL, NoSQL, command injection. Fix: parameterized queries (ORMs do this), validate all input.
4. **Insecure Design** — architecture flaws (e.g. password reset that exposes user existence).
5. **Security Misconfiguration** — default passwords, debug mode in prod, exposed admin panels.
6. **Vulnerable Components** — outdated dependencies. Run `npm audit`.
7. **Authentication Failures** — weak passwords allowed, no rate limit on login, predictable tokens.
8. **Data Integrity Failures** — trusting unsigned data, deserializing untrusted input.
9. **Logging Failures** — can't tell what happened during an incident.
10. **SSRF** — your server fetches a URL the user controls, which targets internal infra.

### The "what if" mental model for code review

For every endpoint, ask:
- What if the user is not who they claim?
- What if they pass garbage input?
- What if they pass valid-looking input for someone else's resource?
- What if they call this endpoint 10,000 times in a second?
- What if the DB call fails halfway?
- What if two users do this simultaneously?

If your code doesn't have an answer to each, it has a bug.

---

## Performance & Scaling

### The event loop in one paragraph

Node.js runs your JavaScript on a single thread. While that thread is busy doing CPU work, no other request can be served. I/O (DB, file, network) is delegated to background threads/OS, so your handler can return to the loop and serve more requests. The implication: if a handler does heavy CPU work (image processing, complex math), it blocks every other user. Fix: offload to worker threads, child processes, or a separate service.

### When to scale vertically vs horizontally

- **Vertical** (bigger server) — easy, has a ceiling, single point of failure
- **Horizontal** (more servers behind a load balancer) — harder, but unlimited

Most apps should start vertical. Move horizontal when (a) you need redundancy, or (b) you've hit the largest reasonable single machine. Don't over-engineer for scale you don't have.

### Caching tiers

1. **In-memory** (per-process Map) — fastest, doesn't survive restart, doesn't share across processes
2. **Redis** — shared across processes, very fast, survives restart
3. **CDN** — for static assets, edge-cached, fastest for global users

For API responses, Redis is usually the right answer. Cache invalidation is the hard part — prefer short TTLs over manual invalidation when you can.

---

## Deployment & Operations

### The 12-Factor checklist

The 12-Factor App methodology is the closest thing the industry has to a deployment standard. The factors that matter most for Node:

- **Config in environment** — never in code
- **Stateless processes** — don't store anything in memory between requests; use Redis/DB
- **Port binding** — your app listens on a port; the platform routes traffic to it
- **Disposability** — fast startup, graceful shutdown (handle SIGTERM)
- **Dev/prod parity** — same DB engine, same Node version, same dependencies, locally and in prod (Docker helps a lot here)

### Graceful shutdown

When your platform sends SIGTERM (deploys, scaling down), you have ~10 seconds to:
1. Stop accepting new connections
2. Finish in-flight requests
3. Close DB connections
4. Exit

```js
process.on('SIGTERM', async () => {
  server.close(() => { /* finish in-flight */ });
  await db.close();
  process.exit(0);
});
```

Without this, you drop user requests on every deploy.

---

## Patterns I'll add as I go

_(Each section of the course will produce 2-5 entries here. Update during Sunday reviews.)_

### Pattern: [name]
**Context:** when this pattern applies
**Solution:** what you do
**Trade-offs:** what it costs
**Example:** small code snippet
