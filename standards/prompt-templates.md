# Prompt Templates — The Upgraded PRD-to-Code Workflow

> **The problem with my old workflow:** I went from problem statement → PRD → chunked prompts → code. That skips the architecture step, so Claude makes implicit architectural decisions chunk-by-chunk, and they drift. Halfway through the project, the auth in `users` works differently than the auth in `admin`.
>
> **The fix:** add an explicit architecture step and post-chunk review checkpoints.

---

## The 5-step workflow

```
1. Problem Statement      (you write)
       ↓
2. PRD                    (Claude generates, you review)
       ↓
3. Architecture Document  (Claude generates, locked-in decisions)
       ↓
4. Chunked Prompts        (each chunk references architecture)
       ↓
5. Code + Review          (audit each chunk against CLAUDE.md)
```

---

## Step 1 — Problem Statement (template)

Keep this short. One paragraph. Then let Claude expand it.

```
PROBLEM STATEMENT

I'm building [type of app] for [target user]. The core problem it solves is [problem].

The 3 most important things it must do:
1. ...
2. ...
3. ...

Constraints:
- Tech stack preferences: [Node + Express + Postgres + Drizzle, or whatever]
- Deployment target: [Vercel / Railway / VPS / client's infra]
- Timeline: [X weeks]
- Scale: [expected users in year 1]

Non-goals (things this app explicitly does NOT need to do):
- ...
- ...
```

---

## Step 2 — PRD generation prompt

```
You are a senior product manager. Based on the problem statement below, 
generate a Product Requirements Document with these sections:

1. Overview (one paragraph)
2. Target users and primary use cases
3. Functional requirements (numbered list, each testable)
4. Non-functional requirements (performance, security, accessibility)
5. User stories (As a... I want to... so that...)
6. Out of scope (explicit non-goals)
7. Success metrics

Be specific. Use concrete numbers (e.g. "support 1000 concurrent users", 
not "support many users"). Flag any assumptions you're making.

PROBLEM STATEMENT:
[paste from step 1]
```

**After Claude generates the PRD, review it carefully.** Ask follow-ups, refine, then save the final version as `docs/PRD.md` in the project repo.

---

## Step 3 — Architecture Document prompt (THE NEW STEP)

This is what was missing before. Save the output as `docs/ARCHITECTURE.md`.

```
You are a senior backend engineer. Based on the attached PRD and the 
attached CLAUDE.md (my coding standards), produce an architecture document.

The document MUST include:

1. **Tech stack decisions**
   - Each choice (DB, ORM, auth strategy, etc.) with the trade-off explained
   - Versions pinned

2. **Folder structure**
   - Complete directory tree
   - One-line purpose for each folder

3. **Database schema**
   - Every table/collection with fields, types, constraints
   - Relationships, indexes, foreign keys
   - Express any non-obvious decisions (denormalization, soft deletes, etc.)

4. **API contract**
   - Every endpoint: method, path, auth requirement, request shape, response shape
   - Group by resource

5. **Authentication & authorization strategy**
   - Token type, expiry, storage location
   - Role/permission model
   - Which routes need which permissions

6. **Error handling approach**
   - Error class hierarchy
   - Response shape for errors
   - Logging strategy

7. **Security considerations**
   - Specific OWASP Top 10 items addressed and how
   - Rate limiting strategy
   - Secrets management

8. **Open questions / decisions needed from me**

Do NOT write any application code yet. This is purely the architectural blueprint.

PRD: [attach or paste]
STANDARDS: [attach CLAUDE.md]
```

**Review the architecture doc. Push back on anything that feels off.** This is the cheapest moment to change direction. Once code is written, changes are 10x more expensive.

---

## Step 4 — Chunked prompts

Now break the build into shippable chunks. Each chunk:
- Builds something verifiable end-to-end (don't chunk by layer — chunk by feature)
- References the architecture doc + CLAUDE.md
- Is small enough that you can review it in one sitting

**Chunking prompt:**

```
Based on the architecture document, break the implementation into 
shippable chunks. Each chunk must:
- Deliver a working, testable slice of functionality (no "skeleton only" chunks)
- Be completable in one focused work session
- List exact files it creates or modifies
- List dependencies it requires from previous chunks

Output as a numbered list. Don't write code yet.
```

**Per-chunk implementation prompt template:**

```
Implement chunk [N]: [chunk name]

Context:
- This is part of [project name]
- Architecture doc: [attach or paste relevant sections]
- Coding standards: [attach CLAUDE.md]
- Already implemented: chunks [1..N-1]

Requirements for this chunk:
[paste chunk requirements from the chunking output]

Constraints:
- Follow CLAUDE.md exactly. If you deviate, flag it explicitly.
- Don't create files outside the agreed folder structure.
- Don't change existing files unless required by this chunk.
- After implementing, list:
  (a) Files created/modified
  (b) Any deviations from CLAUDE.md and why
  (c) Anything in the chunk requirements you couldn't fully implement
```

---

## Step 5 — Post-chunk review

This is where trust gets built. After each chunk:

**Audit prompt:**

```
Audit the code you just wrote in this chunk against CLAUDE.md.

For each file you created or modified, list:
- Which CLAUDE.md rules it follows correctly
- Any rule it deviates from (and why)
- Any rule that doesn't apply
- Any potential issue you'd flag if reviewing this in production code review

Then run the "what if" review for each new endpoint:
- What if user is not authenticated?
- What if input is malformed?
- What if user is accessing someone else's resource?
- What if the DB call fails halfway?
- What if this is called 10,000 times/minute?

Be honest. Don't defend the code — review it.
```

**Manual checks (you, not Claude):**
- [ ] Does it actually run? `npm run dev` and hit the endpoints in Postman.
- [ ] Are the tests in the chunk passing?
- [ ] Did it modify files it shouldn't have?
- [ ] Spot-check one route end-to-end: schema → controller → service → DB → response

---

## Step 6 — Project completion postmortem

After the project ships, run this once to extract lessons:

```
Reflect on this project. List:

1. Things that worked well in the implementation
2. Things you (Claude) got wrong on the first try and had to fix
3. Specific patterns I should add to my CLAUDE.md as new rules 
   (formatted as "ALWAYS X" or "NEVER Y")
4. Anti-patterns I should add to my backend-playbook.md
5. Things you would do differently if starting over

Be specific. Reference exact files/decisions.
```

**Take the output of (3) and (4), edit them down, and add to your standards files.** Each project makes the next one better.

---

## Quick reference: prompts to keep handy

| Situation | Prompt |
|---|---|
| Stuck debugging | "Here's the error and the relevant code. Walk me through what's happening, then propose 3 possible causes ranked by likelihood." |
| Reviewing AI code | "Audit this code against CLAUDE.md. Flag every deviation." |
| Performance concern | "Identify the top 3 performance bottlenecks in this code. Suggest fixes ranked by effort vs impact." |
| Security review | "Run this code through the OWASP Top 10. Flag any concerns." |
| Refactor request | "Refactor this for clarity without changing behavior. List what you changed and why." |
| Test coverage | "Generate Postman test cases for this endpoint covering: happy path, auth failure, validation failure, ownership violation." |
