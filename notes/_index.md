# Notes Index — Backend Vocabulary

> **Purpose:** Build the keyword muscle I'm missing. Every new term goes here with a one-line definition in MY OWN words.
> When I forget what something means, this is the first place I look.

---

## How to use this file

1. As I encounter a new term in any section, add it here immediately
2. Definition must be in my own words, not copy-pasted from docs
3. If I can't write a one-line definition, I don't understand it yet — go back to the lecture
4. Include a tiny code example or analogy where useful

---

## A

- **Asynchronous** — code that doesn't wait; the result comes later via callback/promise/await
- **Authentication (authn)** — proving who you are (username + password, token, etc.)
- **Authorization (authz)** — proving what you're allowed to do (admin? owner? read-only?)

## B

- **bcrypt** — slow-by-design password hashing function with built-in salt; standard for storing passwords
- **Buffer** — Node's way of handling raw binary data (file contents, network packets, etc.)

## C

- **CORS (Cross-Origin Resource Sharing)** — browser security rule that controls which domains can call your API
- **CRUD** — Create, Read, Update, Delete — the four basic data operations

## D

- **Drizzle ORM** — a TypeScript-friendly SQL query builder for Node; lighter than Prisma
- **Docker** — tool for packaging an app + its dependencies into a portable container

## E

- **Event Loop** — Node's mechanism for handling async operations on a single thread
- **EventEmitter** — Node class for the publish/subscribe pattern
- **Express** — minimal web framework for Node; provides routing + middleware + req/res helpers

## F

- **Foreign Key** — a column in one table that references the primary key of another, defining a relationship

## H

- **Helmet** — Express middleware that sets security-related HTTP headers
- **HTTP methods** — GET, POST, PUT, PATCH, DELETE, etc. — verbs telling the server what to do

## I

- **Index (database)** — a B-tree structure that makes queries faster on indexed columns; trade-off is slower writes
- **Idempotent** — calling it multiple times has the same effect as calling it once (GET, PUT, DELETE are; POST usually isn't)

## J

- **JWT (JSON Web Token)** — a signed token containing claims; verify the signature and you can trust the contents

## M

- **Middleware** — a function that runs between the request arriving and the response being sent; can modify req/res or pass to the next
- **Migration** — a versioned file describing a schema change; applied in order to bring a DB to the desired state
- **Mongoose** — popular ODM for MongoDB; adds schemas, validation, middleware on top of the raw driver
- **MVC** — Model-View-Controller; in Node APIs this becomes Routes-Controllers-Services-Models

## N

- **npm / package.json** — Node's package manager / the manifest file describing dependencies and scripts

## O

- **ORM (Object-Relational Mapper)** — translates between objects in code and rows in a relational database

## R

- **RBAC (Role-Based Access Control)** — permissions tied to roles (admin, editor, viewer), users assigned to roles
- **REST** — architectural style: predictable URLs, HTTP verbs, stateless requests

## S

- **Salt** — random data added to a password before hashing so identical passwords produce different hashes
- **Schema** — the definition of data shape (tables/collections, fields, types, constraints)
- **Stateless** — each request is independent; server stores no session data between requests

## T

- **Transaction** — a sequence of DB operations treated as one unit; either all succeed or all roll back

## Z

- **Zod** — TypeScript-first schema validation library; define a schema, validate any input against it

---

_(Add new terms as I encounter them. Sort alphabetically when there's time during Sunday review.)_
