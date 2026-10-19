# 🧩 DSA — Cracking Coding Interviews in TypeScript

> **Goal:** clear the **DSA interview gate** for product companies (Zepto, Swiggy, CRED, Razorpay, Groww, FAANG) — the one thing that filters candidates out *before* they ever reach system design.
> **For:** a React/TypeScript dev who is new to DSA and problem-solving. Zero prior DSA assumed.
> **Part of** your [Skill-Up Hub](../README.md).

---

## The one idea that makes DSA learnable: **patterns, not problems**

There are ~2,000+ LeetCode problems. You will not memorize them. You don't need to. **~15 patterns cover the vast majority of interview questions.** Once you can *recognize the trigger* in a problem and *recall the template*, most problems become variations of something you've already seen.

This track teaches you to do exactly that: for each pattern → **when to use it (triggers)** → **a reusable code template** → **a worked example** → **a curated practice set**.

---

## Why TypeScript here (your edge)

Almost every DSA resource uses Python/Java/C++. This one uses **TypeScript** on purpose:

- It's **your** language — no fighting syntax while also learning algorithms.
- **These companies let you code in JS/TS** in the interview. Use your strength.
- We explicitly flag the **JS/TS interview traps** other guides skip, e.g.:
  - `[3, 20, 100].sort()` returns `[100, 20, 3]` — `sort()` is lexicographic by default. Always pass a comparator: `.sort((a, b) => a - b)`.
  - **JavaScript has no built-in heap / priority queue.** You must know a tiny implementation cold (covered in [14-heaps-and-priority-queues.md](14-heaps-and-priority-queues.md)).
  - `Map` preserves insertion order and takes any key type; a plain object doesn't — prefer `Map`/`Set` for DSA.
  - Numbers are 64-bit floats — integer overflow is rarely your problem, but precision (`0.1 + 0.2`) and `Number.MAX_SAFE_INTEGER` can be.

---

## Pattern → Trigger quick map (your interview cheat sheet)

When you read a problem, scan for these signals:

| If the problem says… | Reach for | Chapter |
|---|---|---|
| "sorted array", "find a pair/triplet", "two ends" | **Two Pointers** | [04](04-two-pointers.md) |
| "longest/shortest substring/subarray with condition", "contiguous" | **Sliding Window** | [05](05-sliding-window.md) |
| "sum of range", "how many subarrays sum to K" | **Prefix Sum** | [06](06-prefix-sum.md) |
| "sorted" + "find X" in O(log n), "minimum value that works" | **Binary Search** | [07](07-binary-search.md) |
| "seen before?", "count of", "group by", "dedupe" | **Hash Map / Set** | [03](03-hashing-maps-and-sets.md) |
| "linked list", "detect cycle", "middle node" | **Fast & Slow Pointers** | [09](09-linked-lists.md) |
| "matching pairs", "next greater/smaller", "valid parentheses" | **Stack / Monotonic Stack** | [10](10-stacks-and-queues.md) |
| "all combinations/permutations/subsets", "generate every…" | **Backtracking** | [11](11-recursion-and-backtracking.md) |
| "tree", "level order", "depth", "path" | **Tree BFS/DFS** | [12](12-trees.md) |
| "top K", "K largest/closest/frequent", "median of stream" | **Heap** | [14](14-heaps-and-priority-queues.md) |
| "grid", "islands", "connected", "shortest path (unweighted)" | **Graph BFS/DFS** | [16](16-graphs-bfs-dfs.md) |
| "prerequisites", "ordering", "cycle in directed graph" | **Topological Sort** | [17](17-graphs-advanced.md) |
| "groups", "are these connected", "number of components" | **Union-Find** | [17](17-graphs-advanced.md) |
| "max/min ways", "can you reach", "optimal choices, overlapping subproblems" | **Dynamic Programming** | [18](18-dynamic-programming-1.md), [19](19-dynamic-programming-2.md) |
| "merge/insert intervals", "meeting rooms", "non-overlapping" | **Intervals / Greedy** | [20](20-greedy-and-intervals.md) |

---

## The curriculum

### Part I — Foundations
- [01 — Big-O & the Problem-Solving Framework](01-big-o-and-problem-solving-framework.md) — how to measure code, and a repeatable method to attack any problem
- [02 — Arrays & Strings](02-arrays-and-strings.md) — the workhorses, and their TS gotchas
- [03 — Hashing: Maps & Sets](03-hashing-maps-and-sets.md) — trade space for time (the most-used interview tool)

### Part II — Core Patterns
- [04 — Two Pointers](04-two-pointers.md)
- [05 — Sliding Window](05-sliding-window.md)
- [06 — Prefix Sum](06-prefix-sum.md)
- [07 — Binary Search](07-binary-search.md) — on arrays *and* on the answer space
- [08 — Sorting](08-sorting.md) — the algorithms + when sorting is the whole trick

### Part III — Linear Data Structures
- [09 — Linked Lists](09-linked-lists.md) — + fast & slow pointers / cycle detection
- [10 — Stacks & Queues](10-stacks-and-queues.md) — + the monotonic stack

### Part IV — Recursion, Trees & Heaps
- [11 — Recursion & Backtracking](11-recursion-and-backtracking.md)
- [12 — Trees & Binary Trees](12-trees.md) — traversals, BFS/DFS
- [13 — Binary Search Trees](13-binary-search-trees.md)
- [14 — Heaps & Priority Queues](14-heaps-and-priority-queues.md) — + the top-K pattern (and JS has no built-in!)
- [15 — Tries (Prefix Trees)](15-tries.md)

### Part V — Graphs
- [16 — Graphs: BFS & DFS](16-graphs-bfs-dfs.md)
- [17 — Graphs: Advanced](17-graphs-advanced.md) — topological sort, union-find, Dijkstra basics

### Part VI — Dynamic Programming & Greedy
- [18 — Dynamic Programming I](18-dynamic-programming-1.md) — the intuition, memo vs tabulation, 1D DP
- [19 — Dynamic Programming II](19-dynamic-programming-2.md) — 2D DP, knapsack, LIS, common templates
- [20 — Greedy & Intervals](20-greedy-and-intervals.md)

### Part VII — Extras & The Plan
- [21 — Bit Manipulation & Math](21-bit-manipulation-and-math.md) — the handful of tricks that actually appear
- [22 — The 150-Problem Plan & Revision System](22-the-150-problem-plan.md) — your curated, week-by-week roadmap

---

## How to study (so it sticks)

1. **Read the chapter, then immediately solve its worked example yourself** without looking. Struggle first — that's where learning happens.
2. **Do the practice set**, easy → hard. Aim to *recognize the pattern* before coding.
3. **Spaced repetition:** re-solve a problem 1 day later, then 1 week later. Re-solving beats reading.
4. **Volume that compounds:** even **1 problem/day = ~250/year**. The [150-problem plan](22-the-150-problem-plan.md) sequences them for you.
5. **Talk out loud** as you solve (clarify → examples → brute force → optimize → code → test). Interviews score your *communication*, not just the answer.

> **Target:** ~50 problems to be dangerous, ~150 to be interview-ready, ~200+ to be comfortable. Quality (understanding patterns) beats blind volume.

---

## What each company tends to ask

| Company | DSA flavor |
|---|---|
| **Zepto / Swiggy** | Arrays, strings, hashing, sliding window, heaps; practical "high-scale" framing |
| **CRED / Razorpay** | Solid mediums: arrays, DP, graphs, intervals; clean code + edge cases matter (fintech = correctness) |
| **Groww** | Arrays, hashing, DP, trees; consistency and clarity valued |
| **FAANG (stretch)** | Full spread incl. graphs + hard DP; optimal complexity expected |

---

**Start here → [01 — Big-O & the Problem-Solving Framework](01-big-o-and-problem-solving-framework.md).** Then work top to bottom, keeping a little daily volume going via the [150-problem plan](22-the-150-problem-plan.md).
