# Node.js Mastery — Backend Engineering from Scratch

> A 4–6 week deep-dive into Node.js, Express, Postgres/Drizzle, MongoDB, Auth, Docker, and System Design.
> Goal: build production-grade backend skills so I can confidently deliver serious business apps to clients.

**Started:** _[fill in your start date]_
**Target completion:** _[start date + 6 weeks]_
**Course:** Node.js — Beginner to Advanced with Projects (Hitesh Choudhary & Piyush Garg)

---

## Why this repo exists

I build apps with AI assistance (Claude Code). AI writes code fast — but **I'm the one accountable to clients** for security, architecture, and what happens when something breaks. This repo is how I close the gap between "code that works" and "code I can defend."

Three deliverables come out of this:
1. **`standards/CLAUDE.md`** — imperative rules I paste into every Claude Code project
2. **`standards/backend-playbook.md`** — human-readable reference for code reviews
3. **`standards/security-checklist.md`** — OWASP-aligned audit list for client work

---

## How to navigate this repo

| Folder | What's in it |
|---|---|
| `standards/` | The reusable artifacts I'm building. These outlive the course. |
| `notes/` | Section-by-section learning notes (3-layer template) |
| `projects/` | The two big builds I do myself: URL Shortener and Basecamp clone |
| `code-along/` | Small snippets I typed during lectures |
| `reviews/` | Weekly Sunday retrospectives |

---

## Progress tracker

- [ ] **Week 1** — Node.js Core, Events/Buffers, Native HTTP _(target: ___)_
- [ ] **Week 2** — Express Fundamentals + Advanced (Middleware, MVC) _(target: ___)_
- [ ] **Week 3** — Postgres + Drizzle ORM + URL Shortener project _(target: ___)_
- [ ] **Week 4** — Authentication & Authorization + Security deep dive _(target: ___)_
- [ ] **Week 5** — MongoDB, Aggregation, Docker _(target: ___)_
- [ ] **Week 6** — Basecamp Mega Project + System Design _(target: ___)_

---

## Daily log

Format: `Date | Section watched | Time spent | Key takeaway | Blocker (if any)`

| Date | Section | Time | Takeaway | Blocker |
|------|---------|------|----------|---------|
| | | | | |

---

## Daily ritual (2 hrs)

| Block | Time | What I do |
|---|---|---|
| Watch | 60–75 min | Course at 1.25–1.5x. Pause only for compounding topics. |
| Type-along | 20–30 min | Re-do compounding parts myself, no video |
| Notes | 15–20 min | Fill the 3-layer template in `notes/` |
| Harvest | 5 min | Add new patterns/anti-patterns to `CLAUDE.md` |

**What "compounds" — type these every time:**
- Auth flows (sessions, JWT, middleware)
- Database schemas, migrations, complex queries
- Middleware chains and error handling
- Project file/folder structure
- `try/catch`, async errors, process-level handlers
- Dockerfiles and docker-compose

**What to skim/skip:**
- Install/setup lectures
- Pure theory intros where I already know the basics
- Repetitive CRUD once the pattern is clear

---

## Weekly ritual (Sunday, 30 min)

1. Re-read the week's notes
2. Update `notes/_index.md` glossary with new vocabulary
3. Harvest patterns into `standards/CLAUDE.md`
4. Write `reviews/week-N.md` retrospective
5. Commit + push everything
6. **Friday recall test** (15 min): write what I remember from blank, then compare to notes. Gap = what I need to re-study.

---

## Resources

- Course: [Udemy link]
- OWASP Top 10: https://owasp.org/www-project-top-ten/
- 12-Factor App: https://12factor.net/
- Node.js docs: https://nodejs.org/docs/latest/api/
- Express docs: https://expressjs.com/
- Drizzle ORM docs: https://orm.drizzle.team/
