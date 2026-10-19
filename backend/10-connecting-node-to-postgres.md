# 10. Connecting Node to PostgreSQL

> **One-liner:** Your Node app talks to Postgres over a network connection — first with the raw **`pg`** driver (where you learn parameterized queries that stop SQL injection), then with **Prisma**, a type-safe ORM that writes the SQL for you and hands back typed objects.
> **Where this fits:** You can already write SQL by hand ([08-sql-and-postgresql-fundamentals.md](08-sql-and-postgresql-fundamentals.md)) and design a schema with migrations ([09-data-modeling-and-migrations.md](09-data-modeling-and-migrations.md)). But so far Habex's data has lived in an in-memory array inside Express ([04-http-servers-and-express-basics.md](04-http-servers-and-express-basics.md)) that vanishes on restart. This chapter is the wire between your Node process and a real database — the moment Habex's data becomes *permanent*. Next chapter ([11-transactions-and-query-performance.md](11-transactions-and-query-performance.md)) makes those reads and writes correct-under-concurrency and fast.

---

## 1. The Idea in Plain Words

PostgreSQL is a **separate server program** — its own process, often on its own machine, listening on TCP **port 5432**. Your Node app is a **client** of that server, exactly like your React app is a client of your API. To use the database you open a **connection** to it, send some SQL text, and read the rows it sends back.

Three terms, defined once:

- **Driver** — the library that actually speaks Postgres's network protocol. It knows how to open the socket, log in, serialize your SQL, and parse the binary rows back into JavaScript values. For Node + Postgres the standard driver is **`pg`** (a.k.a. *node-postgres*). Everything else is built on top of it.
- **Connection string** — one URL that encodes *where* the database is and *who* you are: `postgresql://user:password@host:5432/dbname`. It lives in an environment variable called `DATABASE_URL` (never hardcode it — see [03-project-setup-typescript-node.md](03-project-setup-typescript-node.md)).
- **ORM (Object-Relational Mapper)** — a library that lets you work with **objects and methods** (`prisma.transaction.findMany(...)`) instead of writing SQL strings by hand. It *maps* rows in tables to objects in your language. **Prisma** is our ORM. Under the hood it still uses a driver and still sends SQL — an ORM is a productivity layer *over* the driver, not a replacement for understanding SQL.

The whole journey of one query:

```
your code  →  driver/ORM  →  connection (TCP :5432)  →  Postgres runs SQL  →  rows back  →  JS objects
```

We'll build this twice. First with raw `pg`, so you can *see* the SQL and learn the single most important security habit in backend (parameterization). Then with Prisma, which is what you'll actually use day to day.

---

## 2. Why It Matters

| Reason | What it means for you |
|---|---|
| **Data becomes real** | In-memory arrays die on every deploy and can't be shared across server instances. A database is the memory that survives restarts and scales past one box. |
| **SQL injection is the classic backend vuln** | It has topped the OWASP list for years. Learning *why* parameterized queries are safe — and being able to explain it — is guaranteed interview value, especially at fintech (CRED, Razorpay). |
| **ORMs are the day-job reality** | Product companies mostly use an ORM (Prisma, TypeORM, Sequelize) or a query builder. You need to be fluent in one. |
| **But raw SQL underneath separates seniors from juniors** | Being able to say "here's the SQL Prisma generates, and here's why it's an N+1" is exactly the depth interviewers probe for. |

You'll leave this chapter having wired the real `habex-api` project to a Postgres database with a clean data-access layer you keep using through the capstone.

---

## 3. Picture It (Diagram)

Your Node process doesn't open a fresh connection per query — that's slow (a TCP handshake + auth is tens of milliseconds) and Postgres caps how many connections exist. Instead it keeps a small **connection pool**: a handful of already-open connections it lends out and reclaims. Both `pg` and Prisma manage this pool for you.

```mermaid
flowchart LR
    subgraph App["Node.js process — habex-api"]
        direction TB
        RT["route handler"] --> RP["transactionsRepository"]
        RP --> CL["Prisma client / pg"]
        CL --> POOL["Connection Pool<br/>(e.g. 10 open connections)"]
    end
    POOL -->|"TCP :5432<br/>SQL text + params (separately!)"| PG[("PostgreSQL<br/>server")]
    PG -->|"result rows"| POOL
```

Hold onto that phrase **"SQL text + params separately"** — it's literally the mechanism that defeats SQL injection, and we unpack it next. Pooling gets its own deep dive in [11-transactions-and-query-performance.md](11-transactions-and-query-performance.md).

---

## 4. Hands-On (TypeScript + Node)

### 4a. The raw `pg` driver

```bash
npm install pg
npm install -D @types/pg
```

Put your connection string in `.env` (and `.env` in `.gitignore`):

```bash
# .env
DATABASE_URL="postgresql://habex:secret@localhost:5432/habex"
```

Create a pool once and reuse it everywhere:

```ts
// src/db/pool.ts
import { Pool } from "pg";

// One pool for the whole process. It opens connections lazily and reuses them.
export const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 10, // at most 10 concurrent connections (pool size — see chapter 11)
});

// pool.query() grabs a free connection, runs the SQL, and returns it to the pool.
```

### 4b. Parameterized queries — and WHY they stop SQL injection

This is the most important 20 lines in the chapter. **Read the two versions carefully.**

```ts
// src/db/unsafe-demo.ts  ⚠️ teaching example
import { pool } from "./pool";

// ❌ NEVER build SQL by concatenating user input into the string.
async function findUserUNSAFE(email: string) {
  // If email is normal, fine. But an attacker sends:   ' OR '1'='1
  const sql = `SELECT * FROM users WHERE email = '${email}'`;
  // The text Postgres receives becomes:
  //   SELECT * FROM users WHERE email = '' OR '1'='1'
  // '1'='1' is always true → this returns EVERY user in the table.
  // Worse input:   '; DROP TABLE users; --   → they can delete your data.
  return pool.query(sql);
}

// ✅ ALWAYS pass user input as parameters ($1, $2, ...) in a separate array.
async function findUserSAFE(email: string) {
  const sql = "SELECT * FROM users WHERE email = $1";
  const result = await pool.query(sql, [email]); // values travel separately
  return result.rows[0]; // rows is an array of plain JS objects
}
```

**Why is the parameterized version safe?** Because the placeholder `$1` is **never substituted into the SQL string on your side**. The `pg` driver sends the SQL *template* and the *values* to Postgres as **two separate things**. Postgres parses the SQL structure first (with `$1` as a hole), then plugs the value in as **pure data** that can never be re-interpreted as SQL commands. So `' OR '1'='1` is just... a bizarre email string that matches no rows. The attacker's quotes and semicolons are inert.

Rule to internalize and to *say out loud in interviews*: **user input is data, never code. Parameterize every query. Never concatenate.** This is true for raw `pg`, and Prisma does it for you automatically.

```ts
// Reading rows: `result.rows` is a plain array of objects, `result.rowCount` the count.
const result = await pool.query("SELECT id, email, name FROM users WHERE id = $1", [42]);
console.log(result.rows); // [{ id: 42, email: "...", name: "..." }]
```

### 4c. Prisma — the productive path

```bash
npm install -D prisma          # the CLI (migrations, codegen)
npm install @prisma/client     # the runtime client you import
npx prisma init --datasource-provider postgresql
```

That last command creates `prisma/schema.prisma`. Model the Habex tables there. Note two things: money is `Int` (**integer cents, always** — never a float), and `@map`/`@@map` keep our snake_case SQL column names while giving us camelCase in TypeScript.

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

enum Kind {
  income
  expense
}

model User {
  id           Int           @id @default(autoincrement())
  email        String        @unique
  passwordHash String        @map("password_hash")
  name         String
  createdAt    DateTime      @default(now()) @map("created_at")
  categories   Category[]
  transactions Transaction[]

  @@map("users")
}

model Category {
  id           Int           @id @default(autoincrement())
  userId       Int           @map("user_id")
  user         User          @relation(fields: [userId], references: [id], onDelete: Cascade)
  name         String
  kind         Kind
  color        String
  transactions Transaction[]

  @@map("categories")
}

model Transaction {
  id          Int      @id @default(autoincrement())
  userId      Int      @map("user_id")
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  categoryId  Int      @map("category_id")
  category    Category @relation(fields: [categoryId], references: [id])
  amountCents Int      @map("amount_cents") // integer cents — never a float
  kind        Kind
  note        String?
  occurredAt  DateTime @map("occurred_at")
  createdAt   DateTime @default(now()) @map("created_at")

  // Composite index for the #1 Habex query: "this user's txns in a date range".
  @@index([userId, occurredAt])
  @@map("transactions")
}
```

Turn that schema into real tables with a **migration** (a versioned SQL change file — see [09-data-modeling-and-migrations.md](09-data-modeling-and-migrations.md)) and generate the typed client:

```bash
npx prisma migrate dev --name init   # creates the SQL migration AND applies it
# (migrate dev also runs `prisma generate` for you, producing the typed client)
```

Create **one** Prisma client for the whole process (a singleton). The `globalThis` trick stops dev hot-reload from spawning a new client — and a new pool — on every file save:

```ts
// src/db/prisma.ts
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as { prisma?: PrismaClient };

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({ log: ["query", "warn", "error"] }); // logs the SQL it runs

if (process.env.NODE_ENV !== "production") {
  globalForPrisma.prisma = prisma;
}
```

Now the fun part — **type-safe CRUD**. Autocomplete knows every column; a typo is a compile error:

```ts
import { prisma } from "./db/prisma";

// CREATE
const txn = await prisma.transaction.create({
  data: { userId: 42, categoryId: 3, amountCents: 25000, // ₹250.00 as cents
          kind: "expense", note: "Groceries", occurredAt: new Date("2026-07-03") },
});

// READ many — filtering, sorting, a page size, and a related row pulled in
const page = await prisma.transaction.findMany({
  where: { userId: 42, kind: "expense" }, // WHERE user_id = 42 AND kind = 'expense'
  orderBy: { occurredAt: "desc" },         // ORDER BY occurred_at DESC
  take: 20,                                // LIMIT 20
  include: { category: true },             // JOIN/batch the category in
});

// UPDATE and DELETE — target one row by its unique id
await prisma.transaction.update({ where: { id: txn.id }, data: { note: "Groceries (fixed)" } });
await prisma.transaction.delete({ where: { id: txn.id } });
```

`include: { category: true }` is Prisma's answer to a join — it pulls each transaction's related category along with it, so `page[0].category.name` is right there, fully typed. (How it does that, and the N+1 trap it helps you avoid, is a headline topic of [11-transactions-and-query-performance.md](11-transactions-and-query-performance.md).)

### 4d. The repository pattern — one place for data access

Instead of scattering `prisma.transaction.*` calls across your route handlers, funnel them through a **repository**: a module whose only job is to read and write one kind of thing. Your routes call `transactionsRepository.create(...)` and never see Prisma directly. That keeps business logic testable and means "swap Prisma for raw SQL later" touches one file. (Full treatment in [21-architecture-and-project-structure.md](21-architecture-and-project-structure.md).)

```ts
// src/repositories/transactionsRepository.ts
import { prisma } from "../db/prisma";

export interface CreateTxnInput {
  userId: number;
  categoryId: number;
  amountCents: number; // integer cents
  kind: "income" | "expense";
  note?: string;
  occurredAt: Date;
}

export const transactionsRepository = {
  create(input: CreateTxnInput) {
    return prisma.transaction.create({ data: input });
  },

  /**
   * One month of a user's transactions, newest first, using CURSOR pagination.
   * Cursor pagination = "give me the 20 rows after id X" — it stays fast on page
   * 10,000 because the DB seeks straight to the cursor instead of counting past
   * all the skipped rows the way OFFSET does. Pass the last id you saw as `cursor`.
   */
  getByUserAndMonth(params: {
    userId: number;
    month: string;        // 'YYYY-MM'
    limit?: number;
    cursor?: number;      // id of the last row from the previous page
  }) {
    const { userId, month, limit = 20, cursor } = params;

    // Turn 'YYYY-MM' into a half-open date range [start, nextMonth).
    const start = new Date(`${month}-01T00:00:00.000Z`);
    const end = new Date(start);
    end.setUTCMonth(end.getUTCMonth() + 1);

    return prisma.transaction.findMany({
      where: { userId, occurredAt: { gte: start, lt: end } },
      include: { category: true },
      orderBy: { id: "desc" },
      take: limit,
      // On page 2+, start just AFTER the cursor row.
      ...(cursor ? { cursor: { id: cursor }, skip: 1 } : {}),
    });
  },
};
```

**The exact same `getByUserAndMonth`, written as raw `pg`** — so you can see what Prisma generates under the hood, parameters and all:

```ts
// src/repositories/transactionsRepository.raw.ts
import { pool } from "../db/pool";

export async function getByUserAndMonthRaw(
  userId: number,
  month: string, // 'YYYY-MM'
  limit = 20,
) {
  const sql = `
    SELECT t.*, c.name AS category_name, c.color AS category_color
    FROM transactions t
    JOIN categories c ON c.id = t.category_id
    WHERE t.user_id = $1
      AND t.occurred_at >= $2::date
      AND t.occurred_at <  ($2::date + INTERVAL '1 month')
    ORDER BY t.id DESC
    LIMIT $3;
  `;
  // $1, $2, $3 are parameters — user input never touches the SQL text.
  const result = await pool.query(sql, [userId, `${month}-01`, limit]);
  return result.rows;
}
```

Two forms, same result. Prisma saved you the string, gave you types, and *still* parameterized everything for you.

---

## 5. Apply It to Habex

Back in [chapter 04](04-http-servers-and-express-basics.md) your transactions endpoint looked like this — data in a plain array:

```ts
// BEFORE (chapter 04): in-memory, dies on restart, not per-user, not shared
let transactions: any[] = [];
app.get("/transactions", (_req, res) => res.json(transactions));
app.post("/transactions", (req, res) => {
  transactions.push({ id: transactions.length + 1, ...req.body });
  res.status(201).json(transactions.at(-1));
});
```

Now swap it for **real Prisma-backed persistence** through the repository. Notice how the route stays thin — validation (Zod, [07-input-validation-and-error-handling.md](07-input-validation-and-error-handling.md)) and the `userId` from auth ([12-authentication.md](12-authentication.md)) plug in cleanly:

```ts
// AFTER: src/routes/transactions.ts
import { Router } from "express";
import { z } from "zod";
import { transactionsRepository } from "../repositories/transactionsRepository";

const router = Router();

const createSchema = z.object({
  categoryId: z.number().int().positive(),
  amountCents: z.number().int(), // integer cents, enforced at the door
  kind: z.enum(["income", "expense"]),
  note: z.string().max(280).optional(),
  occurredAt: z.coerce.date(),
});

// GET /transactions?month=2026-07&cursor=123
router.get("/", async (req, res, next) => {
  try {
    const userId = req.user!.id; // set by auth middleware (chapter 12)
    const month = String(req.query.month ?? new Date().toISOString().slice(0, 7));
    const cursor = req.query.cursor ? Number(req.query.cursor) : undefined;

    const items = await transactionsRepository.getByUserAndMonth({ userId, month, cursor });
    const nextCursor = items.length ? items[items.length - 1].id : null;

    res.json({ items, nextCursor }); // client sends nextCursor back for page 2
  } catch (err) {
    next(err); // hand off to the central error handler (chapter 07)
  }
});

// POST /transactions
router.post("/", async (req, res, next) => {
  try {
    const userId = req.user!.id;
    const body = createSchema.parse(req.body); // throws on bad input
    const created = await transactionsRepository.create({ userId, ...body });
    res.status(201).json(created);
  } catch (err) {
    next(err);
  }
});

export default router;
```

That is the shape of essentially every Habex endpoint from here on: **route → validate → repository → Prisma → Postgres.** Restart the server and your data is still there. Run two server instances and they share one source of truth. That is the whole point of a database.

---

## 6. Interview Questions (with answer pointers)

**Q1. "ORM vs raw SQL — when would you use each?" (asked everywhere)**
Pointer: ORM (Prisma) for the 90% of CRUD — type safety, less boilerplate, migrations, portability, automatic parameterization. Drop to raw SQL for complex reporting queries, database-specific features (window functions, `INSERT ... ON CONFLICT`, CTEs), or when you need to hand-tune a hot query the ORM generates poorly. Senior line: *"ORM by default, raw SQL where it earns its keep — and I can read the SQL my ORM emits."*

**Q2. "How do you prevent SQL injection?" (CRED, Razorpay — security favorites)**
Pointer: **Parameterized queries** (`$1`, `$2`), never string concatenation. Explain the *mechanism*: SQL text and values travel to Postgres separately; the value is bound as pure data and can never be parsed as SQL. Bonus: input validation (Zod) as defense-in-depth, and least-privilege DB users. ORMs parameterize automatically — but you can still shoot yourself with their raw-query escape hatch (`$queryRawUnsafe`), so know which methods are safe.

**Q3. "What's an N+1 query problem?" (teaser — full answer in ch11)**
Pointer: You fetch a list (1 query), then loop and fetch each row's relation one-by-one (N queries) → N+1 round-trips. ORMs make it easy to cause accidentally. Fix by loading relations up front (`include` / a join) so it's 1–2 queries total. Deep dive: [11-transactions-and-query-performance.md](11-transactions-and-query-performance.md).

**Q4. "How does your app manage database connections?" (Zepto, Swiggy — scale questions)**
Pointer: A **connection pool** — a fixed set of pre-opened connections reused across requests, because opening one per query is slow and Postgres caps total connections. Both `pg` and Prisma pool by default. Mention serverless makes this hard (each instance gets its own pool → connection storms → use PgBouncer). Detail in ch11.

**Q5. "SQL or NoSQL for Habex, and why does that inform this choice?"**
Pointer: Relational (Postgres) — related, transactional, report-heavy financial data (see [../system-design/09-sql-vs-nosql.md](../system-design/09-sql-vs-nosql.md)). That's *why* we invest in an SQL driver + a relational ORM rather than a document store.

---

## 7. Trade-offs & Pitfalls ("Why Not Just X?")

### ORM vs raw SQL vs query builder

| | Raw driver (`pg`) | Query builder (Knex) | ORM (Prisma) |
|---|---|---|---|
| You write | SQL strings | chained JS that builds SQL | model methods; SQL is generated |
| Type safety | none (rows are `any`) | partial | strong, generated from schema |
| Parameterization | manual (`$1`) but explicit | automatic | automatic |
| Complex/reporting SQL | full power | good | escape to raw when needed |
| Migrations | you wire them up | built-in | first-class (`prisma migrate`) |
| Risk | injection if you concat | leaky for exotic SQL | can hide N+1 / over-fetch |

**"Why not just use raw SQL everywhere — it's the most powerful?"** Power isn't the bottleneck; *human error and velocity* are. Hand-written SQL means every developer must remember to parameterize, keep types in sync by hand, and hand-roll migrations. On a team, an ORM removes whole categories of mistakes (injection, type drift) and speeds up the boring 90%. Keep raw SQL as a scalpel for the 10% that needs it.

**"Why not just use an ORM for everything, then?"** Because ORMs hide the SQL, and hidden SQL bites: the **N+1 problem** (ch11), accidental **over-fetching** (`SELECT *` when you needed two columns), and awkward generated SQL for analytical queries. The senior move is to keep Prisma's query logging on in dev and actually *read* the SQL — an ORM should be a convenience you can see through, not a black box.

**"Why not skip pooling and connect per request?"** Because opening a connection is a full TCP handshake plus authentication (tens of ms), and Postgres has a hard `max_connections` (default ~100). Under load you'd exhaust connections and pay the handshake cost on every request. Pool once, reuse. More in [11-transactions-and-query-performance.md](11-transactions-and-query-performance.md).

**Pitfalls to sidestep:**
- **Floats for money.** `amount_cents` is `Int`. `0.1 + 0.2 !== 0.3` will corrupt a ledger. This is a fintech red line.
- **`$queryRawUnsafe` / string-built raw SQL.** Prisma's `$queryRaw` tagged template is safe (parameterized); the `...Unsafe` variants are not. Prefer the tagged-template form.
- **A new `PrismaClient` per request.** Each one spins up its own pool → connection exhaustion. Use the singleton.
- **`OFFSET` pagination on huge tables.** `OFFSET 100000` still scans and discards 100k rows — prefer the cursor pagination shown above.
- **Leaking the connection string.** `DATABASE_URL` in a git-ignored `.env`; the real secret lives in the host's env vars.

---

## 8. TL;DR

- Postgres is a **separate server** on TCP **:5432**; your Node app is a **client**. A **driver** (`pg`) speaks its protocol; an **ORM** (**Prisma**) is a typed convenience layer on top that generates the SQL.
- **Parameterize every query** (`$1`, `$2`). SQL text and values travel *separately*, so user input is bound as pure data and can never execute as SQL. **Never concatenate user input into SQL** — that's SQL injection, the classic backend vuln.
- **Prisma flow:** model tables in `schema.prisma` (money = `Int` cents), `prisma migrate dev` to create tables, import a **singleton** client, and call type-safe `create` / `findMany` (with `where` / `orderBy` / `take` / `cursor`) / `update` / `delete`. `include` pulls in relations.
- Hide data access behind a **repository** so routes stay thin and Prisma is swappable (→ [21-architecture-and-project-structure.md](21-architecture-and-project-structure.md)).
- **Choose the tool per query:** ORM for the 90%, raw SQL (still parameterized!) for the reporting 10%. Keep query logging on and *read* the SQL your ORM emits.
- **Habex:** the in-memory array from ch04 is now a real `transactions` table behind `transactionsRepository`. Every endpoint becomes **route → validate → repository → Prisma → Postgres**.

Next: making those writes atomic and those reads fast → [11-transactions-and-query-performance.md](11-transactions-and-query-performance.md). Foundations: [08-sql-and-postgresql-fundamentals.md](08-sql-and-postgresql-fundamentals.md), [09-data-modeling-and-migrations.md](09-data-modeling-and-migrations.md), and the *why* behind picking Postgres at all: [../system-design/09-sql-vs-nosql.md](../system-design/09-sql-vs-nosql.md).
