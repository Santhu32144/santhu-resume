# 02. The Node.js Runtime & Event Loop

> **One-liner:** After this you'll be able to explain — and *demonstrate* — how a single-threaded Node process serves thousands of concurrent requests without blocking, and predict the exact order async code runs in.
> **Where this fits:** Chapter 1 showed a server is just a program on a port. This chapter is the engine inside that program. Understanding it is *why* the tiny server from chapter 1 can handle many users at once, and it's the #1 conceptual interview topic for Node roles.

---

## 1. The Idea in Plain Words

**Node.js** is a runtime that lets you run JavaScript *outside the browser* — on a server. It's built from two big pieces:

| Piece | What it is | What it gives Node |
|---|---|---|
| **V8** | Google's JavaScript engine (also inside Chrome) | Compiles and executes your JS/TS-compiled code, fast |
| **libuv** | A C library for async I/O | The **event loop** + a background **thread pool** for file/DNS/crypto work |

So Node = *"V8 to run the JavaScript + libuv to talk to the operating system without blocking."* On top of that, Node adds built-in modules you'll use constantly: `http`, `fs` (files), `crypto`, `process`, etc.

Here's the part that trips up every frontend dev: **your JavaScript runs on a single thread.** One. There is no parallel `for`-loop across cores by default. Yet Node powers some of the highest-throughput APIs in the world. How?

The trick is the **event loop** and the **non-blocking** model. Define the terms:

- **Thread:** a single sequence of instructions a CPU executes. "Single-threaded" = Node runs *your* JS one statement at a time, in order.
- **Blocking:** an operation that makes the thread *wait* and do nothing else until it finishes (e.g. reading a file synchronously). While blocked, your server can't answer anyone.
- **Non-blocking:** you *start* a slow operation (a DB query, reading a file) and immediately move on; Node notifies you later, via a callback/Promise, when the result is ready.
- **Event-driven:** Node's whole life is "wait for something to happen (an event), run the code registered for it, repeat."
- **Event loop:** the loop that watches for finished I/O and runs the callbacks waiting on it. It's the "manager" that keeps the single thread busy instead of idle-waiting.

**Mental model:** Node is a single, very fast waiter in a restaurant. He doesn't stand at one table until the kitchen finishes cooking — he takes an order, hands it to the kitchen (the OS / thread pool), and *immediately* serves other tables. When a dish is ready, the kitchen dings a bell (an event) and the waiter delivers it. One waiter, many tables, almost no idle time. That's non-blocking I/O.

---

## 2. Why It Matters

- **It explains Node's superpower and its Achilles' heel in one sentence:** Node is brilliant at **I/O-bound** work (waiting on databases, other APIs, disk, network — where the CPU is mostly idle) and bad at **CPU-bound** work (heavy computation that keeps the CPU busy), because CPU work on the single thread *blocks everyone*.
- **A backend is almost entirely I/O.** A Habex request = "wait for the DB, wait for Redis, maybe wait for an email service." That's exactly the workload Node is optimized for, which is why it's a fantastic fit.
- **One blocking line can freeze your entire server.** If you understand the event loop, you'll instinctively avoid the mistake that takes down real Node services: doing heavy CPU work (or a `*Sync` call) on the request path.
- **It's the interview gate.** "Explain the event loop", "what's the output of this `setTimeout`/`Promise` snippet", and "microtask vs macrotask" show up in nearly every Node interview (Zepto, Swiggy, CRED, Razorpay, Groww). Section 6 drills them.

---

## 3. Picture It (Diagram)

Two views. First, the big picture — your JS thread, the queues that feed it, and libuv underneath:

```
        YOUR SINGLE JS THREAD
        ┌───────────────────┐
        │    CALL STACK     │  ← runs one function at a time (synchronous code)
        └─────────▲─────────┘
                  │ event loop pushes the next ready callback here
                  │ when the stack is EMPTY
        ┌─────────┴───────────────────────────────────────┐
        │                 EVENT LOOP                       │
        │  after each tick, drain in this priority order:  │
        │   1) process.nextTick queue                      │
        │   2) microtask queue  (Promise .then/catch)      │
        │   3) one macrotask     (timers, I/O, setImmediate)│
        └─────────▲───────────────────────▲────────────────┘
                  │                        │
     ┌────────────┴─────┐        ┌─────────┴──────────────┐
     │  MICROTASKS      │        │  MACROTASKS (task queue)│
     │  Promise.then    │        │  setTimeout, setImmediate│
     │  queueMicrotask  │        │  fs/network callbacks    │
     └──────────────────┘        └──────────▲──────────────┘
                                            │ "ding! I/O done"
                            ┌───────────────┴───────────────┐
                            │   libuv  (C, under the hood)   │
                            │  • OS async network I/O        │
                            │  • thread pool (default 4) for │
                            │    fs / dns / crypto / zlib    │
                            └────────────────────────────────┘
```

Second, the **phases** libuv cycles through on every loop iteration (simplified). Between *every* phase, Node empties the `nextTick` queue and then the microtask (Promise) queue:

```
   ┌───────────────────────────┐
┌─▶│           timers          │  setTimeout / setInterval callbacks fire here
│  └────────────┬──────────────┘
│  ┌────────────▼──────────────┐
│  │     pending callbacks     │  a few deferred system I/O callbacks
│  └────────────┬──────────────┘
│  ┌────────────▼──────────────┐        ┌──────────────────┐
│  │           poll            │◀──────▶ │ incoming I/O:     │  ← the loop waits
│  │  (retrieve new I/O events)│        │ DB reply, socket  │    HERE for work
│  └────────────┬──────────────┘        └──────────────────┘
│  ┌────────────▼──────────────┐
│  │           check           │  setImmediate callbacks fire here
│  └────────────┬──────────────┘
│  ┌────────────▼──────────────┐
│  │      close callbacks      │  e.g. socket 'close' events
│  └────────────┬──────────────┘
└───────────────┘   loop repeats while there's work to do
```

---

## 4. Hands-On (TypeScript + Node)

### 4a. Blocking vs non-blocking, seen with your own eyes

```ts
// blocking-vs-nonblocking.ts
import { readFileSync } from 'node:fs';
import { readFile } from 'node:fs/promises';

// ---- BLOCKING (synchronous) ----
// readFileSync FREEZES the single thread until the whole file is read.
// While this runs, your server can answer NO other request. This is the trap.
console.log('A: before blocking read');
const data = readFileSync('./package.json', 'utf8');
console.log('B: after blocking read, length =', data.length);

// ---- NON-BLOCKING (async) ----
// readFile RETURNS a Promise immediately and hands the actual disk read to
// libuv's thread pool. Node is free to do other work until the file is ready.
console.log('C: before async read');
readFile('./package.json', 'utf8').then((async_data) => {
  console.log('E: async read finished, length =', async_data.length);
});
console.log('D: after async read (runs BEFORE E!)');
```

Run with `npx tsx blocking-vs-nonblocking.ts`. Output:

```
A: before blocking read
B: after blocking read, length = 512
C: before async read
D: after async read (runs BEFORE E!)   ← we moved on without waiting
E: async read finished, length = 512   ← callback fires later, off the event loop
```

`D` printing before `E` is the entire point: the async read didn't block; execution continued and the result arrived later. **Rule of thumb for servers: never use the `*Sync` file/crypto APIs on the request path.** They're fine in a one-off CLI script or at startup, never per-request.

### 4b. The classic ordering puzzle: microtask vs macrotask

This is *the* interview snippet. Type it, run it, and be able to explain it cold:

```ts
// event-loop-order.ts
console.log('1: synchronous start');

setTimeout(() => console.log('2: setTimeout 0  (MACROtask — timers phase)'), 0);

Promise.resolve().then(() => console.log('3: promise.then (MICROtask)'));

queueMicrotask(() => console.log('4: queueMicrotask (MICROtask)'));

process.nextTick(() => console.log('5: process.nextTick (runs before promises)'));

console.log('6: synchronous end');
```

Actual output:

```
1: synchronous start
6: synchronous end
5: process.nextTick (runs before promises)
3: promise.then (MICROtask)
4: queueMicrotask (MICROtask)
2: setTimeout 0  (MACROtask — timers phase)
```

**Why, step by step:**

1. All **synchronous** code runs first, top to bottom → `1`, then `6`. The `setTimeout`/`Promise`/`nextTick` calls only *schedule* callbacks; they don't run yet.
2. The call stack is now empty, so Node drains its **microtask-level** queues *before* touching any macrotask. `process.nextTick` has its own queue that runs *first* → `5`.
3. Then the **microtask queue** (Promise callbacks, `queueMicrotask`) drains in FIFO order → `3`, then `4`.
4. Only *after* all microtasks are gone does Node advance to a **macrotask** — the timer callback → `2`.

The one-liner to say in an interview: **"Sync code first, then `process.nextTick`, then microtasks (Promises), then one macrotask (timers/I/O/`setImmediate`) — and Node fully drains the microtask queue between every macrotask."**

### 4c. The danger: CPU-bound work blocks *everyone*

```ts
// cpu-block.ts — DO NOT do this on a request handler.
import { createServer } from 'node:http';

// A deliberately expensive, purely-CPU function (no I/O to hand off).
function slowFib(n: number): number {
  return n < 2 ? n : slowFib(n - 1) + slowFib(n - 2);
}

createServer((req, res) => {
  if (req.url === '/fib') {
    const result = slowFib(43); // hogs the ONLY thread for ~2s...
    res.end(String(result));    // ...and EVERY other request waits behind it
  } else {
    res.end('fast'); // during a /fib call, even this becomes unresponsive
  }
}).listen(3000);
```

While `slowFib(43)` runs, the single thread is pegged — `/fast` requests, health checks, everything queues up. The event loop is *blocked*. Fixes come in section 5 and chapter 15.

---

## 5. Apply It to Habex

Good news: **almost everything Habex does is I/O-bound**, which is precisely Node's sweet spot. Look at a typical request and where the time goes:

| Habex operation | Bound by | Node fit |
|---|---|---|
| `GET /transactions` → query PostgreSQL | **I/O** (waiting on the DB) | Ideal — thread is free while the DB works |
| Login → `bcrypt.compare` password | **CPU** (hashing is deliberately slow) | Handled off-thread by libuv's crypto pool — safe |
| Send "budget exceeded" email via an API | **I/O** (waiting on the mail service) | Ideal |
| Cache read/write via Redis | **I/O** | Ideal |
| Generate a full-year PDF/CSV spending report | **CPU + memory** (aggregating, formatting) | Danger — must not run inline |

The dangerous one is the **report**. Imagine a Habex endpoint that crunches a year of `transactions` into a summary:

```ts
// habex — the WRONG way: heavy work on the request thread
app.get('/reports/annual', async (req, res) => {
  const rows = await db.query('SELECT * FROM transactions WHERE user_id = $1', [userId]);
  const pdf = renderHeavyPdf(rows); // CPU-bound: blocks the event loop for seconds
  res.setHeader('Content-Type', 'application/pdf').send(pdf);
  // While this runs, EVERY other Habex user's request is frozen.
});
```

Two correct ways to handle CPU work in Node, both of which keep the event loop free:

**Option A — offload to a background queue (the production answer for Habex).** The request returns *immediately* with "report is generating"; a separate **worker** process builds it and notifies the user when done. This is exactly what BullMQ + Redis give us in [15-background-jobs-and-queues.md](15-background-jobs-and-queues.md):

```ts
// habex — the RIGHT way: enqueue the heavy job, respond fast (forward-ref: ch15)
import { reportQueue } from './queues/report.queue.js';

app.post('/reports/annual', async (req, res) => {
  // Just push a job onto the queue (fast, I/O-bound) and return.
  const job = await reportQueue.add('annual', { userId: req.user.id, year: 2026 });
  res.status(202).json({ jobId: job.id, status: 'processing' }); // 202 = Accepted
});
// A separate worker process picks up the job and does the CPU work OFF the API's event loop.
```

> **Money correctness note:** every `amount_cents` in those aggregations is an **integer number of cents** — you `sum` integers, never floats. Floats would let rounding errors accumulate across thousands of rows and produce a wrong total. (See the track README.)

**Option B — `worker_threads`** for CPU work you want to keep *in-process* (e.g. a heavy compute you can't easily externalize). It runs the function on a real OS thread so the main event loop stays free. For Habex, a queue is the better default (it also survives restarts and scales independently), so we prefer Option A.

### Scaling past one core

Node uses one core per process. Your server has 4–8. To use them all, run **multiple Node instances** behind a load balancer, or use the built-in **`cluster`** module to fork one worker per core (all sharing the same port):

```ts
// cluster.ts — run one Node worker per CPU core, all serving port 3000
import cluster from 'node:cluster';
import { availableParallelism } from 'node:os';
import process from 'node:process';

if (cluster.isPrimary) {
  const cores = availableParallelism(); // e.g. 8
  console.log(`Primary ${process.pid} starting ${cores} workers`);
  for (let i = 0; i < cores; i++) cluster.fork(); // each fork = a full Node process
  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} died — restarting`);
    cluster.fork(); // self-heal
  });
} else {
  startHabexServer(); // your normal Express app; the OS load-balances connections
}
```

In production you often let the platform do this (PM2, Docker replicas, or Railway/Render scaling — [20-deployment-and-devops.md](20-deployment-and-devops.md)) rather than hand-rolling `cluster`. Either way it's **horizontal scaling** — more processes/machines, not a bigger single thread. The concept-level treatment is [../system-design/06-scaling-vertical-vs-horizontal.md](../system-design/06-scaling-vertical-vs-horizontal.md).

---

## 6. Interview Questions (with answer pointers)

1. **"Explain the Node event loop."** *(every Node interview)*
   → JS runs single-threaded on V8. Async I/O is handed to libuv (OS async or a thread pool). When the call stack is empty, the event loop takes ready callbacks and runs them, draining `process.nextTick`, then microtasks, then one macrotask per iteration, cycling through phases (timers → poll → check → close). Mention it's *why* one thread serves many connections.

2. **"What's the output of `console.log` + `setTimeout(…,0)` + `Promise.then` + `nextTick`, and why?"** *(Zepto, CRED)*
   → Sync logs first; then `process.nextTick`; then Promise microtasks; then the `setTimeout` macrotask. Because Node fully drains nextTick + microtasks before each macrotask. (See 4b.)

3. **"Microtask vs macrotask?"**
   → Microtasks = Promise callbacks / `queueMicrotask` (and separately `process.nextTick`), drained *completely* after each task and before the next macrotask. Macrotasks = timers, I/O callbacks, `setImmediate`, one per loop turn. Microtasks have priority.

4. **"Node is single-threaded — how does it handle concurrent requests?"** *(Swiggy, Groww)*
   → *JS execution* is single-threaded, but I/O is non-blocking: requests waiting on the DB/network don't occupy the thread. libuv (OS mechanisms + a thread pool) does the waiting; the event loop multiplexes thousands of in-flight requests on one thread.

5. **"When is Node a bad choice?"** *(system-design flavored)*
   → CPU-bound workloads (video encoding, big data crunching, heavy ML inference) — they block the single thread. Mitigate with `worker_threads`, a job queue, or a different runtime for that service. Node shines for I/O-bound APIs.

6. **"How do you handle CPU-heavy work without freezing the server?"** *(CRED, Razorpay)*
   → Offload it: a background queue (BullMQ) processed by a separate worker, `worker_threads`, or a dedicated microservice. Keep the request path free of CPU-bound and `*Sync` calls.

7. **"Is `process.nextTick` a microtask? How's it different from `Promise.then`?"**
   → It has its own queue that Node drains *before* the Promise microtask queue. Overusing it can starve the loop (I/O never gets a turn), so prefer `queueMicrotask`/Promises unless you specifically need nextTick semantics.

8. **"`setTimeout(fn, 0)` vs `setImmediate(fn)` — which runs first?"** *(tricky)*
   → It's order-dependent at the top level, but inside an I/O callback `setImmediate` runs first (it fires in the *check* phase right after *poll*, whereas the timer must wait for the next *timers* phase). Good signal you actually understand the phases.

9. **"Does the thread pool handle network I/O?"** *(deep cut)*
   → No — sockets use the OS's async network APIs (epoll/kqueue/IOCP). The libuv thread pool (default size 4, `UV_THREADPOOL_SIZE`) handles `fs`, `dns.lookup`, `crypto`, and `zlib`. A common source of surprising latency under load.

---

## 7. Trade-offs & Pitfalls ("Why Not Just X?")

- **"Why not just use threads like Java/Go?"** Node trades multi-threading for a simpler, lock-free single-threaded model that's extremely efficient for I/O-bound web workloads (the common case). You give up easy CPU parallelism (recovered via `worker_threads`/queues/multiple processes). It's a deliberate fit for APIs, not number-crunching.

- **Pitfall: a single `*Sync` or heavy loop on the request path.** It blocks *all* users, not just the caller. Watch for `readFileSync`, `JSON.parse` on huge payloads, synchronous `bcrypt`/`crypto`, and giant `for` loops. Move them off-thread.

- **Pitfall: assuming `async` = parallel.** `await` doesn't spawn a thread; it *yields* so the single thread can do other work while the I/O completes. Two independent awaits can overlap with `Promise.all`, but your JS still runs one statement at a time.

- **Pitfall: starving the event loop with `process.nextTick`.** Recursive `nextTick` can prevent the loop from ever reaching the I/O/timers phases. Prefer `setImmediate`/`queueMicrotask` unless you have a specific reason.

- **Pitfall: unhandled Promise rejections.** In modern Node an unhandled rejection can crash the process. Always `try/catch` around `await`, or attach error handling — critical on a long-running server.

- **Pitfall: forgetting Node uses one core.** A single process won't use your 8-core box. Scale with `cluster`/replicas ([20-deployment-and-devops.md](20-deployment-and-devops.md), [../system-design/06-scaling-vertical-vs-horizontal.md](../system-design/06-scaling-vertical-vs-horizontal.md)).

---

## 8. TL;DR

- **Node = V8 (runs JS) + libuv (async I/O + a small thread pool).** Your JavaScript runs on a **single thread**.
- The **event loop** keeps that one thread busy by running callbacks for finished I/O instead of idle-waiting. That's how one thread serves thousands of concurrent requests.
- **Ordering:** synchronous code → `process.nextTick` → microtasks (Promises) → one macrotask (timers/I/O/`setImmediate`). The microtask queue is fully drained between macrotasks. Know the classic snippet cold.
- **Non-blocking wins:** never use `*Sync` or heavy CPU work on the request path — it blocks *every* user. Use async APIs.
- Node is **great for I/O-bound** work (APIs, DB, network — all of Habex's request path) and **bad for CPU-bound** work; offload heavy compute to `worker_threads` or a **queue** ([15-background-jobs-and-queues.md](15-background-jobs-and-queues.md)).
- One process = one core. Scale with **`cluster`/multiple instances** ([../system-design/06-scaling-vertical-vs-horizontal.md](../system-design/06-scaling-vertical-vs-horizontal.md)).

**Next → [03-project-setup-typescript-node.md](03-project-setup-typescript-node.md):** scaffold the real `habex-api` project — package.json, tsconfig, ESM, env config — that every remaining chapter builds on.
