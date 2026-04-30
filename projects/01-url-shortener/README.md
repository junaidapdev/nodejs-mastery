# Project 1: URL Shortener

> **Built during:** Week 3 of Node.js Mastery
> **Tech stack:** Node.js, Express, PostgreSQL, Drizzle ORM, JWT, Zod
> **Goal:** Build it MYSELF first, then watch the course solution and compare.

---

## Why I'm building this

This is the first project where I prove I can hold a real architecture in my head — auth, DB, routing, validation, error handling all working together. The course shows the solution in 1hr 29min across 18 lectures. My approach: try to build it before watching, then compare my solution to the instructor's, then merge the best of both.

---

## Approach

1. **Read the course's lecture titles** to understand scope (don't watch yet)
2. **Write a PRD** for myself using my prompt-templates.md
3. **Generate an Architecture Document** using my workflow
4. **Build chunk by chunk** without watching the course
5. **When stuck:** watch the specific lecture I need, take notes on the difference
6. **After done:** watch the full section, do a postmortem

---

## Features (from course scope)

- User signup with hashed passwords
- User login returning a JWT
- Authenticated middleware that attaches user to request
- Create a short URL from a long URL (auth required)
- Redirect from short URL to long URL (public)
- User can view their shortened URLs (auth required)
- Input validation with Zod throughout

---

## My architecture (filled in when I start)

_(Generate this using the Architecture Document prompt from `standards/prompt-templates.md`)_

---

## My postmortem (filled in when done)

### What I got right
-

### What the course did better
-

### Patterns to add to CLAUDE.md
-

### Patterns to add to backend-playbook.md
-

---

## Files

```
01-url-shortener/
├── README.md           ← this file
├── PRD.md              ← the product requirements I wrote
├── ARCHITECTURE.md     ← the architecture doc Claude generated
├── src/                ← actual code (when built)
└── postman/            ← Postman collection
```
