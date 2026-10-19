# 01. Big-O & the Problem-Solving Framework

> **One-liner:** After this you can look at any piece of code and say *how fast it is* and *how much memory it uses* — and you'll have a repeatable, say-out-loud method to attack any interview problem without panicking.
> **Pattern power:** This is the foundation under *every* other chapter. Big-O is the language interviewers grade you in, and the UMPIRE framework is the script you'll run on all ~150 problems in this track.

Take a breath. If DSA has felt like a wall of scary words, this chapter is the door. You already reason about performance every day — "don't re-render the whole list", "memoize that expensive selector". DSA is the same instinct, just made precise. Let's make it precise.

---

## 1. The Idea in Plain Words

An **algorithm** is just a recipe: a finite list of steps that turns an input into an output. `Array.prototype.map` is an algorithm. Your Habex function that totals this month's expenses is an algorithm. There's nothing mystical here.

Two algorithms can produce the *exact same answer* but take wildly different amounts of work. Interviewers don't just ask "does it work?" — they ask **"how does it behave as the input grows?"** That's what **Big-O notation** measures.

### The n=10 vs n=1,000,000 story

Say Habex needs to find whether a user has any **duplicate transaction** (same amount, same second — a double-charge).

**Approach A — compare every pair:** for each transaction, compare it against every other one. If there are `n` transactions, that's about `n × n = n²` comparisons.

**Approach B — remember what you've seen:** walk the list once, dropping each transaction into a `Set`. Before adding, check if it's already there. That's about `n` steps (a `Set` lookup is effectively instant — more on that in [03-hashing-maps-and-sets.md](03-hashing-maps-and-sets.md)).

Now watch what happens as the user's history grows:

| Transactions (`n`) | Approach A (`n²`) | Approach B (`n`) | Real-world feel |
|---|---|---|---|
| 10 | 100 | 10 | Both instant. Nobody cares. |
| 1,000 | 1,000,000 | 1,000 | Both fine. |
| 1,000,000 | **1,000,000,000,000** (a trillion) | 1,000,000 | A: your tab **freezes for minutes**. B: **milliseconds**. |

Same feature, same result — the difference between "smooth" and "the app hung and the user churned." At the scale Zepto, Swiggy, or Razorpay operate (millions of orders/payments), this *is* the job. That's why the interview gate exists: it's a proxy for "can you write code that survives real traffic?"

The beautiful part: **at n=10 you literally cannot tell the difference.** Efficiency is a story about *growth*, not about small inputs. Big-O is how we talk about growth.

---

## 2. The Complexity Cheat Sheet

**Big-O describes how the work grows as the input grows** — and we deliberately ignore the small stuff:

- **Drop constants:** `O(2n)` → `O(n)`. Doubling the work still grows linearly.
- **Drop lower-order terms:** `O(n² + n)` → `O(n²)`. For large `n`, the `n²` dwarfs the `n`.
- We care about the **worst case** unless told otherwise (interviewers assume worst case).

So Big-O answers one question: *"When I 10× the input, roughly how much more work?"*

### The common classes (with an everyday feel + runnable TS)

**O(1) — constant.** Same work no matter the size. *Everyday:* grabbing the top card off a deck.
```ts
function first<T>(arr: T[]): T | undefined {
  return arr[0]; // one step, whether arr has 5 items or 5 million
}
```

**O(log n) — logarithmic.** You *halve* the problem each step. Blazing fast: a million items takes ~20 steps. *Everyday:* "higher/lower" — guess a number 1–100 by always guessing the middle.
```ts
// Binary search — full chapter: 07-binary-search.md
function contains(sorted: number[], target: number): boolean {
  let lo = 0, hi = sorted.length - 1;
  while (lo <= hi) {
    const mid = Math.floor((lo + hi) / 2); // look at the middle
    if (sorted[mid] === target) return true;
    if (sorted[mid] < target) lo = mid + 1; // throw away the left half
    else hi = mid - 1;                       // throw away the right half
  }
  return false;
}
```

**O(n) — linear.** Touch each item a constant number of times. *Everyday:* reading every page of a book once.
```ts
function sum(arr: number[]): number {
  let total = 0;
  for (const x of arr) total += x; // one pass
  return total;
}
```

**O(n log n) — "linearithmic."** The speed of good sorting, and the practical target for most "process everything, but smartly" problems. *Everyday:* properly sorting a shuffled deck.
```ts
// JS/TS TRAP: sort() is lexicographic by default!
// [3, 20, 100].sort()  ->  [100, 20, 3]   (compares as strings)
const sorted = [...arr].sort((a, b) => a - b); // always pass a comparator for numbers
```

**O(n²) — quadratic.** Every pair. A nested loop over the same data. Fine for small `n`, deadly for large. *Everyday:* everyone in a room shaking hands with everyone else.
```ts
function hasDuplicatePair(arr: number[]): boolean {
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) { // for each i, loop the rest
      if (arr[i] === arr[j]) return true;
    }
  }
  return false;
}
```

**O(2ⁿ) — exponential.** Try every yes/no combination. *Everyday:* every on/off setting of `n` light switches. Only viable for tiny `n` (~≤ 20).
```ts
// Every subset of n items — doubles with each new item
function subsets<T>(arr: T[]): T[][] {
  if (arr.length === 0) return [[]];
  const [head, ...rest] = arr;
  const without = subsets(rest);
  return [...without, ...without.map(s => [head, ...s])];
}
```

**O(n!) — factorial.** Every ordering. *Everyday:* every possible seating arrangement of `n` guests. Explodes almost immediately (10! = 3.6 million). Shows up in permutation/brute-force-travel problems.
```ts
function permutations<T>(arr: T[]): T[][] {
  if (arr.length <= 1) return [arr];
  const out: T[][] = [];
  for (let i = 0; i < arr.length; i++) {
    const rest = [...arr.slice(0, i), ...arr.slice(i + 1)];
    for (const p of permutations(rest)) out.push([arr[i], ...p]);
  }
  return out;
}
```

### How to *read* the Big-O of code

You don't compute this with math — you eyeball the structure:

- **A single loop over `n`** → `O(n)`.
- **A loop nested inside another**, both over `n` → multiply → `O(n²)`. (Loops over *different* inputs `m` and `n` → `O(m·n)`.)
- **Two loops one after another** (sequential) → *add* → `O(n) + O(n) = O(n)`.
- **A loop that halves/doubles** its counter (`i *= 2`) → `O(log n)`.
- **A sort anywhere** → at least `O(n log n)`.
- **Recursion** → *(number of calls) × (work per call)*. Naive Fibonacci makes ~`2ⁿ` calls → `O(2ⁿ)`. A recursion that halves the input → `O(log n)`.
- **Constants and fixed inner loops vanish:** a loop that always runs exactly 100 times inside a loop over `n` is still `O(n)`, not `O(100n)`.

### Time vs Space complexity

- **Time** = how many *steps*.
- **Space** = how much *extra memory* beyond the input you allocate. A `Map` of `n` items is `O(n)` extra space. Swapping elements in place is `O(1)` extra space. (The recursion call stack counts too — depth `n` recursion is `O(n)` space.)

Interviewers ask for *both*. A classic trade is **spend memory to save time** — the whole point of hashing (see [03](03-hashing-maps-and-sets.md)).

### Estimating feasibility from the input size (a superpower)

Interview problems tell you the constraints (e.g., "1 ≤ n ≤ 10⁵"). A CPU does very roughly **10⁸ simple operations per second**. Work backward from `n` to the complexity you're *allowed* to have:

| If `n` is up to… | Your target | Because… |
|---|---|---|
| ~10–12 | `O(n!)` | factorial is tiny here only |
| ~20–25 | `O(2ⁿ)` | ~1M–33M subsets — fine |
| ~500 | `O(n³)` | 1.25×10⁸ — borderline OK |
| ~5,000 | `O(n²)` | 2.5×10⁷ — comfortable |
| ~10⁵ (100k) | `O(n log n)` | ~1.7×10⁶ — `O(n²)` would be 10¹⁰ = too slow |
| ~10⁶ (1M) | `O(n)` or `O(n log n)` | single pass / one sort |
| ~10⁹ or "huge" | `O(log n)` or `O(1)` | binary search or a formula |

**Read this backward in the interview:** "`n` can be 10⁵, so `O(n²)` is out — I need `O(n log n)` or better." That single sentence tells the interviewer you think like an engineer.

---

## 3. Picture It (Diagram)

How the classes pull apart as `n` grows — the whole reason Big-O matters:

```
 operations
   ^
   |                              O(2^n)    O(n!)   <- explode; unusable fast
   |                          .  :
   |                        .   :
   |             O(n^2)   .    :
   |                   . .   :
   |                .  .    :        _______ O(n log n)
   |            .  .    :  _____-----
   |         . .   : __--            _________________ O(n)
   |      . .  :__-       __________/
   |   . . :_-   ________/                  ............ O(log n)
   | .:_--_____/............................................
   |*==================================================== O(1)
   +-----------------------------------------------------------> n
```

And in hard numbers — the operation counts that make it click:

| `n` | O(1) | O(log n) | O(n) | O(n log n) | O(n²) | O(2ⁿ) |
|---|---|---|---|---|---|---|
| 10 | 1 | ~3 | 10 | ~33 | 100 | 1,024 |
| 100 | 1 | ~7 | 100 | ~664 | 10,000 | ~10³⁰ |
| 1,000 | 1 | ~10 | 1,000 | ~9,966 | 1,000,000 | 🚫 astronomical |
| 1,000,000 | 1 | ~20 | 10⁶ | ~2×10⁷ | 10¹² | 🚫 |

Notice O(log n) barely moves and O(1) is a flat line, while O(2ⁿ) is unusable by n=100. **Your job in interviews is to slide problems leftward** — from red columns toward green ones.

---

## 4. The UMPIRE Problem-Solving Framework

The single biggest reason strong coders freeze in interviews: they jump straight to typing. Don't. Run this 6-step script **out loud** every time. It's called **UMPIRE**.

### U — Understand (ask clarifying questions FIRST)

Restate the problem in your words, then interrogate it. Ask these almost every time:

- **"What are the input types and sizes?"** — the constraint on `n` tells you your target complexity (see the table above).
- **"Can the input be empty? Have one element? Have duplicates? Negatives? Already sorted?"**
- **"Is a valid answer guaranteed, or can there be none? What do I return then?"** (`[]`? `-1`? `null`?)
- **"Can there be multiple answers — return any, or all?"**
- **"May I mutate the input?"** (affects space complexity)
- **"Return the *indices* or the *values*? The *count* or the actual *list*?"**

Then write **1–2 concrete examples by hand**, and deliberately include one **edge case** (empty, single element, all-duplicates). This catches misunderstandings before they cost you.

### M — Match (which pattern is this?)

Scan the problem for **trigger words** and match to a pattern. This is exactly what the **[Pattern → Trigger map in the README](00-README.md)** is for — keep it open. A few:
- "seen before / count / group / dedupe" → **Hash Map/Set** ([03](03-hashing-maps-and-sets.md))
- "sorted array / find a pair / two ends" → **Two Pointers** ([04](04-two-pointers.md))
- "longest/shortest contiguous substring/subarray" → **Sliding Window** ([05](05-sliding-window.md))
- "sorted + find X fast" → **Binary Search** ([07](07-binary-search.md))

Most problems are a variation of something you've drilled. Matching the pattern *is* 80% of the solve.

### P — Plan (brute force first, THEN optimize)

Describe your approach in plain English / pseudocode **before** writing real code. Crucial habit: **state a brute-force solution first**, name its complexity, then find the bottleneck and improve it.

> "The brute force is to check every pair — that's `O(n²)`. The slow part is re-searching the array each time. If I remember what I've seen in a hash map, each lookup becomes `O(1)`, so I can get to `O(n)`."

Interviewers *love* this. It shows you can always produce a working answer, and that you can reason about *why* the fast version is faster. A brute force you can state beats an optimal one you can't.

### I — Implement

Now translate the plan into clean TypeScript. Use meaningful names, typed signatures, and handle the edge cases you identified in step U. Narrate as you go ("I'll use a `Map` from value to index...").

### R — Review / Test

Trace your code **by hand** on your examples, especially the edge case. Hunt the usual suspects: off-by-one errors, empty input, a single element, all-identical elements, the no-answer case. Fix bugs *before* the interviewer points them out.

### E — Evaluate

State the **final time and space complexity** and confirm it meets the constraints. Mention trade-offs: "This is `O(n)` time and `O(n)` space; if space were tight, I'd sort and use two pointers for `O(n log n)` time but `O(1)` space."

> **Print this on a sticky note:** Understand → Match → Plan → Implement → Review → Evaluate.

---

## 5. Complexity of Common Operations (memorize this table)

The exact costs of the JS/TS operations you'll reach for constantly. Several of these are **interview traps** because they *look* cheap but aren't (full detail in [02-arrays-and-strings.md](02-arrays-and-strings.md) and [03-hashing-maps-and-sets.md](03-hashing-maps-and-sets.md)):

| Operation | Big-O | Note / trap |
|---|---|---|
| `arr[i]` (index access) | O(1) | direct addressing |
| `arr.push` / `arr.pop` (end) | O(1) amortized | occasional resize averages out |
| `arr.shift` / `arr.unshift` (front) | **O(n)** | ⚠️ re-indexes every element — avoid in loops |
| `arr.splice(i, ...)` | **O(n)** | ⚠️ shifts everything after `i` |
| `arr.includes` / `indexOf` / `find` | **O(n)** | ⚠️ linear scan — a `Set` is O(1) |
| `arr.slice` / `concat` / `[...arr]` | O(n) | ⚠️ copies — hidden cost inside a loop |
| `arr.sort((a,b)=>a-b)` | O(n log n) | ⚠️ pass a comparator for numbers! |
| `map.get` / `set` / `has` / `delete` | O(1) average | O(n) worst case (rare collisions) |
| `set.has` / `add` / `delete` | O(1) average | the fast membership test |
| `str[i]` / `str.charCodeAt(i)` | O(1) | strings are indexable |
| building a string with `+=` in a loop | **O(n²)** | ⚠️ strings are immutable — use an array + `.join('')` |

---

## 6. Worked Example — Two Sum, via the full framework

> **LeetCode 1 · Two Sum (Easy).** Given `nums: number[]` and a `target: number`, return the **indices** of the two numbers that add up to `target`. Exactly one solution exists; you may not use the same element twice.

Let's run **UMPIRE** exactly as you would in the room.

**U — Understand.** Clarifying questions and their answers: *One solution guaranteed?* Yes. *Same element twice?* No. *Sorted?* No. *Return indices or values?* Indices. *Negatives allowed?* Yes.
Examples I'll write down:
- `nums = [2, 7, 11, 15]`, `target = 9` → `[0, 1]` (because 2 + 7 = 9)
- Edge case: `nums = [3, 3]`, `target = 6` → `[0, 1]` (duplicates — I must not reuse one `3` twice)

**M — Match.** "Find a pair that sums to target" and "have I seen the number I need?" → that's the **hash map / seen-before** trigger from the [README map](00-README.md). (Two pointers also solves pair-sum, but that needs a *sorted* array and here I must preserve original indices — so hashing fits better.)

**P — Plan.**
- *Brute force:* check every pair `(i, j)` → `O(n²)` time, `O(1)` space. Works, but too slow if `n` were large.
- *Optimize:* for each number `x`, the partner I need is `target − x` (its "complement"). If I keep a `Map` of every value I've already seen → its index, I can look up the complement in `O(1)`. One pass, `O(n)`.

**I — Implement.**
```ts
function twoSum(nums: number[], target: number): number[] {
  // Map from a value we've already seen -> the index it appeared at
  const seen = new Map<number, number>();

  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i]; // the partner that would complete the pair

    // Have we already passed the partner? If so, we're done.
    if (seen.has(complement)) {
      return [seen.get(complement)!, i]; // "!" : we just checked has(), so it's defined
    }

    // Otherwise, remember the current number for future lookups.
    // Note: we check BEFORE we store, so we can never pair an element with itself.
    seen.set(nums[i], i);
  }

  return []; // unreachable given the "exactly one solution" guarantee
}
```

**R — Review / Test.** Trace `nums = [2, 7, 11, 15]`, `target = 9`:

| `i` | `nums[i]` | `complement = 9 − nums[i]` | `seen` before | in `seen`? | action |
|---|---|---|---|---|---|
| 0 | 2 | 7 | `{}` | no | store `2 → 0` |
| 1 | 7 | 2 | `{2:0}` | **yes** | return `[0, 1]` ✅ |

Now the tricky edge `nums = [3, 3]`, `target = 6`: at `i=0`, complement `3` isn't in `seen` yet → store `3 → 0`. At `i=1`, complement `3` **is** in `seen` (index 0) → return `[0, 1]`. Because we *check before storing*, the two different `3`s are correctly treated as two different elements. ✅

**E — Evaluate.** **Time `O(n)`** — one pass, each `Map` operation is O(1) average. **Space `O(n)`** — the map can hold up to `n` entries. This is the textbook "spend memory to save time" win: we traded `O(1)`→`O(n)` space to cut `O(n²)`→`O(n)` time. For any realistic `n`, that's the right call.

---

## 7. Practice Set — "What's the Big-O?"

Foundations practice isn't LeetCode problems yet — it's **reading complexity fluently**, the skill you'll use in every future chapter. Cover the answer column, work out each, then check. Time *and* space.

```ts
// Snippet A
function a(arr: number[]): number {
  let s = 0;
  for (const x of arr) s += x;
  return s;
}

// Snippet B
function b(arr: number[]): number {
  let count = 0;
  for (let i = 0; i < arr.length; i++)
    for (let j = i + 1; j < arr.length; j++)
      count++;
  return count;
}

// Snippet C
function c(n: number): number {
  let steps = 0;
  for (let i = 1; i < n; i *= 2) steps++;
  return steps;
}

// Snippet D
function d(arr: number[]): number[] {
  const doubled = arr.map(x => x * 2);   // pass 1
  const evens = doubled.filter(x => x % 4 === 0); // pass 2
  return evens;
}

// Snippet E — fib the naive way
function fib(n: number): number {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
}

// Snippet F
function f(words: string[]): Map<string, number> {
  const freq = new Map<string, number>();
  for (const w of words) freq.set(w, (freq.get(w) ?? 0) + 1);
  return freq;
}
```

| # | Time | Space | Why |
|---|---|---|---|
| A | O(n) | O(1) | one pass, a single accumulator |
| B | O(n²) | O(1) | triangular nested loop = n(n−1)/2 ≈ n²; constants drop |
| C | O(log n) | O(1) | `i` doubles each step → ~log₂n iterations |
| D | O(n) | O(n) | two *sequential* passes add (O(2n)→O(n)); builds new arrays |
| E | O(2ⁿ) | O(n) | each call spawns two calls; space = recursion depth `n` |
| F | O(n) | O(n) | one pass; map can hold up to `n` distinct words |

If B tempted you to say `O(n² / 2)` — right instinct, but we **drop the constant**: it's `O(n²)`.

---

## 8. Common Pitfalls & Interview Tips

- **Premature optimization.** Don't chase the clever `O(n)` while you have nothing on the board. State the brute force, get a nod, *then* optimize. A working `O(n²)` beats a broken `O(n)` every time.
- **Not testing edge cases.** Empty array, single element, all-duplicates, no-valid-answer, negatives. Interviewers deliberately feed these. Trace them in step R.
- **Forgetting space complexity.** Many candidates only state time. Always say both — and mention the time/space trade-off you chose.
- **The `sort()` trap.** `[10, 9, 1].sort()` → `[1, 10, 9]` because default sort is lexicographic (string) order. Always `sort((a, b) => a - b)` for numbers. This bug has failed real interviews.
- **Confusing average vs worst case.** Hash `Map`/`Set` ops are `O(1)` *average* but `O(n)` worst case under collisions. Say "O(1) average" to sound precise; adversarial collisions almost never matter in interviews.
- **Thinking constants matter in Big-O** (they don't: `O(2n) = O(n)`) — but remembering they matter **in the real world** (an `O(n)` with a huge constant can lose to an `O(n log n)`). Both truths coexist.
- **What to say out loud:** narrate constraints → "`n` up to 10⁵, so I need O(n log n) or better" → brute force + its Big-O → the bottleneck → the optimization → final complexity. That narration is literally what you're scored on. Silence reads as "stuck."

---

## 9. TL;DR

- An **algorithm** is a recipe; **Big-O** measures how its work grows as input grows. Drop constants and lower-order terms; assume worst case.
- Know the ladder cold: **O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ) < O(n!)**.
- **Read structure, not math:** one loop = O(n), nested = O(n²), halving = O(log n), sort = O(n log n), recursion = (calls × work).
- **Estimate from constraints:** `n ≤ 10⁵` ⇒ aim O(n log n); `n ≤ 5,000` ⇒ O(n²) is fine. CPU ≈ 10⁸ ops/sec.
- Track **time *and* space**; the classic win is **spend memory to save time** (hashing).
- Attack every problem with **UMPIRE**: **U**nderstand (clarify) → **M**atch (pattern) → **P**lan (brute force first) → **I**mplement → **R**eview/test → **E**valuate (complexity).
- **Next:** the workhorses these patterns run on → [02-arrays-and-strings.md](02-arrays-and-strings.md), then the single most-used interview tool → [03-hashing-maps-and-sets.md](03-hashing-maps-and-sets.md).
