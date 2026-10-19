# 03. Project Setup: TypeScript + Node

> **One-liner:** After this you'll have a real, running `habex-api` project — TypeScript, ESM, hot-reload dev, a production build, and validated environment config — the exact skeleton every later chapter grows.
> **Where this fits:** Chapter 1 gave the mental model, chapter 2 the engine. This is where you stop reading and start *building*. From here to the [capstone](22-capstone-habex-backend.md) you add to this one project every chapter.

---

## 1. The Idea in Plain Words

You already scaffold React apps — but Vite/Next hides the setup. A backend has no `create-react-app` you'll want to lean on for interviews; you should be able to wire it by hand, because that's what the job (and the interview whiteboard) expects. The pieces:

| Term | What it is (frontend analogy) |
|---|---|
| **`package.json`** | The manifest: project name, dependencies, and **scripts** (like `npm run dev`). Same file you know from React. |
| **npm scripts** | Named shortcuts for commands (`dev`, `build`, `start`). |
| **dependency vs devDependency** | `dependencies` ship to production (Express, Zod). `devDependencies` are build/dev-only (TypeScript, `tsx`, type packages). |
| **`tsconfig.json`** | Tells the TypeScript compiler how to type-check and (optionally) compile your `.ts` to `.js`. |
| **`tsx` / `ts-node`** | Tools that *run* `.ts` files directly in development — no manual compile step. |
| **`tsc`** | The TypeScript compiler; in production we compile `.ts` → plain `.js` once, then run that with `node`. |
| **CommonJS (CJS) vs ES Modules (ESM)** | Two ways JS files import each other. CJS = `require()`; ESM = `import`/`export` (the syntax you use in React). We pick **ESM**. |
| **Environment variables (env vars)** | Config values (DB URL, secrets) read from *outside* the code via `process.env`, so the same build runs in dev/prod with different settings. |
| **`.env` file** | A local file holding your env vars for development. **Never committed** — it has secrets. |
| **`.gitignore`** | List of files/folders git must ignore (`node_modules`, `dist`, `.env`). |

**The one idea:** set up a project where you write modern TypeScript, run it instantly in dev, compile it for prod, and feed it config/secrets from the environment — not from source code.

---

## 2. Why It Matters

- **Everything composes from here.** The README promised the code snippets add up to one coherent backend. That only works if we all start from the *same* `habex-api` skeleton. This chapter is that skeleton.
- **TypeScript on the server pays off more than on the client.** A wrong type in a request handler can corrupt the database or leak data. Strict TS catches whole classes of bugs before they ship.
- **Config discipline is a security control, not a nicety.** The #1 way startups leak database passwords and API keys is committing a `.env` to a public repo. Getting this right *now* means you never learn it the hard way.
- **Interviewers ask about it.** "CommonJS vs ESM", "how do you manage config across environments", "dev vs prod build for a Node/TS app" are common, and the answers signal real-world experience.

---

## 3. Picture It (Diagram)

The project layout you'll build — flat for now, with the folders later chapters fill in (full rationale in [21-architecture-and-project-structure.md](21-architecture-and-project-structure.md)):

```
habex-api/
├─ src/                     ← all your TypeScript source lives here
│  ├─ index.ts              ← entry point: boots the HTTP server  (this chapter)
│  ├─ config.ts             ← typed + VALIDATED env config        (this chapter)
│  ├─ app.ts                ← Express app: middleware + routes     (ch04–05)
│  ├─ routes/               ← HTTP endpoints                       (ch04, ch06)
│  ├─ controllers/          ← parse request → call service        (ch21)
│  ├─ services/             ← business logic (the real work)      (ch21)
│  ├─ repositories/         ← database access                     (ch10–11)
│  ├─ middleware/           ← auth, validation, error handling    (ch05, ch07, ch12)
│  ├─ db/                   ← pg pool / Prisma client             (ch10)
│  └─ lib/                  ← shared helpers (logger, etc.)        (ch17)
│
├─ .env                     ← REAL secrets — GITIGNORED, never committed
├─ .env.example             ← placeholder template — committed, safe to share
├─ .gitignore               ← ignores node_modules/, dist/, .env
├─ package.json             ← deps + scripts
├─ tsconfig.json            ← TypeScript config for Node + ESM
└─ dist/                    ← compiled JS (created by `npm run build`, gitignored)
```

How the two run-modes flow through it:

```
  DEV (fast feedback):   src/index.ts ──tsx watch──▶ runs directly, reloads on save
  PROD (optimized):      src/*.ts ──tsc build──▶ dist/*.js ──node──▶ runs compiled JS
                                    (config/secrets injected via env vars, not code)
```

---

## 4. Hands-On (TypeScript + Node)

We'll initialize the real `habex-api` now. Run these in an empty folder.

### Step 1 — create the project & install deps

```bash
mkdir habex-api && cd habex-api
npm init -y                                   # creates package.json

# runtime deps (ship to prod):
npm install express dotenv zod

# dev-only deps (build/run TS, type definitions):
npm install -D typescript tsx @types/node @types/express
```

- `express` — web framework (chapter 4). `dotenv` — loads `.env` into `process.env`. `zod` — validation (we'll even validate our *own config* with it).
- `tsx` — runs `.ts` directly and watches for changes (built on esbuild, very fast). `@types/*` — type definitions so TS understands Node and Express.

### Step 2 — make it an ESM project & add scripts

Open `package.json` and set `"type": "module"` plus scripts. The `type` field flips the whole project from CommonJS to ES Modules — so `import`/`export` (the syntax you already use in React) works natively:

```jsonc
// package.json
{
  "name": "habex-api",
  "version": "0.1.0",
  "private": true,
  "type": "module",                        // ← makes .js files ES Modules
  "engines": { "node": ">=20" },           // document the Node version you target
  "scripts": {
    "dev": "tsx watch src/index.ts",       // dev: run TS directly, reload on save
    "build": "tsc -p tsconfig.json",       // prod: compile src/*.ts → dist/*.js
    "start": "node dist/index.js",         // prod: run the compiled output
    "typecheck": "tsc --noEmit"            // CI: type-check without emitting files
  }
}
```

### Step 3 — configure TypeScript for Node + ESM

Create `tsconfig.json`:

```jsonc
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022",                 // modern Node supports this; no downleveling needed
    "lib": ["ES2022"],
    "module": "NodeNext",               // emit ESM the way Node expects
    "moduleResolution": "NodeNext",     // resolve imports the way Node ESM does
    "rootDir": "src",
    "outDir": "dist",                   // compiled JS lands here
    "strict": true,                     // all strict checks on — non-negotiable server-side
    "noUncheckedIndexedAccess": true,   // arr[i] is T | undefined — catches real bugs
    "esModuleInterop": true,            // clean default-imports of CJS packages
    "resolveJsonModule": true,          // allow importing .json
    "skipLibCheck": true,               // faster builds; skip type-checking node_modules
    "forceConsistentCasingInFileNames": true,
    "sourceMap": true                   // stack traces point to your .ts lines
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

> **ESM gotcha (memorize this):** with `NodeNext`, your **relative imports must include a `.js` extension**, even though the file on disk is `.ts`. You import `./config.js` and TypeScript resolves it to `config.ts`, because the *compiled output* will be `config.js`. Omit it and the prod build (`node dist/index.js`) throws `ERR_MODULE_NOT_FOUND`. Bare package imports (`express`) need no extension.

### Step 4 — typed, validated config (`src/config.ts`)

Don't sprinkle `process.env.WHATEVER` (typed `string | undefined`) across your code. Parse the environment **once, at startup**, with Zod. If something's missing or malformed, the app **fails fast with a clear message** instead of blowing up mid-request at 2 a.m.:

```ts
// src/config.ts
import 'dotenv/config';       // load .env into process.env BEFORE we read it
import { z } from 'zod';

// Describe every env var we need, with types, defaults, and constraints.
const EnvSchema = z.object({
  NODE_ENV: z.enum(['development', 'test', 'production']).default('development'),
  PORT: z.coerce.number().int().positive().default(3000), // env is strings; coerce to number
  DATABASE_URL: z
    .string()
    .url()
    .default('postgresql://habex:habex@localhost:5432/habex'), // real value added in ch10
  JWT_SECRET: z.string().min(16, 'JWT_SECRET must be at least 16 characters'), // ch12
});

const parsed = EnvSchema.safeParse(process.env);
if (!parsed.success) {
  // Fail fast and loud — never boot a half-configured server.
  console.error('Invalid environment configuration:');
  console.error(parsed.error.flatten().fieldErrors);
  process.exit(1);
}

// Export a fully-typed, guaranteed-valid config object. Autocomplete everywhere.
export const config = parsed.data;
export type Config = typeof config;
```

### Step 5 — the entry point (`src/index.ts`)

```ts
// src/index.ts
import express from 'express';
import { config } from './config.js'; // ← .js extension: the ESM rule from Step 3

const app = express();
app.use(express.json()); // parse JSON request bodies into req.body

// A health check: the tiny seed of the Habex API. Load balancers and your
// deploy platform hit this to know the server is alive (see ch17, ch20).
app.get('/health', (_req, res) => {
  res.json({ status: 'ok', env: config.NODE_ENV, uptime: process.uptime() });
});

const server = app.listen(config.PORT, () => {
  console.log(`habex-api running on http://localhost:${config.PORT} (${config.NODE_ENV})`);
});

// Graceful shutdown: on deploy/restart, stop accepting new requests and let
// in-flight ones finish instead of dropping them. Matters in production (ch20).
process.on('SIGTERM', () => {
  console.log('SIGTERM received — shutting down gracefully');
  server.close(() => process.exit(0));
});
```

### Step 6 — env files & `.gitignore` (the security-critical part)

Create `.env.example` — a **committed** template with *placeholder* values, so teammates know what to set:

```bash
# .env.example  (COMMIT THIS — no real secrets)
NODE_ENV=development
PORT=3000
DATABASE_URL=postgresql://habex:habex@localhost:5432/habex
JWT_SECRET=replace-with-a-long-random-string-at-least-16-chars
```

Create `.env` — your **real, local** values. This file is **never committed**:

```bash
# .env  (NEVER COMMIT — real secrets live here)
NODE_ENV=development
PORT=3000
DATABASE_URL=postgresql://habex:habex@localhost:5432/habex
JWT_SECRET=1f3c9a7e5b2d8f0c4a6e1d9b7c3f5a2e
```

Create `.gitignore` **before your first commit**:

```bash
# .gitignore
node_modules/
dist/
.env
.env.*.local
*.log
coverage/
```

> Generate a strong secret with `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"`. Never invent one by hand.

### Step 7 — run it

```bash
npm run dev
# → habex-api running on http://localhost:3000 (development)

curl http://localhost:3000/health
# → {"status":"ok","env":"development","uptime":0.42}
```

Edit `src/index.ts`, save, and watch `tsx watch` reload automatically. For a production dry-run:

```bash
npm run build   # tsc → dist/
npm start       # node dist/index.js  (this is what Railway/Render will run in ch20)
```

---

## 5. Apply It to Habex

**This chapter *is* Habex's backend.** The `habex-api` folder you just made is the foundation the entire track assembles onto. Nothing here is throwaway demo code:

| You built now | Grows into | Chapter |
|---|---|---|
| `src/index.ts` boot + `/health` | Full Express app with all routes mounted | [04](04-http-servers-and-express-basics.md), [06](06-rest-api-design-in-practice.md) |
| `src/config.ts` (Zod-validated) | Adds `REDIS_URL`, mail keys, refresh-token secrets | [12](12-authentication.md), [14](14-caching-with-redis.md) |
| `DATABASE_URL` in config | Wired to a real PostgreSQL pool / Prisma client | [10](10-connecting-node-to-postgres.md) |
| Empty `routes/ services/ repositories/` | The layered architecture that keeps Habex maintainable | [21](21-architecture-and-project-structure.md) |
| `.env` discipline | Same pattern, injected as platform env vars in the cloud | [20](20-deployment-and-devops.md) |

The Zod config pattern also foreshadows a track-wide correctness habit: **validate untrusted input at the boundary**. Here the "input" is the environment; in [07-input-validation-and-error-handling.md](07-input-validation-and-error-handling.md) it's the request body — same Zod, same fail-fast instinct. And when Habex stores money, remember it's always `amount_cents` as an **integer**, never a float (README correctness rule) — a discipline that starts with taking types seriously, exactly what strict TS buys you here.

Commit the skeleton now (this becomes chapter 22's starting point):

```bash
git init
git add .            # .gitignore keeps .env and node_modules OUT — verify with `git status`
git commit -m "chore: scaffold habex-api (TS + ESM + validated config)"
```

> **Do this every time:** after `git add .`, run `git status` and confirm **`.env` is not listed**. If it is, your `.gitignore` is wrong — fix it *before* committing. A committed secret is compromised even if you delete it in the next commit (it lives in git history forever).

---

## 6. Interview Questions (with answer pointers)

1. **"CommonJS vs ES Modules — what's the difference and which would you pick?"** *(common)*
   → CJS uses `require`/`module.exports`, loads synchronously, is the legacy Node default. ESM uses `import`/`export`, is the standard (same as browsers/React), supports top-level `await` and better tree-shaking. Pick ESM for new projects via `"type": "module"`; note the `.js`-extension-in-imports gotcha and occasional CJS interop friction.

2. **"How do you run TypeScript in development vs production?"** *(all)*
   → Dev: run `.ts` directly with `tsx`/`ts-node` for instant feedback and hot reload. Prod: compile once with `tsc` to `dist/`, then run plain `node dist/index.js` — faster startup, no compiler in the runtime image, catches type errors at build time.

3. **"How do you manage configuration and secrets across environments?"** *(CRED, Razorpay — security-minded)*
   → Env vars (12-factor). Local dev uses a gitignored `.env`; prod injects vars via the platform. Validate them at startup (Zod) and fail fast. Keep a committed `.env.example` template. Never hardcode secrets or commit `.env`.

4. **"Why validate environment variables at startup?"**
   → `process.env.X` is `string | undefined`; a missing/malformed value otherwise surfaces as a confusing runtime crash mid-request. Parsing once with a schema gives typed, guaranteed-valid config and a clear boot-time error instead.

5. **"What does `"type": "module"` do?"**
   → Tells Node to treat `.js` files in the package as ES Modules (enabling `import`/`export`, top-level await) instead of CommonJS.

6. **"dependencies vs devDependencies — why separate them?"**
   → `dependencies` are needed at runtime in prod (Express, Zod); `devDependencies` are only for building/testing (TypeScript, `tsx`, `@types/*`). Prod installs skip devDeps, yielding smaller, safer images.

7. **"Should you commit `node_modules` or the lockfile?"**
   → Never commit `node_modules` (huge, platform-specific — regenerate with `npm ci`). **Always** commit `package-lock.json` for reproducible installs across machines and CI.

8. **"You accidentally committed a `.env` with a live DB password. What now?"** *(scenario)*
   → Treat the secret as compromised: rotate it immediately (new password/keys). Removing it in a later commit isn't enough — it's in git history. Purge history if needed (e.g. `git filter-repo`), then add `.env` to `.gitignore`. Prevention > cleanup.

---

## 7. Trade-offs & Pitfalls ("Why Not Just X?")

- **"Why not `ts-node` instead of `tsx`?"** `ts-node` works but is slower and has more ESM friction. `tsx` (esbuild-based) starts fast, handles ESM/CJS cleanly, and has built-in `watch`. Note: `tsx` transpiles *without* type-checking for speed — so keep a separate `npm run typecheck` (and run it in CI) to actually enforce types.

- **"Why not skip TypeScript and write JS?"** You'd lose the strongest safety net exactly where mistakes are costliest (data corruption, auth bugs). Types also serve as always-accurate documentation across the codebase. On a server, TS earns its keep.

- **"Why not Deno or Bun — they run TS natively?"** Both are promising and skip some of this setup, but Node has the deepest ecosystem, hosting support, and interview relevance. We use Node to match what product companies run. (Bun/Deno are worth a mention as modern alternatives.)

- **"Why not CommonJS to avoid the `.js`-extension hassle?"** CJS sidesteps that one quirk but is the legacy direction; the ecosystem is moving to ESM, and ESM matches your React mental model. We accept the small quirk for the future-proof choice.

- **Pitfall: committing `.env`.** The classic, painful mistake. `.gitignore` it *before* the first commit and verify with `git status`.
- **Pitfall: forgetting `.js` in ESM relative imports.** Works under some dev runners but breaks the `tsc` prod build. Always add it.
- **Pitfall: reading `process.env` all over the code.** Centralize in `config.ts`; everything else imports typed `config`.
- **Pitfall: no graceful shutdown.** Without handling `SIGTERM`, deploys drop in-flight requests. The 3-line handler above prevents it.

---

## 8. TL;DR

- Scaffold a Node+TS backend by hand: `package.json` (set `"type": "module"` for **ESM**), `tsconfig.json` (`NodeNext`, `strict`), and npm scripts for `dev`/`build`/`start`.
- **Dev:** run `.ts` directly with **`tsx watch`** (fast, hot reload). **Prod:** compile with **`tsc`** to `dist/`, run with **`node`**.
- **ESM rule:** relative imports need a **`.js` extension** (`./config.js`), even for `.ts` files.
- **Config discipline (security-critical):** read config from **env vars**, **validate at startup with Zod** and fail fast, keep real values in a **gitignored `.env`**, commit a placeholder **`.env.example`**, and **never commit secrets**.
- What you built — `habex-api` with a `/health` endpoint and typed config — **is** the skeleton every remaining chapter extends, all the way to the [capstone](22-capstone-habex-backend.md).

**Next → [04-http-servers-and-express-basics.md](04-http-servers-and-express-basics.md):** turn this skeleton into a real API with Express routes, `req`/`res`, and your first Habex endpoints.
