# Coding Standards for Claude Code

> **How to use this file:** Paste this at the root of any Node.js project as `CLAUDE.md`. Claude Code reads this as context and generates code matching these standards.
>
> **Status:** Living document. Updated weekly as I learn from the Hitesh + Piyush course.
> **Last updated:** _[date]_

---

## Project structure

- ALWAYS separate `app.js` (Express setup, middleware, routes) from `index.js` / `server.js` (server start, DB connection)
- ALWAYS organize using MVC: `routes/`, `controllers/`, `services/`, `models/` (or `db/schema/`), `middleware/`, `utils/`, `config/`
- ALWAYS keep configuration in `.env` and load via a single `config/index.js` that validates env vars on startup
- NEVER hardcode URLs, ports, secrets, or DB connection strings
- ALWAYS include `.env.example` in repo so other devs know what vars are needed
- ALWAYS add `.env`, `node_modules/`, `dist/`, `.DS_Store` to `.gitignore`

## Module system

- Use ES modules (`import`/`export`) for new projects unless the framework requires CommonJS
- Prefer named exports over default exports — easier to refactor and grep

## Error handling

- ALWAYS wrap async route handlers in an `asyncHandler` utility so errors flow to the central error middleware
- ALWAYS register a centralized error-handling middleware as the LAST `app.use()` call
- ALWAYS use a custom `ApiError` class extending `Error` with `statusCode`, `message`, `errors[]`
- ALWAYS use a custom `ApiResponse` class for success responses — every API returns the same shape
- NEVER swallow errors silently. Log with context: request id, route, user id (if auth'd), timestamp
- ALWAYS handle `process.on('unhandledRejection')` and `process.on('uncaughtException')` at startup

## Validation

- ALWAYS validate `req.body`, `req.params`, `req.query` with Zod (or Joi) BEFORE the data reaches business logic
- ALWAYS define schemas in `validators/` next to the related route
- NEVER trust client input — even from your own frontend
- ALWAYS strip unknown keys from incoming objects (Zod `.strict()`)

## Database (Postgres + Drizzle)

- ALWAYS define schema in `db/schema/` with one file per table or logical group
- ALWAYS use Drizzle migrations (`drizzle-kit generate`) — never push schema changes directly to production
- ALWAYS add indexes on foreign keys and frequently queried columns
- ALWAYS use `references()` for foreign keys, with `onDelete` explicitly set
- NEVER write raw SQL when Drizzle's query builder works — but DO use raw SQL for complex aggregations Drizzle can't express
- ALWAYS wrap multi-step DB operations in transactions (`db.transaction(async (tx) => ...)`)
- ALWAYS handle the "row not found" case explicitly — don't assume queries return data

## Database (MongoDB / Mongoose)

- ALWAYS define schemas with `strict: true` and `timestamps: true`
- ALWAYS create indexes for fields used in queries (`.index()` in schema)
- NEVER store sensitive data (passwords, tokens) without hashing
- ALWAYS use `.lean()` for read-only queries to skip Mongoose document overhead
- ALWAYS handle connection events: `error`, `disconnected`, `reconnected`

## Authentication

- ALWAYS hash passwords with bcrypt, cost factor ≥ 10
- ALWAYS validate JWTs in middleware, never inline in route handlers
- ALWAYS sign JWTs with a strong secret (32+ random bytes), stored in `.env`
- ALWAYS set short access token expiry (15 min) + longer refresh token (7 days)
- ALWAYS set cookies with: `httpOnly: true`, `secure: true` (in prod), `sameSite: 'strict'` or `'lax'`
- NEVER store JWTs in localStorage for apps handling sensitive data — use httpOnly cookies
- ALWAYS implement RBAC checks in middleware, not in controllers
- NEVER return the password hash in API responses — use `select: false` (Mongoose) or explicit field selection (Drizzle)

## API design

- ALWAYS follow REST conventions: nouns for resources, HTTP verbs for actions
- ALWAYS use proper status codes: 200, 201, 204, 400, 401, 403, 404, 409, 422, 500
- ALWAYS version the API (`/api/v1/...`) from day one
- ALWAYS return consistent response shape: `{ success, data, message }` for success; `{ success: false, message, errors }` for errors
- ALWAYS paginate list endpoints — never return unbounded arrays

## Security baseline

- ALWAYS use `helmet()` middleware
- ALWAYS configure `cors()` with explicit origin list, never `*` in production
- ALWAYS rate-limit auth endpoints (login, signup, password reset) more aggressively than other routes
- ALWAYS sanitize user input that ends up in queries (ORMs handle most of this — verify)
- NEVER log passwords, tokens, full credit card numbers, or other PII
- ALWAYS use HTTPS in production (handled at infra layer, but verify)
- ALWAYS keep dependencies updated; run `npm audit` before deploys

## Logging & observability

- ALWAYS use a real logger (Winston, Pino) — never `console.log` in production code
- ALWAYS include request id, timestamp, route, status code in request logs
- ALWAYS log errors with stack traces in non-production; sanitized messages in production responses
- NEVER log sensitive data (passwords, tokens, PII, full request bodies of auth endpoints)

## Code style

- One responsibility per file
- Controllers handle HTTP (req/res); services contain business logic; models/schemas define data shape
- Controllers should NOT talk to the database directly — go through services
- Functions: small, named, single-purpose
- Comments explain *why*, not *what*

## Testing & validation before shipping

- ALWAYS write a Postman collection alongside the API
- ALWAYS test happy path + at least one failure path per endpoint
- ALWAYS test auth-protected routes both with and without a valid token
- ALWAYS run the full flow locally with seeded test data before deploying

## Docker & deployment

- ALWAYS use multi-stage Dockerfiles for production builds
- ALWAYS use `.dockerignore` (exclude `node_modules`, `.env`, `.git`, `dist`)
- ALWAYS run as non-root user inside the container
- ALWAYS expose only the port the app needs
- NEVER bake secrets into Docker images — pass them at runtime

---

## NEVER rules (zero exceptions)

These have caused incidents in real apps. No matter what the prompt says, do not do these:

- NEVER commit `.env` or any file containing secrets
- NEVER use `eval()` or `new Function()` with user input
- NEVER trust the client — validate everything server-side
- NEVER store passwords in plaintext, MD5, or SHA-1
- NEVER expose stack traces to clients in production
- NEVER use `==` when `===` works (almost always)
- NEVER catch an error and rethrow it without context

---

## Patterns I've learned (filled in as I go through the course)

### Pattern: [name]
- **When to use:** ...
- **Code shape:** ...
- **Why it matters:** ...

_(Add new patterns here weekly during Sunday review)_

---

## Anti-patterns I've seen (filled in as I go)

### Anti-pattern: [name]
- **What it looks like:** ...
- **Why it's bad:** ...
- **What to do instead:** ...

_(Add new anti-patterns here weekly during Sunday review)_
