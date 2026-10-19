# ⚙️ Backend Depth — From React Dev to Full-Stack Engineer

> **Goal:** close your biggest visible gap (backend) so you can (1) pass full-stack interviews at product companies and (2) turn **Habex** ([habex.santhu.site](https://habex.santhu.site)) from a frontend-only app into a **real full-stack product** — your strongest interview story.
> **For:** a strong React/TypeScript dev who knows almost nothing about servers, databases, or deployment yet. Every backend term is defined the first time.
> **Part of** your [Skill-Up Hub](../README.md). This is the **hands-on implementation** companion to the concept-level [System Design track](../system-design/00-README.md).

---

## The mindset shift

As a frontend dev you've been the *client*: you `fetch()` data and render it. Backend is the other side of that call — the program that **receives the request, checks who you are, runs the business logic, talks to a database, and sends data back.** That's it. You already know the hard language part (TypeScript); now you learn where the data actually lives and how the server thinks.

By the end you'll have **built and deployed a complete Habex backend**: auth, a PostgreSQL database, a REST API, caching, background jobs for recurring expenses, tests, and a Docker setup running in the cloud.

---

## One pinned stack (so everything composes)

Every chapter uses the **same tools**, so the code snippets add up to one coherent Habex backend instead of 24 disconnected demos:

| Layer | Choice | Why |
|---|---|---|
| Language | **TypeScript** | Your strength; types catch bugs server-side too |
| Runtime | **Node.js** (LTS) | You already know JS; huge ecosystem |
| Web framework | **Express** | The interview-standard; simple and everywhere (we note Fastify/NestJS as alternatives) |
| Database | **PostgreSQL** | The default relational DB at product companies |
| DB access | **Prisma** (+ raw SQL to learn fundamentals) | TS-first ORM; type-safe queries |
| Validation | **Zod** | TS-native input validation |
| Auth | **JWT** (access+refresh) + **bcrypt** | Standard token auth; we compare with sessions |
| Cache & queues | **Redis** + **BullMQ** | Caching + background jobs |
| Logging | **pino** | Fast structured logs |
| Testing | **Vitest** + **supertest** | Fast, TS-friendly API testing |
| Containers | **Docker** + docker-compose | App + Postgres + Redis locally and in prod |
| Deploy | **Railway / Render** + **GitHub Actions** | Easy managed hosting + CI |

---

## The Habex data model (used throughout)

Every schema/query example uses this same model, so the capstone is just "assemble the pieces":

```
users(id, email UNIQUE, password_hash, name, created_at)
categories(id, user_id → users, name, kind['income'|'expense'], color)
transactions(id, user_id → users, category_id → categories,
             amount_cents INT, kind['income'|'expense'], note, occurred_at, created_at)
recurring_rules(id, user_id, category_id, amount_cents, kind,
                cadence['daily'|'weekly'|'monthly'], next_run_at, active)
budgets(id, user_id, category_id, month['YYYY-MM'], limit_cents)
```

> **Correctness rule drilled all track long:** store money as **integer cents** (`amount_cents`), never floating point. `0.1 + 0.2 !== 0.3`. This exact point matters at fintech companies (CRED, Razorpay, Groww).

---

## The curriculum

### Part I — Backend Foundations
- [01 — What Is a Backend?](01-what-is-backend.md) — the mental model, from a frontend dev's POV
- [02 — The Node.js Runtime & Event Loop](02-nodejs-runtime-and-event-loop.md) — how Node does async without threads (big interview topic)
- [03 — Project Setup: TypeScript + Node](03-project-setup-typescript-node.md) — package.json, tsconfig, structure, env config

### Part II — Building APIs with Express
- [04 — HTTP Servers & Express Basics](04-http-servers-and-express-basics.md) — routes, req/res, your first API
- [05 — Middleware & the Request Lifecycle](05-middleware-and-request-lifecycle.md) — the chain that powers auth, logging, errors
- [06 — REST API Design in Practice](06-rest-api-design-in-practice.md) — CRUD, status codes, pagination, versioning
- [07 — Input Validation & Error Handling](07-input-validation-and-error-handling.md) — Zod, centralized errors, safe responses

### Part III — Databases (PostgreSQL)
- [08 — SQL & PostgreSQL Fundamentals](08-sql-and-postgresql-fundamentals.md) — tables, CRUD, joins, constraints
- [09 — Data Modeling & Migrations](09-data-modeling-and-migrations.md) — schema design, relationships, migrations
- [10 — Connecting Node to PostgreSQL](10-connecting-node-to-postgres.md) — the `pg` driver and Prisma
- [11 — Transactions & Query Performance](11-transactions-and-query-performance.md) — ACID in code, pooling, indexes, the N+1 trap

### Part IV — Auth & Security
- [12 — Authentication](12-authentication.md) — login/signup, password hashing, JWT access+refresh
- [13 — Authorization & Security](13-authorization-and-security.md) — RBAC, OWASP basics, CORS, secrets

### Part V — Performance, Async & Realtime
- [14 — Caching with Redis](14-caching-with-redis.md) — cache-aside, sessions, rate limiting
- [15 — Background Jobs & Queues](15-background-jobs-and-queues.md) — BullMQ, workers, cron, retries, idempotency
- [16 — Realtime with WebSockets](16-realtime-websockets.md) — Socket.IO, when to use it, scaling

### Part VI — Production-Readiness
- [17 — Logging, Monitoring & Observability](17-logging-monitoring-observability.md) — pino, health checks, error tracking
- [18 — Testing the Backend](18-testing-backend.md) — unit, integration, supertest, a test DB
- [19 — Docker & Containers](19-docker-and-containers.md) — Dockerfile, docker-compose (app + Postgres + Redis)
- [20 — Deployment & DevOps](20-deployment-and-devops.md) — Railway/Render, env config, CI/CD, 12-factor

### Part VII — Putting It Together
- [21 — Architecture & Project Structure](21-architecture-and-project-structure.md) — controllers/services/repositories, monolith-first
- [22 — Capstone: The Habex Backend, End to End](22-capstone-habex-backend.md) — assemble everything on your own app
- [23 — Backend Interview Prep](23-backend-interview-prep.md) — the questions, per-company focus, telling your Habex story

---

## How to use this track

1. **Type the code, don't just read it.** Backend clicks when you run it. Start a `habex-api` project in chapter 3 and grow it every chapter.
2. **Follow the Habex thread.** Each chapter's *"Apply It to Habex"* section adds one real piece to the same app. By chapter 22 you have a deployed backend.
3. **Pair with system design.** When a chapter says "see [../system-design/07-caching.md](../system-design/07-caching.md)", that's the *why* behind the *how* you're building here.
4. **Keep DSA warm** in parallel (the [dsa/](../dsa/00-README.md) track) — the two interview gates never sleep.

---

## What each company probes on backend

| Company | Backend focus |
|---|---|
| **Zepto / Swiggy** | High-throughput APIs, caching, DB indexing, queues for order/event flows |
| **CRED / Razorpay** | Correctness (money = integer cents!), transactions, idempotency, auth/security, webhooks |
| **Groww** | Data consistency, transactions, clean REST APIs, reliability |
| **All** | "Explain the Node event loop", "how would you scale this endpoint", "SQL vs NoSQL for X" |

---

**Start here → [01 — What Is a Backend?](01-what-is-backend.md).** Build the same `habex-api` project all the way through to a deployed capstone in [chapter 22](22-capstone-habex-backend.md).
