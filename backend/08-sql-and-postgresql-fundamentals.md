# 08. SQL & PostgreSQL Fundamentals

> **One-liner:** A relational database stores your app's data in strict, related **tables**, and **SQL** is the English-like language you use to create those tables and ask them questions — this chapter gets PostgreSQL running on your machine and teaches you every SQL verb you'll use to power Habex.
> **Where this fits:** Chapters 04–07 built an Express API that validates input and returns JSON — but every response so far is faked or held in memory, so it vanishes on restart. This chapter gives your server a **memory that survives**. You learned the *why* (SQL vs NoSQL, ACID) at the concept level in [../system-design/09-sql-vs-nosql.md](../system-design/09-sql-vs-nosql.md); here you learn the *how* by hand, in raw SQL, before we put an ORM (Prisma) in front of it from chapter 10. It leads straight into schema design & migrations ([09-data-modeling-and-migrations.md](09-data-modeling-and-migrations.md)), wiring Node to Postgres ([10-connecting-node-to-postgres.md](10-connecting-node-to-postgres.md)), and transactions & performance ([11-transactions-and-query-performance.md](11-transactions-and-query-performance.md)).

---

## 1. The Idea in Plain Words

A **database** is a program whose entire job is to store data safely and hand it back fast. Your React app forgets everything on refresh; a database is the memory that outlives the request, the deploy, and the power cut.

The most common kind — and the one every product company runs — is a **relational database**. It stores data in **tables**. If you have used a spreadsheet or written an HTML `<table>`, you already understand most of it. Let me define every word, because these exact terms come up in interviews:

- **Table** — a grid that holds one *kind* of thing. One table for users, one for transactions. (Also called a *relation* — that's where "relational" comes from.)
- **Row** — one single record in a table. One user. One expense. (Also called a *record* or *tuple*.)
- **Column** — one named field every row has, with a fixed **data type**. `email` (text), `amount_cents` (integer), `created_at` (timestamp). (Also called an *attribute*.)
- **Schema** — the *shape* of your tables: which columns exist, their types, and their rules. It's decided up front, and every row must obey it. Try to put the word `"lunch"` into an `INTEGER` column and the database rejects it. This is called **schema-on-write**, and it's a feature: bad data is stopped at the door.
- **Primary key (PK)** — a column whose value *uniquely* identifies a row, usually an auto-generated `id`. No two rows share it. It's how you point at exactly one row.
- **Foreign key (FK)** — a column that holds the primary key of a row in *another* table. It's the "link". A transaction's `user_id` is a foreign key into `users`; it's how the database knows *this expense belongs to that user*, and it refuses to let you point at a user who doesn't exist (that guarantee is called **referential integrity**).
- **Index** — a small, sorted lookup structure the database keeps on the side so it can jump to the rows you asked for instead of scanning the whole table. You'll add your first ones in chapter 9; the deep dive is [../system-design/10-database-indexing.md](../system-design/10-database-indexing.md).
- **Constraint** — a rule the database enforces on every write: `NOT NULL`, `UNIQUE`, `CHECK (amount_cents > 0)`. Constraints are correctness you don't have to remember to write in application code.

**PostgreSQL** ("Postgres") is a specific, free, open-source relational database — the default sane choice at product companies in 2026. It's rock-solid, strictly correct, and packed with features (rich types, JSON support, powerful queries). Others exist (MySQL, SQLite, SQL Server), but everything you learn here transfers.

You talk to Postgres in **SQL** (Structured Query Language) — a text language that reads almost like English:

```sql
SELECT note, amount_cents
FROM transactions
WHERE user_id = 1
ORDER BY occurred_at DESC
LIMIT 10;
```

SQL comes in two flavours worth naming: **DDL** (Data *Definition* Language — `CREATE TABLE`, `ALTER TABLE`: defines shape) and **DML** (Data *Manipulation* Language — `SELECT`, `INSERT`, `UPDATE`, `DELETE`: works with rows). That's the whole vocabulary; the rest of this chapter is you using it.

---

## 2. Why It Matters

- **Persistence is the point of a backend.** Without a database your API is a stateless echo. The database is where Habex's users, expenses, and budgets actually live between requests.
- **SQL is a guaranteed interview round.** Product companies (Zepto, Swiggy, CRED, Razorpay, Groww) hand you a schema and say "write a query for total spend per category this month." That's a `JOIN` + `GROUP BY` — exactly what you'll write below.
- **Correctness is a fintech hiring signal.** The single most common "gotcha" for a money app is **storing money as a float**. Getting it right (integer cents) is a small thing that says "this person has shipped real financial code."
- **It's the foundation the next four chapters stand on.** Modeling, migrations, the `pg` driver, Prisma, transactions, indexing — all of it assumes you can read and write SQL fluently. Learn it raw once and the ORM becomes a convenience instead of a crutch.

---

## 3. Picture It (Diagram)

Here's the slice of Habex you'll build in this chapter — three tables, linked by foreign keys. Read the crow's-foot `||--o{` as "one user has **many** transactions."

```mermaid
erDiagram
    USERS ||--o{ CATEGORIES : "owns"
    USERS ||--o{ TRANSACTIONS : "records"
    CATEGORIES ||--o{ TRANSACTIONS : "labels"

    USERS {
        bigint id PK
        text email UK
        text password_hash
        text name
        timestamptz created_at
    }
    CATEGORIES {
        bigint id PK
        bigint user_id FK
        text name
        text kind
        text color
    }
    TRANSACTIONS {
        bigint id PK
        bigint user_id FK
        bigint category_id FK
        integer amount_cents
        text kind
        text note
        timestamptz occurred_at
        timestamptz created_at
    }
```

The data is **split into clean tables** and reconnected with **joins** at read time. `PK` = primary key, `FK` = foreign key, `UK` = unique. Notice a transaction carries *two* foreign keys: it belongs to a user **and** to a category.

---

## 4. Hands-On (real SQL + running Postgres)

### 4.1 Get Postgres running in one line (Docker)

The easiest way to run Postgres locally is Docker — no system-wide install, and you can throw it away and start clean anytime. (Full Docker treatment is chapter 19; this one line is all you need now.)

```bash
docker run --name habex-pg \
  -e POSTGRES_USER=habex \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=habex \
  -p 5432:5432 \
  -d postgres:16
```

That starts Postgres 16 in the background, creates a database called `habex`, and exposes it on the standard port `5432`. Check it's up with `docker ps`. Stop/start it later with `docker stop habex-pg` / `docker start habex-pg`.

### 4.2 Connect with psql

`psql` is Postgres's built-in interactive terminal. The container already ships with it:

```bash
docker exec -it habex-pg psql -U habex -d habex
```

If you have `psql` installed on your host, you can instead connect with a **connection string** (memorize this format — it becomes your `DATABASE_URL` env var in chapter 10):

```bash
psql "postgresql://habex:secret@localhost:5432/habex"
#            user  :password @host    :port /database
```

Inside psql, **meta-commands** start with a backslash:

| Command | Does |
|---|---|
| `\l` | list databases |
| `\dt` | list tables |
| `\d transactions` | describe a table (columns, types, indexes) |
| `\x` | toggle expanded (row-per-line) display — great for wide rows |
| `\i file.sql` | run a `.sql` file |
| `\q` | quit |

**Prefer a GUI?** Point **TablePlus**, **pgAdmin**, or **DBeaver** at host `localhost`, port `5432`, user `habex`, password `secret`, database `habex`. A GUI is lovely for browsing data and eyeballing your schema; psql is faster for running scripts and is always available on a server. Learn both.

### 4.3 Core data types (and the money rule)

Every column needs a type. These are the ones you'll actually use:

| Type | Use it for | Notes |
|---|---|---|
| `INTEGER` (`INT`) | whole numbers, **money as cents** | 4 bytes, range ±2.1 billion |
| `BIGINT` | IDs, large counts, big money sums | 8 bytes; the safe default for keys |
| `TEXT` | strings of any length | In Postgres, prefer this — no length limit, no perf cost |
| `VARCHAR(n)` | strings with a *real* max | Only when the cap is a genuine business rule |
| `BOOLEAN` | true / false | e.g. a recurring rule's `active` flag |
| `TIMESTAMPTZ` | an instant in time | Timezone-aware; **always use this**, not `TIMESTAMP` |
| `DATE` | a calendar day, no time | e.g. a birthday |
| `SERIAL` / `IDENTITY` | auto-incrementing PK | See below — prefer `IDENTITY` |
| `UUID` | globally-unique IDs | `gen_random_uuid()`; useful when IDs are generated by clients |
| `NUMERIC(p,s)` | exact decimals | Exact, but we *still* use integer cents for money — see below |

**Auto-increment IDs — `SERIAL` vs `IDENTITY`.** You'll see both. `id SERIAL PRIMARY KEY` is the classic, everywhere-in-old-tutorials form. The modern, SQL-standard form Postgres recommends is `id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY`. They both mean "the database fills in 1, 2, 3, … for me." We'll use `IDENTITY` in this track.

**The money rule — store integer cents, never a float.** This is the hill to die on for a finance app:

```js
0.1 + 0.2            // 0.30000000000000004   ← floating point can't represent these exactly
```

`FLOAT` / `REAL` / `DOUBLE PRECISION` are *binary* floating point. They round, and rounding errors in money compound into wrong totals and failed reconciliations. So the rule:

> **Store money as an integer number of the smallest currency unit** — paise for ₹, cents for $. `₹12.50` is stored as `1250` in an `amount_cents INTEGER` column. Integer math is exact and fast, and it matches how payment APIs (Stripe, Razorpay) already send you money. Format it back to rupees in your app/UI layer, never in the database.

```ts
// Node/TS formatting helper — the ONLY place cents become a decimal
const formatINR = (cents: number) =>
  (cents / 100).toLocaleString("en-IN", { style: "currency", currency: "INR" });

formatINR(1250);   // "₹12.50"     (1250 paise)
formatINR(125000); // "₹1,250.00"  (125000 paise)
```

What about `NUMERIC`? `NUMERIC(12,2)` *is* exact (it's decimal, not binary) and is a perfectly valid way to store money. Many banks use it. But our track standard is **integer cents** because integer math is unambiguous, trivially fast to `SUM`, and impossible to accidentally feed a float. Pick one convention and hold it everywhere — we hold integer cents. One caveat to know for interviews: a single `INTEGER` caps at ₹21,474,836.47; if a *row* could exceed that, use `BIGINT`. Sums are safe either way because Postgres widens `SUM(integer)` to `bigint` automatically.

### 4.4 CREATE TABLE — build the Habex tables

Now the real thing. Paste this into psql (or a `.sql` file and `\i` it). Read the comments — every constraint is doing a job.

```sql
CREATE TABLE users (
  id            BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  email         TEXT        NOT NULL UNIQUE,          -- no two users share an email
  password_hash TEXT        NOT NULL,                 -- never store the raw password (ch12)
  name          TEXT        NOT NULL,
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now()    -- DB fills this in
);

CREATE TABLE categories (
  id       BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id  BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,  -- FK
  name     TEXT   NOT NULL,
  kind     TEXT   NOT NULL CHECK (kind IN ('income', 'expense')),   -- only these 2 values
  color    TEXT
);

CREATE TABLE transactions (
  id           BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id      BIGINT  NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  category_id  BIGINT  REFERENCES categories(id) ON DELETE SET NULL, -- nullable = "uncategorized"
  amount_cents INTEGER NOT NULL CHECK (amount_cents > 0),            -- always positive; sign comes from `kind`
  kind         TEXT    NOT NULL CHECK (kind IN ('income', 'expense')),
  note         TEXT,
  occurred_at  TIMESTAMPTZ NOT NULL DEFAULT now(),                   -- when the money moved
  created_at   TIMESTAMPTZ NOT NULL DEFAULT now()                    -- when the row was inserted
);
```

Three modelling decisions worth calling out:

- **`ON DELETE CASCADE`** on `user_id`: delete a user and *their* categories and transactions go too. Sensible — nobody else's data references them.
- **`ON DELETE SET NULL`** on `category_id`, plus making it **nullable**: if a user deletes the "Food" category, their old food transactions aren't destroyed — they just become uncategorized. Deleting a label shouldn't delete history.
- **`amount_cents` is always positive**, and direction (`income` vs `expense`) lives in `kind`. This avoids a whole class of sign bugs. (We go deeper on FK actions and enums-vs-lookup-tables in [09-data-modeling-and-migrations.md](09-data-modeling-and-migrations.md).)

### 4.5 INSERT sample rows

```sql
INSERT INTO users (email, password_hash, name)
VALUES ('santhosh@habex.app', '$2b$dummyhash', 'Santhosh');

-- Categories for user 1
INSERT INTO categories (user_id, name, kind, color) VALUES
  (1, 'Food',      'expense', '#ef4444'),
  (1, 'Rent',      'expense', '#3b82f6'),
  (1, 'Transport', 'expense', '#f59e0b'),
  (1, 'Salary',    'income',  '#10b981');

-- A few July transactions (amounts in paise: 125000 = ₹1,250.00)
INSERT INTO transactions (user_id, category_id, amount_cents, kind, note, occurred_at) VALUES
  (1, 1, 45000,   'expense', 'Groceries',   '2026-07-02'),
  (1, 1, 32000,   'expense', 'Swiggy',      '2026-07-04'),
  (1, 2, 1800000, 'expense', 'July rent',   '2026-07-01'),
  (1, 3, 12000,   'expense', 'Auto',        '2026-07-05'),
  (1, 4, 9000000, 'income',  'July salary', '2026-07-01'),
  (1, NULL, 5000, 'expense', 'Uncategorized tea', '2026-07-06');
```

Note the last row has `category_id = NULL` — a legitimately uncategorized expense. Hold that thought for the LEFT JOIN below.

### 4.6 SELECT — WHERE, ORDER BY, LIMIT

`SELECT` reads rows. Choose columns, filter with `WHERE`, sort with `ORDER BY`, cap with `LIMIT`.

```sql
-- Santhosh's 10 most recent transactions
SELECT id, note, amount_cents, occurred_at
FROM transactions
WHERE user_id = 1
ORDER BY occurred_at DESC
LIMIT 10;
```

```sql
-- All of July's expenses, biggest first.
-- Use a half-open date range (>= start AND < next month) — it's index-friendly and dodges timezone edge cases.
SELECT note, amount_cents
FROM transactions
WHERE user_id = 1
  AND kind = 'expense'
  AND occurred_at >= '2026-07-01'
  AND occurred_at <  '2026-08-01'
ORDER BY amount_cents DESC;
```

A word on `NULL`: it means "unknown", and it infects comparisons. `category_id = NULL` is never true — you must write `category_id IS NULL` / `IS NOT NULL`. This three-valued logic (true/false/unknown) is a classic interview trip-wire.

### 4.7 UPDATE and DELETE (mind the WHERE!)

```sql
-- Fix a note and amount on one row
UPDATE transactions
SET note = 'Groceries — BigBasket', amount_cents = 47500
WHERE id = 1;

-- Remove one transaction
DELETE FROM transactions
WHERE id = 6;
```

> **The most expensive footgun in SQL:** forget the `WHERE` and `UPDATE transactions SET note = 'x'` rewrites *every row in the table*; `DELETE FROM transactions` empties it. Always write the `WHERE` first. In production you protect against this with transactions (`BEGIN` … `COMMIT`/`ROLLBACK`), covered in [11-transactions-and-query-performance.md](11-transactions-and-query-performance.md).

### 4.8 JOINs — stitch tables back together

A **join** follows a foreign key to combine rows from two tables. Picture the two tables and the result:

```text
categories                 transactions (category_id)
 id | name                  id | category_id | amount_cents
 ---+-------                ---+-------------+-------------
  1 | Food                   1 |     1       |  45000
  2 | Rent                   2 |     1       |  32000
  3 | Transport              3 |     2       | 1800000
                             6 |    NULL     |   5000   ← uncategorized
```

**INNER JOIN** keeps only rows that match on both sides — so the `NULL`-category transaction (id 6) is dropped:

```sql
SELECT t.note, t.amount_cents, c.name AS category
FROM transactions t
INNER JOIN categories c ON c.id = t.category_id   -- "INNER" is the default; you can omit it
WHERE t.user_id = 1;
-- → rows 1,2,3 with their category names. Row 6 vanishes (no matching category).
```

**LEFT JOIN** keeps *every* left-side (transactions) row, filling `NULL` where there's no match — so the uncategorized transaction survives:

```sql
SELECT t.note, t.amount_cents, c.name AS category
FROM transactions t
LEFT JOIN categories c ON c.id = t.category_id
WHERE t.user_id = 1;
-- → rows 1,2,3 AND row 6, whose `category` column is NULL.
```

Rule of thumb: **INNER JOIN = "only where both exist"; LEFT JOIN = "everything on the left, matched where possible."** Reach for LEFT JOIN when the right side is optional (like a nullable category, or "users and their orders, including users with zero orders").

### 4.9 Aggregation — COUNT, SUM, GROUP BY, HAVING

This is Habex's home turf: dashboards. **Aggregate functions** collapse many rows into one number. `GROUP BY` runs them *per group*.

```sql
-- Habex dashboard: total spend per category in July.
SELECT c.name AS category,
       SUM(t.amount_cents) AS total_cents,     -- SUM(integer) -> bigint, so no overflow
       COUNT(*)            AS num_txns
FROM transactions t
JOIN categories c ON c.id = t.category_id
WHERE t.user_id = 1
  AND t.kind = 'expense'
  AND t.occurred_at >= '2026-07-01'
  AND t.occurred_at <  '2026-08-01'
GROUP BY c.name
ORDER BY total_cents DESC;
```

```text
 category  | total_cents | num_txns
-----------+-------------+----------
 Rent      |     1800000 |        1
 Food      |       77000 |        2
 Transport |       12000 |        1
```

**`HAVING`** filters *after* grouping (where `WHERE` filters *before*). "Only show categories where I spent more than ₹5,000":

```sql
SELECT c.name, SUM(t.amount_cents) AS total_cents
FROM transactions t
JOIN categories c ON c.id = t.category_id
WHERE t.user_id = 1 AND t.kind = 'expense'
GROUP BY c.name
HAVING SUM(t.amount_cents) > 500000    -- > ₹5,000; can't use WHERE for an aggregate
ORDER BY total_cents DESC;
```

Remember: **`WHERE` filters rows, `HAVING` filters groups.** That distinction is a favourite interview question.

To include categories with **zero** spend (so the dashboard shows "Salary: ₹0" instead of hiding it), flip to a LEFT JOIN from `categories` and default the sum with `COALESCE`:

```sql
SELECT c.name, COALESCE(SUM(t.amount_cents), 0) AS total_cents
FROM categories c
LEFT JOIN transactions t
       ON t.category_id = c.id AND t.kind = 'expense'
WHERE c.user_id = 1
GROUP BY c.name
ORDER BY total_cents DESC;
```

`COALESCE(x, 0)` returns the first non-NULL argument — here it turns "no matching transactions → NULL sum" into a clean `0`.

---

## 5. Apply It to Habex

Everything above *is* Habex — three real tables and the three queries the app's UI needs:

1. **Transaction list screen** → the `SELECT … WHERE user_id = 1 ORDER BY occurred_at DESC LIMIT 20` from 4.6. This becomes `GET /transactions?month=2026-07`.
2. **Spend-by-category donut chart** → the `GROUP BY` in 4.9. This becomes `GET /summary?month=2026-07`, and it's the single most satisfying endpoint to build because the database does all the math.
3. **Transactions with their category labels** → the JOIN in 4.8, so the API can return `{ note, amount, category: "Food" }` instead of a bare `category_id`.

You now have the schema and the queries. In [10-connecting-node-to-postgres.md](10-connecting-node-to-postgres.md) you'll run these exact strings from Express via the `pg` driver (and later Prisma); in [09-data-modeling-and-migrations.md](09-data-modeling-and-migrations.md) you'll turn these `CREATE TABLE`s into versioned migration files and add `budgets` and `recurring_rules`.

**A note for the React dev in you:** it's tempting to store data in the nested shape your components want (`user.transactions[0].category`). Resist. Store it *normalized* in these flat, related tables — the database's job is correctness and flexible querying — and let your API *assemble* the nested JSON at read time. Keeping those two jobs separate is a senior instinct (more in [../system-design/09-sql-vs-nosql.md](../system-design/09-sql-vs-nosql.md)).

---

## 6. Interview Questions (with answer pointers)

**Q1. INNER JOIN vs LEFT JOIN? (asked everywhere)**
Pointer: INNER keeps only rows with a match on both sides; LEFT keeps *every* left-side row and fills NULL where the right has no match. Use LEFT when the right side is optional — "all categories including those with zero transactions." Bonus: mention that a `WHERE right.col = x` after a LEFT JOIN silently turns it back into an inner join (put such conditions in the `ON` clause instead).

**Q2. `WHERE` vs `HAVING`? / What does `GROUP BY` do?**
Pointer: `GROUP BY` collapses rows into groups so aggregate functions (`SUM`, `COUNT`, `AVG`) run per group. `WHERE` filters individual rows *before* grouping; `HAVING` filters *groups* after aggregation. You can't use an aggregate in `WHERE`.

**Q3. Why store money as integer cents, not a float? (CRED, Razorpay, Groww love this)**
Pointer: Binary floats can't represent decimal money exactly (`0.1 + 0.2 = 0.30000000000000004`), so errors compound and totals stop reconciling. Store the smallest unit as an `INTEGER`/`BIGINT` (paise/cents); it's exact, fast to `SUM`, and matches payment-gateway APIs. Format to a decimal only in the presentation layer. Mention `NUMERIC` as an exact alternative but say you standardize on integer cents.

**Q4. Primary key vs foreign key — what does each guarantee?**
Pointer: A PK uniquely identifies a row (unique + not null) and is how you address exactly one record. An FK holds another table's PK and enforces **referential integrity** — you can't insert a transaction pointing at a non-existent user, and `ON DELETE` rules decide what happens to children when a parent is removed (CASCADE / SET NULL / RESTRICT).

**Q5. What's an index and when would you add one? (teaser)**
Pointer: An index is a sorted side-structure that lets the DB jump to matching rows instead of scanning the whole table — turning O(n) scans into ~O(log n) lookups, at the cost of extra storage and slightly slower writes. Add one on columns you filter/join/sort by a lot — for Habex, `(user_id, occurred_at)`. Full story: [../system-design/10-database-indexing.md](../system-design/10-database-indexing.md), and you'll add real indexes in [09-data-modeling-and-migrations.md](09-data-modeling-and-migrations.md).

**Q6. `UNIQUE` vs `PRIMARY KEY`?**
Pointer: Both enforce uniqueness. A table has exactly *one* primary key (which is also `NOT NULL` and the row's identity); it can have *many* `UNIQUE` constraints, and a `UNIQUE` column may allow a NULL. `users.email` is a natural `UNIQUE`; `users.id` is the PK.

**Q7. Write a query: top 3 spending categories this month.**
Pointer: `JOIN` transactions→categories, filter by user + `kind='expense'` + a half-open date range, `GROUP BY c.name`, `ORDER BY SUM(amount_cents) DESC`, `LIMIT 3`. This one question exercises joins, filtering, aggregation, sorting, and limiting at once — exactly the query from section 4.9 with a `LIMIT 3`.

---

## 7. Trade-offs & Pitfalls ("Why Not Just X?")

- **"Why not just use a `FLOAT` for money — it's simpler?"** Because it's *wrong*: floats round, and a rounding error on money is a bug you can't ship at a fintech. Integer cents cost you nothing and remove the entire failure mode.
- **"Why not `SELECT *` everywhere?"** Fine at the psql prompt while exploring; bad in app code. It fetches columns you don't need (including large ones you add later), and it makes your code silently depend on column order. Name the columns you want.
- **"Why not `VARCHAR(255)` like the old tutorials?"** In Postgres, `TEXT` and `VARCHAR` perform identically; a length cap only helps when it's a real rule. Default to `TEXT`; add a `CHECK (char_length(x) <= n)` if you genuinely need a limit.
- **`TIMESTAMP` vs `TIMESTAMPTZ`:** always use `TIMESTAMPTZ`. Plain `TIMESTAMP` throws away the timezone and will bite you the first time a server runs in UTC and a user is in IST. Store instants as `TIMESTAMPTZ` (effectively UTC), format to the user's zone in the UI.
- **Forgetting `WHERE` on `UPDATE`/`DELETE`:** rewrites/empties the whole table. Guard destructive changes inside a transaction so you can `ROLLBACK` (chapter 11).
- **"Why not just store one big JSON blob per user?"** You'd lose per-column constraints, foreign keys, joins, and index-backed filtering — the exact things that make Habex's dashboards fast and its money correct. Use `JSONB` for genuinely schemaless bits (say, per-user UI settings), not for your core relational data. The trade-off in full: [../system-design/09-sql-vs-nosql.md](../system-design/09-sql-vs-nosql.md).
- **The N+1 trap (teaser):** fetching a list, then firing one more query per row to get its category, is death by a thousand round-trips. The fix is a single `JOIN` (or a batched query). Covered in [11-transactions-and-query-performance.md](11-transactions-and-query-performance.md).
- **The slow query you'll eventually hit** is almost never "switch to NoSQL" — it's "add the right index." Keep that reflex; it's the highest-leverage DB skill you can carry into an interview.

---

## 8. TL;DR

- A **relational database** stores data in **tables** (rows + typed columns) with a fixed **schema**. **PostgreSQL** is the product-company default; you talk to it in **SQL**.
- Get it running instantly with one Docker command, connect via **psql** (`\dt`, `\d table`, `\q`) or a GUI (TablePlus/pgAdmin/DBeaver). The connection string `postgresql://user:pass@host:port/db` becomes your `DATABASE_URL`.
- Core types: `INTEGER`/`BIGINT`, `TEXT`, `BOOLEAN`, `TIMESTAMPTZ`, `IDENTITY` (prefer over `SERIAL`), `UUID`. **Store money as integer cents (`amount_cents`), never a float** — the fintech correctness signal.
- Essential SQL: `CREATE TABLE` (with `PRIMARY KEY`, `REFERENCES`, `NOT NULL`/`UNIQUE`/`CHECK`), `INSERT`, `SELECT` (`WHERE`/`ORDER BY`/`LIMIT`), `UPDATE`, `DELETE` — always with a `WHERE`.
- **Joins** reconnect tables: INNER = only matches, LEFT = keep everything on the left. **Aggregation** (`COUNT`/`SUM`/`GROUP BY`/`HAVING`) powers Habex's "spend per category this month" dashboard.
- **Foreign keys** enforce referential integrity; **indexes** (next chapters) keep queries fast.

Next: turn these tables into a designed, versioned schema — entities, relationships, `budgets`, `recurring_rules`, and **migrations** → [09-data-modeling-and-migrations.md](09-data-modeling-and-migrations.md).
