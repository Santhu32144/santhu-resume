# System Design — From Zero to Interview-Ready

> A chapter-by-chapter curriculum built for **Santhosh** — a frontend-heavy full-stack dev (React/TypeScript) with strong UI instincts but light backend/system-design exposure, prepping for product-company interviews (**Zepto, Swiggy, CRED, Razorpay, Groww**).

You don't need any backend knowledge to start. Chapter 1 assumes zero. By the last chapter you'll be running a full 45-minute system-design interview end to end.

---

## How to use this

1. **Read in order the first time.** Each chapter builds on the previous ones. The early chapters deliberately over-explain; that's the point.
2. **Do the interview questions out loud.** Every chapter has a "Interview Questions" section with answer pointers. Cover the answer, try it, then check.
3. **Tie it to your own apps.** Every chapter has a *"Tie-Back to Your Projects"* section connecting the concept to [santhu.site](https://santhu.site) (your portfolio) and [habex.santhu.site](https://habex.santhu.site) (your Habex expense/habit tracker). This is your unfair advantage — you can speak about a real system you built.
4. **Revise from the TL;DRs.** The night before an interview, re-read only the `## TL;DR` sections + `26-interview-playbook.md`.

---

## How each chapter is structured

Every chapter follows the same 8-part template so you always know where to look:

| Section | What it gives you |
|---|---|
| **1. The Idea in Plain Words** | The concept with zero assumed knowledge |
| **2. Picture It (Diagram)** | An ASCII / Mermaid diagram of how the pieces connect |
| **3. Real-World Analogy** | Something everyday that makes it stick |
| **4. Concrete Example** | A real system (Instagram / WhatsApp / Uber / URL shortener) with numbers |
| **5. Tie-Back to Your Projects** | How it shows up in santhu.site / Habex |
| **6. Interview Questions** | 5–8 real questions + answer pointers, tagged by company |
| **7. Trade-offs & "Why Not Just X?"** | The judgement layer — interviews test this, not memorization |
| **8. TL;DR** | A 5–8 bullet cheat-sheet for last-minute revision |

---

## The full curriculum

### Part I — Fundamentals (how the internet actually works)
- [01 — What Is System Design](01-what-is-system-design.md) — the mindset, requirements, and the interview framework
- [02 — Client-Server Basics](02-client-server-basics.md) — frontend, backend, request/response
- [03 — IP & DNS](03-ip-and-dns.md) — how a domain name becomes a real machine
- [04 — HTTP & HTTPS](04-http-and-https.md) — the language the web speaks
- [05 — Load Balancers](05-load-balancers.md) — spreading traffic across many servers

### Part II — Scaling Basics
- [06 — Vertical vs Horizontal Scaling](06-scaling-vertical-vs-horizontal.md) — bigger machine vs more machines
- [07 — Caching](07-caching.md) — the single biggest performance lever
- [08 — CDN](08-cdn.md) — serving content from close to the user

### Part III — Databases
- [09 — SQL vs NoSQL](09-sql-vs-nosql.md) — choosing your data store
- [10 — Database Indexing](10-database-indexing.md) — why queries are fast (or slow)
- [11 — Replication](11-replication.md) — copies for reads and survival
- [12 — Sharding & Partitioning](12-sharding-and-partitioning.md) — splitting data across machines

### Part IV — Consistency, Availability & Messaging
- [13 — CAP Theorem](13-cap-theorem.md) — the fundamental distributed-systems trade-off
- [14 — ACID vs BASE & Consistency Models](14-acid-vs-base-and-consistency.md) — how "correct" you need to be
- [15 — Message Queues & Pub-Sub](15-message-queues-and-pubsub.md) — doing work asynchronously
- [16 — Kafka & Webhooks](16-kafka-and-webhooks.md) — event streaming and callbacks (very Razorpay-relevant)

### Part V — API Design
- [17 — API Design: REST, GraphQL, gRPC](17-api-design-rest-graphql.md)
- [18 — Rate Limiting, Pagination & Versioning](18-rate-limiting-pagination-versioning.md)

### Part VI — Reliability, Security & Architecture
- [19 — Reliability Patterns](19-reliability-patterns.md) — failover, retries, circuit breakers, idempotency
- [20 — Security Basics](20-security-basics.md) — auth, OAuth, JWT, encryption
- [21 — Architecture Patterns](21-architecture-patterns.md) — monolith vs microservices, event-driven, CQRS

### Part VII — End-to-End Case Studies
- [22 — Design a URL Shortener](22-case-study-url-shortener.md) — the classic warm-up
- [23 — Design Instagram](23-case-study-instagram.md) — feeds, fan-out, media at scale
- [24 — Design a Chat App (WhatsApp)](24-case-study-chat-app.md) — realtime, delivery receipts
- [25 — Design an Expense Tracker (your Habex)](25-case-study-expense-tracker.md) — the capstone, on your own project
- [26 — Interview Playbook](26-interview-playbook.md) — the framework, per-company focus, cheat sheets

---

## Suggested pace

| Week | Chapters | Goal |
|---|---|---|
| 1 | 01–05 | Speak confidently about how a request travels the internet |
| 2 | 06–08 | Explain how systems handle 100× more traffic |
| 3 | 09–12 | Choose and scale a database out loud |
| 4 | 13–16 | Reason about consistency and async processing |
| 5 | 17–21 | Design clean APIs and reliable, secure services |
| 6 | 22–26 | Run full mock interviews; drill the playbook |

---

## A note on mindset

System-design interviews are **not** about memorizing "the right answer." There is no single right answer. Interviewers at product companies want to see:

1. You **clarify requirements** before designing.
2. You **estimate** roughly (back-of-envelope) instead of guessing blindly.
3. You **make trade-offs explicitly** and can defend them.
4. You **know when something is overkill** (a URL shortener doesn't need microservices).

Every chapter's *"Trade-offs & Why Not Just X?"* section is training for exactly that. Start with [01 — What Is System Design](01-what-is-system-design.md).
