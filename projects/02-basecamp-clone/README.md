# Project 2: Basecamp Clone — Project Management Platform

> **Built during:** Week 6 of Node.js Mastery (the capstone)
> **Tech stack:** Node.js, Express, MongoDB, Mongoose, JWT, Express middleware, professional project structure
> **Goal:** Production-quality backend with everything I've learned. This is the portfolio piece.

---

## Why this is the capstone

The course spends 6hr 41min on this project across 36 lectures. It includes:
- A real PRD (the course teaches reading PRDs)
- Professional project structure
- Auth with middleware and JWT
- MongoDB with Mongoose
- Standard API response/error patterns
- Healthcheck endpoints
- Async error handling
- CORS configuration

This isn't just "another CRUD." It's the project I show clients to prove I know what I'm doing.

---

## Approach

Different from URL Shortener. For this one:
- **Watch lectures first** because the project is large enough that going blind would waste days
- **Pause and predict** before each implementation lecture: "what would I do here?"
- **Type along** for the architecture/structure parts (these are the highest-value habits to internalize)
- **Skip type-along** for repetitive CRUD once the pattern is clear
- **Add features beyond the course** at the end — that's how I prove I really learned

---

## Features (from course scope)

- Backend-focused project with proper structure (`controllers/`, `routes/`, `models/`, `middleware/`)
- Auth (signup, login, JWT)
- CORS configuration
- Standard `ApiResponse` and `ApiError` classes
- Async handler wrapper
- Healthcheck endpoint
- MongoDB connection with proper error handling
- (Plus whatever else the lectures introduce)

---

## My additions (beyond what the course teaches)

_To prove deep understanding, I'll add these AFTER finishing the course version:_

- [ ] Rate limiting on auth endpoints
- [ ] Refresh token rotation (not just access token)
- [ ] Postman collection covering happy + failure paths
- [ ] Dockerfile + docker-compose with MongoDB
- [ ] Logging with Winston/Pino instead of console.log
- [ ] Run security checklist from `standards/security-checklist.md` and document results

---

## My postmortem (filled in when done)

### What I built that wasn't in the course
-

### Patterns to add to CLAUDE.md
-

### Real-world client scenarios this prepares me for
-

### Confidence check
- [ ] I can rebuild this from scratch without notes
- [ ] I can explain every middleware in the chain to a junior dev
- [ ] I can identify the security implications of every endpoint
- [ ] I can deploy this to production without help

---

## Files

```
02-basecamp-clone/
├── README.md
├── PRD.md              ← the PRD from the course (or my version)
├── ARCHITECTURE.md
├── src/
├── postman/
├── Dockerfile
└── docker-compose.yml
```
