# 07. Binary Search

> **One-liner:** Repeatedly halve the search space so you find your answer in **O(log n)** instead of O(n) — on a sorted array, *or* on a hidden "space of possible answers."
> **Pattern power:** The difference between scanning 1,000,000,000 items and touching only ~30 of them. It also unlocks a killer interview trick — **binary search on the answer** — that turns many "find the minimum/maximum value that works" problems from hard into routine.

Prereqs that pair with this chapter: [04 — Two Pointers](04-two-pointers.md) (the other "sorted array" workhorse) and [08 — Sorting](08-sorting.md) (how the array got sorted in the first place).

---

## 1. The Idea in Plain Words

You've played "guess my number between 1 and 100." You don't start at 1 and count up — you guess **50**. Too high? Now you know the answer is 1–49, so you guess **25**. Each guess **throws away half of what's left**. From 100 numbers you're done in ~7 guesses, not 100.

That's binary search. Two conditions make it work:

1. **The data is ordered** (sorted), so from one look at the middle you can tell which half to discard.
2. **The check is monotonic** — once "too small" flips to "big enough," it never flips back. In a sorted array that's automatic; in the advanced version (section 4) *you* supply the monotonic check.

**Why "O(log n)"?** Every step halves the space: n → n/2 → n/4 → … → 1. The number of halvings to get from n down to 1 is `log₂(n)`. Doubling the input adds just **one** more step. That's why binary search laughs at huge inputs.

New term — **monotonic:** a yes/no test that, as you move along the axis, goes `no, no, no, YES, YES, YES` and never back to `no`. That single "flip point" is what binary search hunts for.

---

## 2. When to Use It — Pattern Triggers

| If the problem says / has… | Why binary search fits |
|---|---|
| "**sorted** array" + "find X" / "does X exist" in **O(log n)** | The canonical case — halve the array each step. |
| "find the **first / last** position of X", "insertion point", "count of X" | Leftmost/rightmost boundary = `lowerBound` / `upperBound`. |
| "**minimum** capacity / speed / size **that works**", "**maximum** value such that…", "smallest k so that…" | **Binary search on the answer** — the answer space is monotonic even when the input isn't sorted. |
| "**rotated** sorted array", "mountain array", "peak element" | One half is always still sorted — decide which half to keep. |
| "**k-th smallest / largest**" in a sorted matrix or a value range | Binary search the *value*, count how many are ≤ it. |
| Answer is a number in a known range `[lo, hi]` and a candidate is **easy to verify but hard to construct** | Search the range, verify each guess with a predicate. |

> **Gut check:** whenever you catch yourself thinking *"if I fix the answer to some value X, I can easily check whether X is good enough"* — and "good enough" only gets easier as X grows (or only harder) — that monotonic relationship is a flashing neon sign for binary search.

---

## 3. Picture It (Diagram)

Searching for **11** in a sorted array. `lo`/`hi` bound the live region; `mid` is the midpoint we test.

```
Step 1 — full array. mid = (0+6)/2 = 3.
 index:   0    1    2    3    4    5    6
 value: [ 1    3    5    7    9   11   13 ]
          lo             mid            hi
                          ▲ nums[3] = 7 < 11  →  answer is to the RIGHT
                            discard indices 0..3,  set lo = mid + 1 = 4

Step 2 — live region is [4..6]. mid = 4 + (6-4)/2 = 5.
 index:   ·    ·    ·    ·    4    5    6
 value: [ ·    ·    ·    ·    9   11   13 ]
                             lo   mid   hi
                                   ▲ nums[5] = 11 == target  →  FOUND at index 5
```

Two looks, not seven. Each step the pool of candidates roughly halves: `7 → 3 → 1`.

---

## 4. The Template (TypeScript)

There are two templates. Memorize both; 95% of problems are one of them.

### 4a. Classic exact match — inclusive bounds `[lo, hi]`, loop while `lo <= hi`

Here `hi` is a **real index** that might hold the answer, so the window is *inclusive on both ends* and we must keep looping while `lo <= hi` (test even a 1-element window). Because we fully test `mid`, we move **past** it with `mid ± 1`.

```typescript
// LeetCode 704 — return the index of target, or -1 if absent.
function search(nums: number[], target: number): number {
  let lo = 0;
  let hi = nums.length - 1;          // inclusive: the live window is [lo, hi]

  while (lo <= hi) {                 // <=  because hi is a valid, still-untested index
    // Math.floor is MANDATORY: (lo + hi) / 2 can be a fraction like 2.5,
    // and nums[2.5] is undefined. lo + (hi - lo)/2 == (lo + hi)/2 but never
    // overflows (a habit that matters in Java/C++, and for huge answer-space bounds).
    const mid = lo + Math.floor((hi - lo) / 2);

    if (nums[mid] === target) return mid;      // hit
    else if (nums[mid] < target) lo = mid + 1; // target is to the right of mid
    else hi = mid - 1;                         // target is to the left of mid
  }
  return -1;                          // lo > hi → window empty → not found
}
```

### 4b. Boundary search — half-open bounds `[lo, hi)`, loop while `lo < hi`

To find the **leftmost** index that satisfies a condition (a.k.a. `lower_bound` / `upper_bound`), use a half-open window: `hi` is **one past** the range and is *not* a testable index. The invariant: everything left of `lo` fails, everything at/right of `hi` passes; we squeeze until they meet.

```typescript
// lowerBound: first index i where nums[i] >= target.
// If every element is smaller, returns nums.length (the correct insertion point).
function lowerBound(nums: number[], target: number): number {
  let lo = 0;
  let hi = nums.length;             // EXCLUSIVE upper bound → note it's length, not length-1

  while (lo < hi) {                 // <  (strict): when lo === hi the answer is pinned
    const mid = lo + Math.floor((hi - lo) / 2);
    if (nums[mid] < target) lo = mid + 1; // mid too small → answer is strictly right
    else hi = mid;                        // mid MIGHT be the answer → keep it in range
  }
  return lo;                        // lo === hi === leftmost qualifying index
}

// upperBound: first index i where nums[i] > target. Change >= to >.
function upperBound(nums: number[], target: number): number {
  let lo = 0, hi = nums.length;
  while (lo < hi) {
    const mid = lo + Math.floor((hi - lo) / 2);
    if (nums[mid] <= target) lo = mid + 1;
    else hi = mid;
  }
  return lo;
}

// Bonus you get for free: how many times does target appear?
const countOf = (nums: number[], t: number) => upperBound(nums, t) - lowerBound(nums, t);
```

> **Why `hi = mid` and not `mid - 1` here?** Because `mid` itself could be the boundary answer, so we can't throw it away. And why does the loop still terminate? `lo = mid + 1` always advances, and `hi = mid` always shrinks (since `mid < hi` whenever `lo < hi`). If you *ever* write `lo = mid` in this template, you get an **infinite loop** when `hi = lo + 1`.

### 4c. THE BIG IDEA — binary search **on the answer**

When the array isn't sorted but the *answer is a number in a known range* `[lo, hi]`, and a candidate is **easy to verify** with a monotonic `feasible(x)` (once true, always true for larger x), search the *answer space* itself:

```typescript
// Find the SMALLEST x in [lo, hi] for which feasible(x) is true.
// Requires feasible to be monotonic:  false, false, …, TRUE, TRUE, TRUE.
function minFeasible(
  lo: number,
  hi: number,
  feasible: (x: number) => boolean,
): number {
  while (lo < hi) {
    const mid = lo + Math.floor((hi - lo) / 2);
    if (feasible(mid)) hi = mid;   // mid works → maybe a smaller x also works, keep mid
    else lo = mid + 1;             // mid fails → the answer must be larger
  }
  return lo;                       // smallest feasible value
}
```

That's the exact same skeleton as `lowerBound` — you've just replaced "compare to a neighbor in an array" with "call a predicate." Mastering this one template is worth a dozen medium problems.

---

## 5. Complexity

| Operation | Time | Space | Note |
|---|---|---|---|
| Classic / boundary search | **O(log n)** | O(1) | n = array length |
| Binary search on answer | **O(log(range) × C)** | O(1) | C = cost of one `feasible()` call (often O(n)) |
| Linear scan (the thing you're beating) | O(n) | O(1) | — |

Feel the gap — steps to search different sizes:

| n (items) | Linear scan (worst) | Binary search (worst) |
|---|---|---|
| 10 | 10 | ~4 |
| 1,000 | 1,000 | ~10 |
| 1,000,000 | 1,000,000 | ~20 |
| 1,000,000,000 | 1,000,000,000 | **~30** |

One caveat interviewers love: binary search assumes **O(1) random access**, which arrays give you. On a [linked list](09-linked-lists.md) there's no index jump, so binary search degrades to O(n) and loses its point.

---

## 6. Worked Example (a classic LeetCode problem)

### Warm-up — LC 704 "Binary Search"

Given a sorted `nums` and a `target`, return its index or `-1`. This is template **4a** verbatim — read it above, then re-type it from memory. If you can, you own the classic case.

### The real interview question — LC 875 "Koko Eating Bananas" (binary search on the answer)

> Koko has `piles` of bananas and `h` hours before the guards return. Each hour she picks one pile and eats up to **k** bananas from it (if the pile has fewer, she eats it and stops for that hour). Find the **minimum** eating speed `k` so she finishes all piles within `h` hours.

The array `piles` isn't sorted and sorting it wouldn't help — there's nothing to binary-search *in the input*. The trick is to binary-search the **answer** `k`.

**Step 1 — spot monotonicity.** If speed `k` lets her finish in time, then any *faster* speed also finishes in time. So `canFinish(k)` looks like `false, false, …, TRUE, TRUE`. Monotonic → binary-searchable. We want the **leftmost** `true`.

**Step 2 — bound the answer.** Slowest sensible speed is `1`. Fastest we'd ever need is `max(piles)` — eating the biggest pile in a single hour is always enough (never faster helps). So `k ∈ [1, max(piles)]`.

**Step 3 — the feasibility check.** Hours needed at speed `k` = `Σ ceil(pile / k)` (each pile takes `⌈pile/k⌉` whole hours). Feasible when that total ≤ `h`.

```typescript
function minEatingSpeed(piles: number[], h: number): number {
  // Hours required to eat everything at speed k.
  const hoursAt = (k: number): number =>
    piles.reduce((sum, p) => sum + Math.ceil(p / k), 0);

  let lo = 1;                        // slowest speed worth considering
  let hi = Math.max(...piles);       // fast enough to clear the biggest pile in one hour

  while (lo < hi) {                  // half-open boundary search (template 4b/4c)
    const mid = lo + Math.floor((hi - lo) / 2);
    if (hoursAt(mid) <= h) hi = mid; // fast enough → try to go SLOWER (mid might be the min)
    else lo = mid + 1;               // too slow → must go FASTER
  }
  return lo;                         // smallest speed that finishes within h hours
}
```

**Trace it** with `piles = [3, 6, 7, 11]`, `h = 8`. Start `lo = 1, hi = 11`.

| lo | hi | mid | `hoursAt(mid)` = Σ⌈pile/mid⌉ | ≤ 8? | action |
|---|---|---|---|---|---|
| 1 | 11 | 6 | ⌈3/6⌉+⌈6/6⌉+⌈7/6⌉+⌈11/6⌉ = 1+1+2+2 = **6** | yes | too easy → `hi = 6` |
| 1 | 6 | 3 | 1+2+3+4 = **10** | no | too slow → `lo = 4` |
| 4 | 6 | 5 | 1+2+2+3 = **8** | yes | just fits → `hi = 5` |
| 4 | 5 | 4 | 1+2+2+3 = **8** | yes | still fits → `hi = 4` |
| 4 | 4 | — | loop ends (`lo === hi`) | — | **return 4** |

Answer: **4** bananas/hour. Notice we never "constructed" a schedule — we only *verified* guesses, and let monotonicity guide us to the smallest one. That is the whole pattern.

Same recipe cracks **LC 1011 Capacity To Ship Packages**, **LC 410 Split Array Largest Sum**, **LC 774 Minimize Max Distance** — always: *bound the answer → write a monotonic `feasible()` → boundary-search it.*

---

## 7. Practice Set (curated LeetCode)

Do them in order — each adds one twist.

| # | Problem | Difficulty | Why it's here |
|---|---|---|---|
| 704 | Binary Search | Easy | The classic template 4a. Type it blind. |
| 35 | Search Insert Position | Easy | `lowerBound` in disguise. |
| 34 | Find First and Last Position | Medium | `lowerBound` + `upperBound` together. |
| 278 | First Bad Version | Easy | Boundary search on a monotonic predicate (a gentle "search on answer"). |
| 74 | Search a 2D Matrix | Medium | Treat the matrix as one flat sorted array (index math). |
| 33 | Search in **Rotated** Sorted Array | Medium | Decide which half is sorted, then discard. Interview favorite. |
| 153 | Find Minimum in Rotated Sorted Array | Medium | Compare `mid` to `hi` to locate the pivot. |
| 852 | Peak Index in a Mountain Array | Medium | Monotonic "am I still climbing?" predicate. |
| 875 | Koko Eating Bananas | Medium | Search on the answer (worked above). |
| 1011 | Capacity To Ship Packages Within D Days | Medium | Same skeleton, different `feasible()`. |
| 410 | Split Array Largest Sum | Hard | The boss level of "search on answer." |
| 4 | Median of Two Sorted Arrays | Hard | Binary search on a partition. Attempt last. |

> **Rotated-array cheat (LC 33), so the pattern isn't a mystery:** compute `mid`; **one side is always still sorted**. If `nums[lo] <= nums[mid]`, the left half is sorted — check if `target` lies within `[nums[lo], nums[mid]]` to decide whether to go left or right. Otherwise the right half is sorted — apply the mirror check. Still O(log n).

---

## 8. Common Pitfalls & Interview Tips

| Pitfall | Fix |
|---|---|
| **Forgetting the array must be sorted.** Binary search on unsorted data returns garbage silently. | Confirm/establish order first (see [08 — Sorting](08-sorting.md)). If you sort just to search, that's O(n log n) — only worth it for many queries, else a [hash set](03-hashing-maps-and-sets.md) lookup is O(1). |
| **Float index:** `const mid = (lo + hi) / 2` gives `2.5`, and `nums[2.5]` is `undefined`. | Always `Math.floor((lo + hi) / 2)` — or `lo + Math.floor((hi - lo) / 2)`. |
| **Off-by-one / infinite loop:** mixing `lo <= hi` with `hi = mid`, or `lo < hi` with `lo = mid`. | Pick a template and keep its invariants matched: **inclusive `[lo,hi]` ⇒ `lo <= hi` + `mid ± 1`**; **half-open `[lo,hi)` ⇒ `lo < hi` + `hi = mid` / `lo = mid + 1`**. Never cross them. |
| **Non-monotonic predicate** in "search on answer." If `feasible` flips `true → false → true`, binary search lands anywhere. | Prove monotonicity out loud before coding: *"once k works, every larger k works because…"* If you can't, it's not binary-searchable. |
| **Wrong boundary returned:** grabbing `lowerBound` when the problem wants `upperBound` (or the index *before* it). | On paper, mark which side of the flip you need. `lowerBound` = first `>=`; `upperBound` = first `>`; last `<= x` is `upperBound(x) - 1`. |
| **Overflow paranoia (Java/C++ habit).** In JS, numbers are 64-bit floats, so `lo + hi` only misbehaves past `Number.MAX_SAFE_INTEGER` (2⁵³). | Use `lo + Math.floor((hi - lo) / 2)` anyway — it costs nothing, reads clearly, and ports to other languages where overflow is real. |

**Interview scripts that score points**

- Narrate the invariant: *"I'll keep the answer inside a half-open window `[lo, hi)`; everything left of `lo` fails, everything from `hi` on isn't a candidate."* Interviewers care that you *know why* it terminates.
- Always test the empty array, a 1-element array, target smaller than all / larger than all, and duplicates.
- If someone gives you an **unbounded** or **"guess the number"** problem, mention **exponential search**: double `hi` (1, 2, 4, 8, …) until `feasible(hi)` is true, then binary-search `[hi/2, hi]`. Shows range.
- "Can you do better than O(n)?" on sorted input is almost always asking for binary search. Say the words.

---

## 9. TL;DR

- Binary search = **halve the search space each step → O(log n)**. Needs **order** (sorted data) or a **monotonic predicate**.
- **Two templates, don't mix them:**
  - *Exact match:* inclusive `[lo, hi]`, `while (lo <= hi)`, move with `mid ± 1`, return `-1` if empty.
  - *Boundary (`lowerBound`/`upperBound`):* half-open `[lo, hi)`, `while (lo < hi)`, `hi = mid` / `lo = mid + 1`, return `lo`.
- Always `mid = lo + Math.floor((hi - lo) / 2)` — **`Math.floor`** kills float indices; the `lo + (hi-lo)/2` form is a portable overflow-safe habit.
- **The big idea — binary search on the answer:** bound the answer `[lo, hi]`, write a **monotonic** `feasible(x)`, boundary-search it. Koko, ship capacity, split array all fall to this.
- Reach for it on: sorted + find in O(log n), first/last/insert position, "minimum/maximum value that works," k-th smallest, rotated arrays.
- Pitfalls: unsorted input, float mid, template mismatch (infinite loop), and a predicate that isn't actually monotonic.

Next: [08 — Sorting](08-sorting.md) — how arrays *become* searchable, and when ordering data is the entire trick.
