# 19. Docker & Containers

> **One-liner:** After this you'll be able to package the entire Habex backend — API, BullMQ worker, Postgres, and Redis — into containers that boot with one command (`docker compose up`) and run *identically* on your laptop, in CI, and in production.
> **Where this fits:** This takes the project you set up in [03-project-setup-typescript-node.md](03-project-setup-typescript-node.md) and makes it portable. It bundles the worker from [15-background-jobs-and-queues.md](15-background-jobs-and-queues.md), assumes the health checks from [17-logging-monitoring-observability.md](17-logging-monitoring-observability.md), and is the last stop before you ship in [20-deployment-and-devops.md](20-deployment-and-devops.md).

---

## 1. The Idea in Plain Words

You've hit this: code runs on your machine, breaks on a teammate's, and nobody knows why. Different Node version, a missing Postgres, an env var you forgot you set months ago. **"Works on my machine"** is not a joke — it's the single biggest source of wasted time in backend work.

The cause is that your app secretly depends on its *environment*: a specific Node version, a running Postgres, a Redis, certain OS libraries, certain env vars. Copying the source code copies none of that.

A **container** fixes this by shipping the app *and its whole environment together* as one sealed unit. Inside the container is a pinned Node, your compiled code, its dependencies, and the OS libraries it needs — frozen. That same sealed unit runs bit-for-bit the same everywhere, because it *carries* its environment with it.

Let's define the terms:

| Term | Plain meaning |
|---|---|
| **Container** | A running, isolated process that carries its own filesystem, dependencies, and config — but shares the host's OS kernel. Your app in a sealed box. |
| **Image** | The read-only *blueprint* a container is created from — app code + deps + OS libs, built once. Think "class". |
| **Container (instance)** | A live, running copy of an image. Think "object". You can run many containers from one image. |
| **Dockerfile** | A recipe (text file) listing the steps to build an image. |
| **Layer** | Each Dockerfile instruction produces a cached filesystem layer; unchanged layers are reused between builds (fast rebuilds). |
| **Registry** | A store for images (Docker Hub, GitHub Container Registry) — like npm, but for images. |
| **Volume** | Storage that *outlives* a container, so a DB's data survives restarts. |
| **docker-compose** | A tool to define and run *multiple* containers together (api + worker + postgres + redis) from one YAML file. |

### Container vs Virtual Machine

People confuse these constantly, and it's a guaranteed interview question. A **VM** virtualizes a *whole machine* — it runs a full guest operating system with its own kernel on top of a hypervisor. A **container** virtualizes only the *process* — it shares the host's kernel and just isolates the filesystem, network, and process view. That difference is everything:

| | Virtual Machine | Container |
|---|---|---|
| Virtualizes | A whole machine (guest OS **+ its own kernel**) | Just the process (**shares the host kernel**) |
| Size | Gigabytes | Tens–hundreds of MB |
| Startup | Seconds to minutes | Milliseconds to a couple seconds |
| Isolation | Strong (separate kernel) | Process-level (Linux namespaces + cgroups) |
| Overhead | Heavy (run N full OSes) | Light (run N processes) |
| Analogy | A whole house per tenant | Apartments in one building sharing plumbing |

So containers give you *most* of a VM's isolation at a *fraction* of the cost — which is why you can run your API, worker, Postgres, and Redis as four containers on one laptop without it melting.

### Images vs containers (the one-liner that lands)

**An image is the blueprint; a container is a running instance of it.** You build the image once, then run many containers from it. Containers are *ephemeral* — throw one away and start another from the same image, and you're in an identical state. (That's why databases need **volumes**: the data must live *outside* the disposable container.)

---

## 2. Why It Matters

**For Habex:** your backend is now four moving parts — Express API, BullMQ worker, Postgres, Redis. Onboarding a teammate (or future-you on a new laptop) shouldn't mean "install Postgres 16, then Redis, then match my Node version, then set six env vars." With Docker it's `docker compose up`. One command, whole stack, identical every time.

**For prod parity:** the container that passes your tests in CI is the *exact same image* that runs in production. No "it worked in staging" surprises — staging and prod run the identical artifact. This is the backbone of reliable deploys ([20-deployment-and-devops.md](20-deployment-and-devops.md)).

**For interviews:** "container vs VM", "image vs container", "why multi-stage builds", and "how would you shrink an image" are standard questions at Zepto/Swiggy/CRED/Razorpay/Groww. They signal you can operate software, not just write it. Being able to read and write a Dockerfile is now a baseline backend skill.

---

## 3. Picture It (Diagram)

`docker-compose` brings up four services on one private network. They find each other by **service name** (the API connects to `postgres:5432`, not an IP), and Postgres persists to a named volume so data survives restarts:

```
   Your laptop  (docker compose up)
   ┌──────────────────────────────────────────────────────────────────────┐
   │                     docker network: "habex_default"                    │
   │                                                                        │
   │   :3000 ┌───────────────┐        ┌───────────────┐                     │
   │ ◀──────▶│   api         │        │   worker      │   (SAME image,      │
   │ host    │ node dist/    │        │ node dist/    │    different CMD)    │
   │         │  server.js    │        │  worker.js    │                     │
   │         └──────┬────────┘        └──────┬────────┘                     │
   │                │  DATABASE_URL          │  REDIS_URL                    │
   │        postgres:5432 / redis:6379 (by service NAME, not IP)            │
   │                ▼                        ▼                               │
   │         ┌───────────────┐        ┌───────────────┐                     │
   │         │  postgres     │        │  redis        │                     │
   │         │  :5432        │        │  :6379        │                     │
   │         └──────┬────────┘        └───────────────┘                     │
   │                │ volume mount                                           │
   │         ┌──────▼────────┐                                              │
   │         │ pgdata (named │  ← data OUTLIVES the container (survives      │
   │         │   volume)     │     `docker compose down` unless -v)          │
   │         └───────────────┘                                              │
   └──────────────────────────────────────────────────────────────────────┘
```

```mermaid
flowchart LR
    host((Host :3000)) <--> api
    subgraph net["docker network: habex_default"]
        api["api\nnode dist/server.js"]
        worker["worker\nnode dist/worker.js\n(same image)"]
        pg[("postgres :5432")]
        redis[("redis :6379")]
        api -->|DATABASE_URL| pg
        api -->|REDIS_URL| redis
        worker -->|DATABASE_URL| pg
        worker -->|REDIS_URL| redis
    end
    pg -. persists .-> vol[("pgdata volume")]
```

The key ideas the diagram encodes: **one shared network** (services reach each other by name), **api and worker share one image** (built once, run twice with different start commands), and **the volume keeps the database durable** across container restarts.

---

## 4. Hands-On (TypeScript + Node)

### 4.1 A multi-stage Dockerfile for `habex-api`

A **multi-stage build** uses one stage with the *full* toolchain to compile TypeScript and generate the Prisma client, then copies only the *results* into a slim final image that carries no compiler, no dev dependencies, no test libraries. Smaller image = faster deploys, less to attack.

```dockerfile
# syntax=docker/dockerfile:1

##### Stage 1 — build: full toolchain. Compiles TS and generates the Prisma client. #####
FROM node:lts-alpine AS build
WORKDIR /app

# Prisma's query engine needs OpenSSL on Alpine.
RUN apk add --no-cache openssl

# Copy ONLY the manifests first. This layer is cached and re-runs `npm ci` *only*
# when package.json / lockfile change — not on every source edit. Big speedup.
COPY package.json package-lock.json ./
RUN npm ci

# Now bring in the source and build.
COPY . .
RUN npx prisma generate        # typed client → node_modules/.prisma
RUN npm run build              # tsc → dist/

##### Stage 2 — runner: slim final image. Production deps only, non-root. #####
FROM node:lts-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
RUN apk add --no-cache openssl

# Install ONLY production deps — no tsc, no vitest → smaller, smaller attack surface.
COPY package.json package-lock.json ./
RUN npm ci --omit=dev && npm cache clean --force

# Copy the build outputs from stage 1: compiled JS, generated Prisma client, schema.
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules/.prisma ./node_modules/.prisma
COPY --from=build /app/prisma ./prisma

# SECURITY: drop root. The official node image ships an unprivileged 'node' user.
# If the app is ever compromised, it isn't running as root inside the container.
USER node

EXPOSE 3000                    # documents the port (informational; publishing is done by -p / compose)
CMD ["node", "dist/server.js"] # the start command; exec form so signals (SIGTERM) reach Node
```

Why each highlighted choice matters:

| Choice | Reason |
|---|---|
| **Multi-stage** | Final image has no compiler/dev deps → often 10× smaller and safer |
| **`node:lts-alpine`** | Alpine is a tiny (~5MB) Linux base → small images |
| **Copy `package.json` first** | Layer caching: deps reinstall only when the lockfile changes, not on every code change |
| **`npm ci --omit=dev`** | Ship runtime deps only |
| **`USER node`** | Never run as root inside a container (privilege-escalation defense) |
| **`EXPOSE 3000`** | Documents which port the app listens on |
| **Exec-form `CMD`** | `SIGTERM` reaches Node directly so it can shut down gracefully (drain requests, close the DB pool) |

### 4.2 `.dockerignore` (do this first — it's easy to forget)

Without it, `COPY . .` drags your local `node_modules`, `.git`, `.env`, and `dist` into the build — bloating the image, breaking caches, and **leaking secrets**. It's the Docker sibling of `.gitignore`:

```dockerignore
node_modules
dist
.git
.gitignore
.env
.env.*
Dockerfile
docker-compose.yml
coverage
*.md
.vscode
npm-debug.log
```

### 4.3 Build and run just the API image

```bash
# Build the image and tag it.
docker build -t habex-api .

# Run one container. --env-file passes runtime config; -p publishes the port to your host.
docker run --rm -p 3000:3000 --env-file .env habex-api
# → curl http://localhost:3000/healthz
```

### 4.4 Environment variables & secrets at runtime

Config and secrets are **injected at runtime**, never baked into the image (an image is shared and could be pushed to a registry — bake in a secret and you've published it). This is the 12-factor rule ([20-deployment-and-devops.md](20-deployment-and-devops.md)): the *same* image gets different config in dev vs prod purely through env vars.

| Where | How secrets arrive |
|---|---|
| Local dev | `--env-file .env`, or compose's `environment:` / `env_file:` |
| Production | The platform's secret store (Railway/Render/K8s Secrets) injects env vars into the container |
| **Never** | Hard-coded in the Dockerfile, or committed in `.env` (it's in `.dockerignore` + `.gitignore`) |

### 4.5 `docker-compose.yml` — the whole Habex stack

Now wire all four services together. Note `depends_on` with **healthchecks**: the API won't start until Postgres and Redis are actually *ready* (accepting connections), not merely *started* — this eliminates the classic "API crashes on boot because the DB isn't up yet" race.

```yaml
# docker-compose.yml
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: habex
      POSTGRES_PASSWORD: habex_dev_pw
      POSTGRES_DB: habex
    ports:
      - "5432:5432"                       # expose to host so you can psql/Prisma Studio locally
    volumes:
      - pgdata:/var/lib/postgresql/data   # named volume → data survives `compose down`
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U habex"]
      interval: 5s
      timeout: 3s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5

  # One-shot: apply DB migrations, then exit. Uses the BUILD stage (it has the Prisma CLI).
  migrate:
    build:
      context: .
      target: build                       # the build stage still has devDeps + prisma CLI
    command: npx prisma migrate deploy
    environment:
      DATABASE_URL: postgres://habex:habex_dev_pw@postgres:5432/habex
    depends_on:
      postgres:
        condition: service_healthy

  api:
    build: .                              # builds the Dockerfile above (runner stage)
    command: node dist/server.js
    environment:
      NODE_ENV: production
      DATABASE_URL: postgres://habex:habex_dev_pw@postgres:5432/habex
      REDIS_URL: redis://redis:6379
    ports:
      - "3000:3000"
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      migrate:
        condition: service_completed_successfully  # start only after migrations finish

  worker:
    build: .                              # SAME image as api…
    command: node dist/worker.js          # …just a different start command → the BullMQ worker
    environment:
      NODE_ENV: production
      DATABASE_URL: postgres://habex:habex_dev_pw@postgres:5432/habex
      REDIS_URL: redis://redis:6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      migrate:
        condition: service_completed_successfully

volumes:
  pgdata:                                 # declares the named volume used by postgres
```

Two things to notice. **The network is automatic:** compose creates a private network and each service is reachable by its name — that's why `DATABASE_URL` says `postgres:5432`, not an IP. **Migrations run as their own step:** the `migrate` service applies the schema and exits, and both `api` and `worker` wait for it to *complete successfully* before booting — a clean, race-free startup order.

### 4.6 Run the whole thing

```bash
docker compose build          # build the api/worker/migrate images
docker compose up             # start everything (add -d to run detached in the background)

# migrations run automatically via the `migrate` service; to run them manually instead:
docker compose run --rm migrate

docker compose logs -f api    # tail the API logs (your pino JSON from ch17)
docker compose ps             # see what's running and healthy
docker compose down           # stop and remove containers (KEEPS the pgdata volume)
docker compose down -v        # also delete the volume → wipes the database (fresh start)
```

That's the payoff: one command boots a production-shaped Habex — API, worker, Postgres, Redis, migrations — on any machine with Docker installed.

---

## 5. Apply It to Habex

Containerizing Habex makes the "four moving parts" concrete and reproducible:

| Service | Image | Role |
|---|---|---|
| **api** | your `habex-api` image | Serves the Express REST API on :3000 |
| **worker** | the **same** image, `CMD` = `node dist/worker.js` | Runs the BullMQ worker for recurring rules ([15](15-background-jobs-and-queues.md)) |
| **postgres** | `postgres:16-alpine` + `pgdata` volume | Durable store for users/transactions/budgets |
| **redis** | `redis:7-alpine` | Queue backend + cache ([14](14-caching-with-redis.md)) |

The **api-and-worker-share-one-image** pattern is worth internalizing: you build *one* artifact and run it two ways (different `CMD`). That guarantees the API and the worker use the *same* code, the *same* Prisma client, and the *same* dependencies — no drift between "the thing that writes transactions" and "the thing that posts recurring ones." When you deploy ([20](20-deployment-and-devops.md)), you push that one image and start two services from it.

This also means your local dev, your CI test run ([18-testing-backend.md](18-testing-backend.md) can point its test DB at a compose Postgres), and your production deploy are all running the *same containers*. "Works on my machine" becomes "works in the container, which is the same everywhere." And the `/healthz` you built in [17-logging-monitoring-observability.md](17-logging-monitoring-observability.md) is exactly what a compose healthcheck (or a prod orchestrator) polls to decide the container is ready.

---

## 6. Interview Questions (with answer pointers)

1. **"Container vs virtual machine?"** *(guaranteed warm-up)*
   → A VM virtualizes a whole machine including its own guest OS/kernel (heavy, GBs, slow boot). A container virtualizes just the process and **shares the host kernel** (light, MBs, boots in ms), isolating via Linux namespaces/cgroups. Containers give near-VM isolation at a fraction of the overhead.

2. **"Image vs container?"**
   → An image is the read-only blueprint (built from a Dockerfile); a container is a running instance of that image. One image → many containers. Containers are ephemeral, which is why persistent data lives in volumes.

3. **"Why a multi-stage build?"** *(CRED, Razorpay)*
   → To keep the *final* image slim and safe: a build stage carries the full toolchain (tsc, dev deps, Prisma CLI) to compile, and the runtime stage copies only the compiled output + prod deps. Result is far smaller and drops the compiler/dev-dependency attack surface.

4. **"How would you shrink a Docker image?"**
   → Multi-stage builds; a small base (`alpine` or distroless); `npm ci --omit=dev`; a good `.dockerignore` so junk never enters the context; order layers so cacheable steps (deps) come before frequently-changing ones (source); combine/clean up in the same layer (`npm cache clean`).

5. **"What does layer caching do and how do you exploit it?"**
   → Each instruction is a cached layer; unchanged layers are reused. Copy `package.json` and install deps *before* copying source, so editing code doesn't bust the (slow) dependency-install layer. Massively speeds up rebuilds.

6. **"How does docker-compose help?"**
   → It defines and runs a multi-container app from one YAML: builds/starts api + worker + postgres + redis, puts them on a shared network (reachable by service name), wires volumes for persistence, and orders startup with `depends_on` + healthchecks. One command spins up the whole stack reproducibly.

7. **"Where do secrets and config go?"**
   → Injected at **runtime** as environment variables (compose `environment`/`env_file` locally; the platform's secret store in prod) — never baked into the image or committed. Same image, different config per environment (12-factor). Add `.env` to `.dockerignore`.

8. **"How do you run database migrations with containers?"**
   → As a discrete step, not inside app boot: a one-shot `migrate` service (or a CI job / init container) runs `prisma migrate deploy` and exits; the app/worker start only after it *completes successfully*. Keeps schema changes explicit and startup race-free.

9. **"Why run the container as non-root?"**
   → Defense in depth: if the app is compromised, the attacker isn't root inside the container (harder to escalate or escape). Use the base image's `node` user via `USER node`.

---

## 7. Trade-offs & Pitfalls ("Why Not Just X?")

- **"Why not just install Node + Postgres + Redis directly on the server?"** That's exactly the "works on my machine" trap at scale — every server drifts (different versions, leftover state), onboarding is a wiki of manual steps, and prod ≠ dev. Containers make the environment part of the artifact, so it's identical everywhere and reproducible from a file.

- **"Why not one big container with the app AND Postgres AND Redis inside?"** It breaks the **one-process-per-container** principle: you can't scale the API independently of the DB, a crash in one can take the others, logs get tangled, and you lose managed-database benefits. One concern per container; compose wires them.

- **"Why not put data inside the container?"** Containers are ephemeral — recreate one and its filesystem is gone. Databases *must* use a **volume** so data outlives the container. (Forgetting this and running `compose down -v` is how people wipe their local DB.)

- **Pitfall: no `.dockerignore`.** `COPY . .` then sucks in `node_modules`, `.git`, and `.env` — bloating the image, poisoning the cache, and leaking secrets into a layer. Write it before your first build.

- **Pitfall: baking secrets into the image.** Anything `COPY`d or `ENV`d in the Dockerfile is baked into a shareable, pushable artifact. Secrets come from the runtime environment only.

- **Pitfall: `depends_on` without healthchecks.** Plain `depends_on` waits for the container to *start*, not to be *ready*. Your API then races ahead and crashes because Postgres isn't accepting connections yet. Use `condition: service_healthy`.

- **Pitfall: Prisma on Alpine.** The query engine needs OpenSSL and matching musl binaries — `apk add --no-cache openssl` (done above), or use `node:lts-slim` (Debian) if you hit engine issues. Know this one; it bites people.

- **Pitfall: shell-form `CMD`.** `CMD node dist/server.js` (shell form) can swallow `SIGTERM`, so your app never shuts down gracefully. Use exec form: `CMD ["node", "dist/server.js"]`.

---

## 8. TL;DR

- Containers solve **"works on my machine"** by shipping your app *with its whole environment* as one sealed unit that runs identically everywhere.
- **Container vs VM:** a container shares the host kernel and isolates just the process (light, MBs, boots in ms); a VM runs a whole guest OS (heavy, GBs). **Image vs container:** the image is the blueprint, the container is a running instance of it.
- Use a **multi-stage Dockerfile**: build with the full toolchain, then copy only the results into a slim `node:lts-alpine` runtime. Copy `package.json` first (layer caching), `npm ci --omit=dev`, run as **non-root**, `EXPOSE` the port, exec-form `CMD`.
- Add a **`.dockerignore`** (no `node_modules`/`.git`/`.env`), and inject **secrets at runtime** — never bake them into the image.
- **docker-compose** runs the whole Habex stack — api + worker + postgres + redis — on a shared network (reach services by name), with **volumes** for durable Postgres and **healthchecks** in `depends_on` so nothing starts before its dependencies are ready.
- Run **migrations as a one-shot step** (`prisma migrate deploy`) that completes before the app boots.
- For **Habex**, the api and worker **share one image** (different `CMD`) so they can never drift — the same artifact you'll push to prod in [20-deployment-and-devops.md](20-deployment-and-devops.md).

**Next → [20-deployment-and-devops.md](20-deployment-and-devops.md):** take these exact images and ship them — Railway/Render, environment config, and a GitHub Actions pipeline that builds, tests, and deploys on every push.
