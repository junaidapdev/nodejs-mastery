# Security Review Checklist

> **Purpose:** Run through this list before delivering any backend project to a client. Every "no" is either a fix, a documented exception, or a reason not to ship.
>
> Aligned with OWASP Top 10 (2021) plus Node-specific concerns.

---

## Pre-deploy audit (run for every project)

### 🔐 Authentication & session management

- [ ] Passwords hashed with bcrypt, cost factor ≥ 10 (never plain SHA, MD5, or unsalted)
- [ ] No password length cap below 64 chars; minimum length ≥ 8
- [ ] Login endpoint rate-limited (e.g. 5 attempts per 15 min per IP)
- [ ] Signup endpoint rate-limited and CAPTCHA-protected if public
- [ ] Password reset uses single-use tokens with short expiry (≤ 1 hour)
- [ ] Password reset doesn't reveal whether an email exists ("If this email is registered, we've sent a link")
- [ ] JWT secret is 32+ random bytes, stored in env, rotated periodically
- [ ] JWT access tokens expire in ≤ 1 hour; refresh tokens in ≤ 30 days
- [ ] Logout actually invalidates the session/refresh token
- [ ] Session cookies set with `httpOnly`, `secure` (in prod), `sameSite`

### 🛡️ Authorization

- [ ] Every endpoint that touches user data verifies the user owns / can access that data (not just authenticated)
- [ ] Role/permission checks are in middleware, not duplicated in controllers
- [ ] Admin endpoints exist on a separate route group with extra verification
- [ ] No endpoint relies solely on "obscure URL = secure" — every protected route has explicit auth checks

### 💉 Injection prevention

- [ ] All DB queries use ORM/parameterized queries — NO string concatenation with user input
- [ ] All `req.body`, `req.params`, `req.query` validated with Zod/Joi before use
- [ ] No `eval()`, `new Function()`, or `child_process.exec()` with user-controlled input
- [ ] File path inputs sanitized — no `..` traversal possible
- [ ] If using MongoDB: `$where` operators avoided; user input not directly placed in query objects

### 🍪 Headers & transport

- [ ] `helmet()` middleware enabled
- [ ] CORS configured with explicit origin list — no `*` in production
- [ ] HTTPS enforced (handled at infra layer; verify in prod)
- [ ] HSTS header set (helmet does this)
- [ ] CSP header configured if app serves HTML
- [ ] X-Frame-Options set (helmet defaults are fine)

### 📝 Input/output handling

- [ ] All input validated server-side (client-side validation is UX only, not security)
- [ ] Output encoded properly when reflected to HTML/JSON
- [ ] File uploads: size-limited, type-validated (magic bytes, not just extension), stored outside web root
- [ ] User-generated content sanitized before rendering

### 🔒 Secrets management

- [ ] No secrets in code or git history (run `git log -p | grep -i secret` to verify)
- [ ] `.env` in `.gitignore`
- [ ] `.env.example` documents required vars without values
- [ ] Production secrets rotated from any defaults
- [ ] No secrets in Docker images (passed at runtime via env or secret manager)
- [ ] No secrets in client-bundled JS

### 📊 Logging & monitoring

- [ ] Real logger used (Winston/Pino) — no `console.log` in production paths
- [ ] Logs include request id for tracing
- [ ] Authentication events logged (login success, failure, logout, password change)
- [ ] Authorization failures logged with user id + attempted resource
- [ ] **Logs do NOT contain:** passwords, tokens, full credit cards, full SSNs, request bodies of auth endpoints
- [ ] Error responses to clients don't leak stack traces in production

### 🗄️ Database

- [ ] Foreign keys defined with explicit `onDelete` behavior
- [ ] Indexes on foreign keys + frequently queried columns
- [ ] DB user has minimum required permissions (no `SUPERUSER` / no `db.adminCommand` for app user)
- [ ] DB backups configured and tested (a backup you haven't restored is not a backup)
- [ ] Sensitive columns (PII, financial data) considered for encryption at rest
- [ ] Migrations versioned in source control, never edited after merge

### 🌐 API surface

- [ ] No debug/admin endpoints exposed in production (e.g. `/admin/eval`, `/debug`)
- [ ] No `/health` endpoint that leaks version info, env vars, or DB structure
- [ ] Error responses use consistent shape; never expose internal field names or stack traces
- [ ] Pagination on all list endpoints — no unbounded responses
- [ ] Rate limiting on all endpoints; stricter on auth + write endpoints

### 📦 Dependencies

- [ ] `npm audit` shows no high/critical vulnerabilities
- [ ] No `git+http://` or unsigned package sources
- [ ] Lock file (`package-lock.json`) committed
- [ ] Dependencies pinned to exact versions or caret ranges that don't include majors
- [ ] No suspicious packages (check downloads, last update, maintainer history)

### 🐳 Docker / deployment

- [ ] Container runs as non-root user
- [ ] Multi-stage build — production image doesn't contain build tools, dev dependencies
- [ ] Only required ports exposed
- [ ] No secrets baked into image — passed via env at runtime
- [ ] Base image is official + recent (`node:20-alpine` not `node:14`)
- [ ] `.dockerignore` excludes `node_modules`, `.env`, `.git`, `dist`, tests

### 🚨 Operational

- [ ] Process manager (pm2, Docker, k8s) restarts on crash
- [ ] `process.on('unhandledRejection')` and `uncaughtException` log + exit cleanly
- [ ] Graceful shutdown on SIGTERM
- [ ] Health check endpoint returns 200 if app is healthy
- [ ] Monitoring/alerting set up for: error rate spikes, response time, DB connectivity

---

## The "what if" review (do this for every endpoint)

Before marking an endpoint done, mentally answer each:

1. **What if the user is not authenticated?** → returns 401, no data leak
2. **What if the user is authenticated but accessing someone else's resource?** → returns 403 or 404, no data leak
3. **What if input is missing/malformed/oversized?** → returns 400 with clear validation error
4. **What if input is valid but contains injection payloads (`<script>`, `'; DROP TABLE`, `{$gt: ''}`)?** → safely handled by ORM/validators
5. **What if this endpoint is called 10,000 times in a minute?** → rate limiter blocks; DB doesn't fall over
6. **What if the DB call fails halfway through a multi-step operation?** → transaction rolls back, no partial state
7. **What if two users perform the same action simultaneously?** → no race conditions (use transactions, optimistic locking, or unique constraints)
8. **What if this endpoint runs slow?** → no unbounded queries; pagination present; timeout configured

---

## Incident response prep

- [ ] Know how to revoke all sessions / invalidate all JWTs in an emergency
- [ ] Know how to rotate the JWT signing secret
- [ ] Know how to take the app down for maintenance (and have a maintenance page ready)
- [ ] Know how to roll back the last deployment
- [ ] Know how to restore the DB from the most recent backup
- [ ] Have client communication plan for security incidents (who to contact, what to say)

---

## Sign-off

Project: ____________________
Date: ____________________
Reviewer: ____________________

- [ ] All applicable items above checked
- [ ] Documented exceptions for any unchecked items: _______________
- [ ] Client briefed on remaining risks (if any)
