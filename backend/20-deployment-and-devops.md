# 20. Deployment & DevOps

> **One-liner:** After this you'll take `habex-api` from "runs on my laptop" to "running for real users on the internet, redeploying itself safely on every push to `main`" — with a managed database, secrets, health checks, and a rollback button.
> **Where this fits:** This is the payoff of Part VI. It stands on [19-docker-and-containers.md](19-docker-and-containers.md) (the image you ship), [17-logging-monitoring-observability.md](17-logging-monitoring-observability.md) (the health checks and logs you rely on), and [03-project-setup-typescript-node.md](03-project-setup-typescript-node.md) (env config). It's the hands-on twin of [../system-design/05-load-balancers.md](../system-design/05-load-balancers.md) and [../system-design/19-reliability-patterns.md](../system-design/19-reliability-patterns.md), and it sets up the [capstone (22)](22-capstone-habex-backend.md).

---

## 1. The Idea in Plain Words

You already ship your React frontend to a host. **Deployment** for a backend is the same idea — put your code on a machine the public can reach — but with more moving parts, because a backend is *stateful and always-on*: it holds secrets, owns a database, and must keep answering while you push new code.

Let's define the vocabulary once:

| Term | Plain meaning |
|---|---|
| **Deployment** | Getting your built code running on a server the internet can reach. |
| **Hosting / host** | The company/machine that runs your process (Railway, Render, AWS…). |
| **PaaS** | *Platform as a Service.* You hand over a repo or image; the platform builds, runs, restarts, and scales it. Railway/Render. |
| **VPS** | *Virtual Private Server.* A bare Linux box you rent and manage entirely yourself (EC2, a DigitalOcean droplet). |
| **Serverless** | Functions that spin up per-request and disappear after (AWS Lambda, Vercel Functions). No server to babysit; different constraints. |
| **Managed database** | A Postgres/Redis the host runs, backs up, and patches for you — you just get a connection URL. |
| **Environment** | A named copy of the running system: `development` (your laptop), `staging`, `production`. |
| **Environment variable (env var)** | A `KEY=value` the OS hands your process at startup (`process.env.DATABASE_URL`). How config gets *in*. |
| **Secret** | A sensitive env var (DB password, `JWT_SECRET`, API keys). Same mechanism, extra care. |
| **CI** | *Continuous Integration.* On every push, a robot installs, typechecks, tests, and builds your code. |
| **CD** | *Continuous Deployment/Delivery.* After CI passes, the same pipeline ships it. |
| **Migration** | A versioned change to the DB schema (see [09](09-data-modeling-and-migrations.md)). |
| **Health check** | A tiny endpoint (`GET /health`) the platform pings to decide if an instance is alive. |
| **Zero-downtime deploy** | Shipping a new version without users ever seeing an error. |
| **Rollback** | Instantly reverting to the previous known-good version. |

### The four hosting shapes (and why we pick PaaS first)

| Option | You manage | Great for | The catch |
|---|---|---|---|
| **PaaS** (Railway, Render, Fly) | Almost nothing — push code, set env vars | Your first real backend; small teams | Less low-level control; costs more per CPU at huge scale |
| **Raw VPS** (EC2, droplet) | *Everything* — OS patches, nginx, TLS, systemd, firewalls, backups | Learning Linux; full control; cheap raw compute | You are now the ops team; easy to get security wrong |
| **Serverless** (Lambda, Vercel fns) | Just function code | Spiky/bursty traffic, glue tasks | Cold starts, execution time limits, awkward for a long-lived worker + persistent DB pool |
| **Big cloud** (AWS/GCP/Azure raw) | A galaxy of services | Large orgs at scale | Enormous complexity; overkill for one app |

**For Habex, start with a PaaS (Railway or Render).** It gives you a managed Postgres and Redis in one click, deploys straight from your GitHub repo, injects a `PORT`, terminates TLS, and does rolling deploys with health checks — all the *concepts* this chapter teaches, with none of the yak-shaving. You can graduate to a VPS or AWS later; you'll understand exactly what they're automating because you learned it here. This mirrors the "start simple, don't over-engineer" theme from [../system-design/21-architecture-patterns.md](../system-design/21-architecture-patterns.md).

---

## 2. Why It Matters

- **A project isn't a product until it's deployed.** "I built and deployed a full-stack app with CI/CD" is a far stronger interview line than "I have a repo." Habex live at a real URL *is* your story.
- **Deployment is where correctness and safety get real.** A bad migration or a leaked secret in production is a very different stakes level than on localhost. The disciplines here — safe migrations, health checks, rollbacks — are exactly what fintech interviewers (CRED, Razorpay) probe.
- **CI/CD makes shipping boring, and boring is the goal.** A robot that typechecks and tests every push catches the bug *before* your users do, and makes deploys a non-event you do ten times a day instead of a scary quarterly ritual.
- **The [Twelve-Factor](https://12factor.net) principles make your app portable and scalable.** Follow them and your app runs identically on your laptop, in CI, and in prod — and scales by just running more copies.

---

## 3. Picture It (Diagram)

The pipeline: one `git push` flows through CI, and only if it's green does it deploy the two Habex services against the managed datastores.

```mermaid
flowchart LR
    Dev["git push origin main"] --> GH["GitHub repo"]
    GH --> CI
    subgraph CI["GitHub Actions — CI"]
      direction LR
      I["npm ci"] --> T["typecheck"] --> L["lint"] --> U["test (vitest + supertest)"] --> B["build (tsc)"]
    end
    CI -->|pass| Hook["deploy hook"]
    CI -->|fail| Stop["block merge, notify — nothing ships"]
    Hook --> Rel
    subgraph Rel["Railway / Render — CD"]
      direction TB
      Mig["release step: prisma migrate deploy"] --> Web["habex-api (web service)"]
      Mig --> Wrk["habex-worker (BullMQ)"]
    end
    Web --> PG[("Managed Postgres")]
    Web --> RD[("Managed Redis")]
    Wrk --> PG
    Wrk --> RD
```

The rule the diagram encodes: **build once, release, then run** — migrations run in a dedicated release step *before* new code serves traffic, and the web and worker are separate running processes sharing one image, one database, and one Redis.

---

## 4. Hands-On (TypeScript + Node)

### 4a. The `package.json` scripts CI and the platform call

Everything downstream is just these named scripts, so they work identically locally, in CI, and in prod:

```jsonc
// package.json (scripts section)
{
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "prisma generate && tsc -p tsconfig.json",
    "start": "node dist/server.js",          // the web process
    "start:worker": "node dist/worker.js",   // the BullMQ worker process
    "typecheck": "tsc --noEmit",
    "lint": "eslint .",
    "test": "vitest run",
    "migrate:deploy": "prisma migrate deploy" // apply migrations, never prompts
  }
}
```

> `prisma migrate deploy` is the *production* migration command: it applies already-created migration files and never generates or resets anything. `prisma migrate dev` is for your laptop only.

### 4b. Bind to the platform's `PORT` and shut down gracefully

Two Twelve-Factor rules in code. The platform tells you which port to use via an env var, and it stops your old version by sending `SIGTERM` — you must finish in-flight requests and exit, or the platform kills you mid-request.

```ts
// src/server.ts
import { app } from './app';          // an Express app that does NOT call listen()
import { prisma } from './db/prisma';
import { redis } from './queue/connection';
import { logger } from './shared/logger';

const port = Number(process.env.PORT) || 3000; // factor XI: the platform injects PORT
const server = app.listen(port, () => logger.info({ port }, 'habex-api up'));

// factor IX — disposability: fast startup, GRACEFUL shutdown on SIGTERM.
async function shutdown(signal: string) {
  logger.info({ signal }, 'shutting down');
  server.close(async () => {          // 1. stop accepting new connections
    await prisma.$disconnect();       // 2. release DB pool
    await redis.quit();               // 3. release Redis
    process.exit(0);
  });
  setTimeout(() => process.exit(1), 10_000).unref(); // 4. backstop: force-exit if stuck
}
process.on('SIGTERM', () => shutdown('SIGTERM'));
process.on('SIGINT', () => shutdown('SIGINT'));
```

### 4c. Health checks (what the platform pings)

The platform routes traffic only to instances that pass a health check — this is what makes rolling deploys and failover automatic (deep-dive: [17-logging-monitoring-observability.md](17-logging-monitoring-observability.md), [../system-design/05-load-balancers.md](../system-design/05-load-balancers.md)).

```ts
// inside src/app.ts
app.get('/health', (_req, res) => res.json({ status: 'ok' })); // liveness: "process alive?"
app.get('/ready', async (_req, res) => {                       // readiness: "can I serve?"
  try {
    await prisma.$queryRaw`SELECT 1`;                          // DB reachable?
    res.json({ status: 'ready' });
  } catch {
    res.status(503).json({ status: 'not-ready' });            // 503 → pull me out of rotation
  }
});
```

### 4d. The CI pipeline — `.github/workflows/ci.yml`

Install → typecheck → lint → test (against throwaway Postgres+Redis) → build. It runs on every push and PR; on `main`, a second job triggers the deploy.

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    # Spin up real Postgres + Redis so integration tests hit the real thing.
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: habex_test
        ports: ['5432:5432']
        options: >-
          --health-cmd "pg_isready -U test"
          --health-interval 10s --health-timeout 5s --health-retries 5
      redis:
        image: redis:7
        ports: ['6379:6379']
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s --health-timeout 5s --health-retries 5
    env:
      DATABASE_URL: postgresql://test:test@localhost:5432/habex_test
      REDIS_URL: redis://localhost:6379
      JWT_SECRET: test-secret-not-used-in-prod-000000
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci                     # reproducible install from lockfile
      - run: npm run typecheck          # tsc --noEmit: fail fast on type errors
      - run: npm run lint
      - run: npx prisma migrate deploy  # build the test DB schema
      - run: npm test                   # vitest + supertest
      - run: npm run build              # prove it compiles to dist/

  deploy:
    needs: build                        # only if CI is green
    if: github.ref == 'refs/heads/main' # only on main, never on PRs
    runs-on: ubuntu-latest
    steps:
      - name: Trigger Render/Railway deploy
        run: curl -fsSL "$DEPLOY_HOOK_URL"
        env:
          DEPLOY_HOOK_URL: ${{ secrets.DEPLOY_HOOK_URL }} # stored in GitHub secrets
```

> **CI vs CD in this file:** the `build` job is **CI** (verify every change). The `deploy` job is **CD** (ship the verified change). The `if:` guard is the gate — PRs get verified but never deployed. Many teams skip the explicit `deploy` job entirely and let the platform *auto-deploy* on push to `main`, using GitHub **required status checks** so a red `build` blocks the merge. Both are correct; the hook version makes the CI→CD handoff visible.

### 4e. The platform config — `render.yaml` (Blueprint)

One declarative file describes all four pieces: the web API, the worker, managed Postgres, and managed Redis — with migrations run in a **pre-deploy** step so the schema is ready *before* new code serves traffic.

```yaml
# render.yaml — infrastructure as code for Habex
services:
  # ---- Web API (public) ----
  - type: web
    name: habex-api
    runtime: node                       # set to `docker` + dockerfilePath for max dev/prod parity (ch19)
    plan: starter
    buildCommand: npm ci && npm run build
    startCommand: npm run start
    healthCheckPath: /health            # platform gates the rolling deploy on this
    preDeployCommand: npm run migrate:deploy  # runs ONCE per release, before traffic shifts
    envVars:
      - key: NODE_ENV
        value: production
      - key: DATABASE_URL
        fromDatabase: { name: habex-db, property: connectionString }
      - key: REDIS_URL
        fromService: { name: habex-redis, type: redis, property: connectionString }
      - key: JWT_SECRET
        sync: false                     # set by hand in the dashboard; never in git

  # ---- Background worker (no public port) ----
  - type: worker
    name: habex-worker
    runtime: node
    plan: starter
    buildCommand: npm ci && npm run build
    startCommand: npm run start:worker
    envVars:
      - key: NODE_ENV
        value: production
      - key: DATABASE_URL
        fromDatabase: { name: habex-db, property: connectionString }
      - key: REDIS_URL
        fromService: { name: habex-redis, type: redis, property: connectionString }

  # ---- Managed Redis (private; BullMQ needs noeviction) ----
  - type: redis
    name: habex-redis
    plan: starter
    maxmemoryPolicy: noeviction         # REQUIRED by BullMQ — never drop queued jobs
    ipAllowList: []                     # empty = internal-only, no public access

databases:
  # ---- Managed Postgres (daily backups by the platform) ----
  - name: habex-db
    plan: basic-256mb                   # plan names change; pick the smallest that fits
    databaseName: habex
    user: habex
```

The equivalent on **Railway** is dashboard-driven plus a small `railway.json` per service; you add the Postgres and Redis *plugins*, set `startCommand` to `npm run start` (web) and `npm run start:worker` (worker), and put `npm run migrate:deploy` in the service's **pre-deploy command**:

```jsonc
// railway.json (web service)
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": { "builder": "NIXPACKS" },
  "deploy": {
    "startCommand": "npm run start",
    "preDeployCommand": "npm run migrate:deploy",
    "healthcheckPath": "/health",
    "restartPolicyType": "ON_FAILURE"
  }
}
```

### 4f. Safe migrations on deploy — expand/contract

The dangerous moment in any deploy is the schema change, because for a few seconds **old code and new code run at the same time** (rolling deploy). A destructive migration (`DROP COLUMN`, rename) run against still-live old code crashes it. The fix is the **expand/contract** (a.k.a. parallel-change) pattern: make every change backward-compatible, split across multiple deploys.

Say you want to rename `transactions.note` → `transactions.memo`:

| Phase | Migration | Code deployed | Old code still works? |
|---|---|---|---|
| **Expand** | `ADD COLUMN memo TEXT` (nullable) | writes both `note` and `memo`, reads `note` | ✅ ignores `memo` |
| **Backfill** | `UPDATE … SET memo = note` | reads `memo`, still writes both | ✅ |
| **Contract** | `DROP COLUMN note` | reads/writes only `memo` | ✅ (deployed first) |

Additive changes (a new nullable column, a new table, a new index built `CONCURRENTLY`) are safe in one step. Only the *removing/renaming* half needs the multi-deploy dance. This is the single most important operational skill in this chapter — see reliability in [../system-design/19-reliability-patterns.md](../system-design/19-reliability-patterns.md).

---

## 5. Apply It to Habex

Here's the full production topology. Your frontend already lives on its own host; the backend adds two services plus two managed datastores on Railway/Render.

```
                 Browser (your React app)
                          │
        habex.santhu.site │ (frontend: Vercel/Netlify/Render static)
                          │
                          │  fetch("https://api.habex.santhu.site/...")
                          ▼
        ┌──────────────────────────────────────────────┐
        │              RAILWAY / RENDER                 │
        │                                               │
        │   api.habex.santhu.site                       │
        │   ┌───────────────┐        ┌───────────────┐  │
        │   │  habex-api    │─enqueue│  habex-worker │  │
        │   │  (web, public)│───────▶│  (BullMQ, no  │  │
        │   └──────┬─┬──────┘  jobs  │  public port) │  │
        │          │ │               └──────┬─┬──────┘  │
        │   SQL    │ │  cache/queue    SQL  │ │ jobs    │
        │          ▼ ▼                      ▼ ▼         │
        │   ┌────────────┐            ┌────────────┐    │
        │   │ Postgres   │            │  Redis     │    │
        │   │ (managed)  │            │ (managed)  │    │
        │   └────────────┘            └────────────┘    │
        └──────────────────────────────────────────────┘
```

**Two DNS records** (at your domain registrar) point the two hostnames at the two hosts:

| Type | Host | Points to |
|---|---|---|
| CNAME | `habex.santhu.site` | your frontend host |
| CNAME | `api.habex.santhu.site` | your Railway/Render `habex-api` service |

**Because the frontend and API are on different subdomains, the browser enforces CORS** — the API must explicitly allow its frontend origin (full treatment in [13-authorization-and-security.md](13-authorization-and-security.md)):

```ts
// src/app.ts — allow only the real frontend in prod, localhost in dev.
import cors from 'cors';
const allowedOrigins =
  process.env.NODE_ENV === 'production'
    ? ['https://habex.santhu.site']
    : ['http://localhost:5173'];
app.use(cors({ origin: allowedOrigins, credentials: true }));
```

**Env vars per service** — same image, different roles, secrets never in git:

| Variable | `habex-api` | `habex-worker` | Source |
|---|---|---|---|
| `NODE_ENV=production` | ✅ | ✅ | literal in config |
| `PORT` | ✅ (injected) | — | platform |
| `DATABASE_URL` | ✅ | ✅ | managed Postgres |
| `REDIS_URL` | ✅ | ✅ | managed Redis |
| `JWT_SECRET` | ✅ | — | dashboard secret |
| `CORS_ORIGIN` | ✅ | — | dashboard |

**The deploy loop you now have:** push to `main` → GitHub Actions typechecks/tests/builds → green triggers the platform → the release step runs `prisma migrate deploy` → the new `habex-api` boots, passes `/health`, and takes traffic while the old one drains → the `habex-worker` restarts too. If anything breaks, hit **Rollback** in the dashboard to instantly re-point to the previous release (your backward-compatible migrations mean the old code still runs against the new schema). The capstone [(22)](22-capstone-habex-backend.md) wires this end-to-end.

---

## 6. Interview Questions (with answer pointers)

1. **"What is the Twelve-Factor App? Name the factors that matter most."** *(all)*
   → A methodology for portable, scalable services. Hit: **config in the environment** (not in code), **stateless/share-nothing processes** (state lives in Postgres/Redis, so you can run many copies), **port binding** (the app is self-contained and binds `$PORT`), **logs to stdout** as an event stream, **strict build/release/run separation**, and **dev/prod parity** (Docker from [19](19-docker-and-containers.md) makes laptop ≈ prod). Bonus: run admin tasks (migrations) as one-off release processes.

2. **"Where do secrets and config go, and why not in the repo?"** *(all, security-flavored)*
   → In environment variables / the platform's secret store, injected at runtime; `.env` is git-ignored. Reasons: the repo is shared and cloned widely (a leaked key is a breach), and the *same build* must run in dev/staging/prod with only config differing. Rotate secrets; never log them.

3. **"Difference between CI and CD?"** *(all)*
   → CI = continuously *verify* every change (install, typecheck, test, build) so `main` is always releasable. CD = continuously *ship* what CI verified. CI protects quality; CD removes the manual, error-prone deploy step. CI is the gate; CD is the door.

4. **"How do you deploy with zero downtime?"** *(Swiggy, Zepto, Groww)*
   → Run multiple instances behind a load balancer; deploy a **rolling** update — start the new version, wait for it to pass a **health check**, shift traffic, then drain and stop the old one. The app must handle **SIGTERM gracefully** (finish in-flight requests) and be **stateless** so any instance can serve any request. Tie to [../system-design/05-load-balancers.md](../system-design/05-load-balancers.md).

5. **"How do you run database migrations safely during a deploy?"** *(CRED, Razorpay — high value)*
   → Make them **backward-compatible** and use **expand/contract**: add columns/tables first (safe), deploy code that tolerates both shapes, backfill, then remove the old shape in a *later* deploy. Never destructive-change-and-deploy in one step, because old and new code overlap during a rolling deploy. Run migrations as a **release-phase step**, decoupled from app boot, once per deploy — not per instance.

6. **"What's a rollback, and why can migrations make it hard?"** *(CRED, Razorpay)*
   → Rollback = re-point to the previous known-good release/image (fast, one click). Code rolls back cleanly; **schema doesn't** — if you already dropped a column, the old code breaks. That's *why* migrations must be backward-compatible: it keeps rollback safe. In practice you often **roll forward** (ship a fix) rather than un-migrate.

7. **"Explain blue-green and canary deploys."** *(senior signal)*
   → **Blue-green:** run two full environments; deploy to the idle one (green), test it, then flip the load balancer from blue→green instantly (and flip back to roll back). **Canary:** release to a small % of traffic first, watch error rates/latency, then ramp to 100% (or abort). Both reduce blast radius; both assume statelessness and backward-compatible schemas.

8. **"Why Railway/Render over raw AWS EC2 for this app?"**
   → PaaS gives managed Postgres/Redis, TLS, `PORT` injection, rolling deploys, health checks, and backups out of the box, so you ship in an afternoon and focus on the app. A VPS/EC2 means you own OS patching, nginx, systemd, firewalls, and backups — more control, far more surface area to get wrong. Right-size the ops burden to the team; scale up the platform later.

---

## 7. Trade-offs & Pitfalls ("Why Not Just X?")

- **"Why not just SSH in and `git pull`?"** It works exactly once. There's no build isolation, no tests gate, no rollback, no health check, and a botched pull leaves prod half-updated with no record of what's running. CI/CD gives you repeatability and an undo button.

- **"Why not serverless functions for everything?"** Serverless shines for spiky, short tasks, but Habex has a *long-lived* worker consuming a queue and a persistent DB connection pool — awkward and pricey to model as per-request functions (cold starts, connection storms, execution limits). A long-running process is simpler to reason about and interview on. (Serverless is still *a* backend — see [01](01-what-is-backend.md).)

- **"Why not Kubernetes now?"** k8s is a powerful way to orchestrate many services across many machines — and massive overkill for one API + one worker. You'd spend weeks on cluster ops instead of shipping. PaaS *is* your orchestrator for now; reach for k8s when you have many services and a platform team.

- **Pitfall: config baked into the image.** Hard-coding `DATABASE_URL` or origins means you rebuild to change environments and risk shipping a prod secret in a layer. Config lives in env vars (factor III), always.

- **Pitfall: destructive migration + rolling deploy.** `DROP COLUMN` while old instances still read it = crashes for the duration of the rollout. Expand/contract, always.

- **Pitfall: no graceful shutdown.** Without `SIGTERM` handling, every deploy kills in-flight requests and mid-flight jobs. Users see 502s; jobs get orphaned.

- **Pitfall: logging to a file.** On a PaaS the disk is ephemeral and there are many instances — file logs vanish on redeploy and can't be aggregated. Log to **stdout** (factor XI) and let the platform collect them ([17](17-logging-monitoring-observability.md)).

- **Pitfall: secrets in the frontend or in git.** Anything in the React bundle or the repo is public. Backend-only, secret-store-only.

---

## 8. TL;DR

- **Deployment** = running your built backend on an always-on, internet-reachable host with a managed DB, secrets, health checks, and a rollback button.
- Pick a **PaaS (Railway/Render)** first: managed Postgres + Redis, deploy-from-GitHub, `PORT` injection, TLS, and rolling deploys — the concepts without the ops toil. VPS/serverless/big-cloud come later.
- Follow **Twelve-Factor**: config in env, stateless processes, bind `$PORT`, logs to stdout, strict build→release→run, dev/prod parity (via Docker, [19](19-docker-and-containers.md)).
- **CI** verifies every push (install → typecheck → lint → test → build); **CD** ships what CI verified. Gate deploys on `main` + green checks.
- Run migrations in a **release step** (`prisma migrate deploy`) and keep them **backward-compatible** with **expand/contract** — that's what makes rolling deploys and rollbacks safe.
- **Zero-downtime** = multiple stateless instances + health checks + graceful `SIGTERM` + backward-compatible schema; blue-green/canary shrink the blast radius further.
- Habex ships as **two services** (`habex-api` web + `habex-worker`) on one image, against **managed Postgres + Redis**, with `api.habex.santhu.site` fronting the API and CORS allowing `habex.santhu.site`.

**Next → [21-architecture-and-project-structure.md](21-architecture-and-project-structure.md):** now that it deploys, structure the code so it stays maintainable — layers, DI, and a modular monolith you could split later.
