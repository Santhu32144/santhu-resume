# 06. Prefix Sum

> **One-liner:** Precompute a **running cumulative total** once, so the sum of *any* range becomes a single subtraction — `sum(i..j) = prefix[j+1] - prefix[i]` — answered in **O(1)**.
> **Pattern power:** Turns *repeated* range-sum questions from `O(n)` each into `O(1)` each, and — combined with a **hash map** — cracks the "count subarrays that sum to K" family that trips up almost everyone. A staple of CRED/Razorpay array rounds.

---

## 1. The Idea in Plain Words

A **prefix sum** (also "cumulative sum" / "running total") is a new array where each slot holds *the sum of everything up to that point*. If Habex shows your daily spend, the prefix-sum array is your **running month-to-date total** after each day.

```
nums   =  [ 3 ,  1 ,  4 ,  1 ,  5 ]
prefix = [0, 3 ,  4 ,  8 ,  9 , 14 ]
          ↑
   prefix[0] = 0  (sum of nothing — the crucial "empty prefix")
   prefix[k] = nums[0] + nums[1] + ... + nums[k-1]
```

Here's the whole trick. The sum of a **subarray** `nums[i..j]` (inclusive) is *"total up to and including j"* minus *"total before i"*:

```
sum(nums[i..j]) = prefix[j + 1] - prefix[i]
```

Everything from index `i` onward that both totals share **cancels**, leaving exactly the middle chunk. One subtraction — no matter how long the range. You pay `O(n)` **once** to build `prefix`, then every range query is `O(1)`.

> Why the `+1` offset and the leading `0`? Making `prefix` one longer than `nums`, with `prefix[0] = 0`, means the formula works even when the subarray starts at index `0` (`prefix[j+1] - prefix[0]`). That leading zero — the **empty prefix** — is the single most important detail in this whole chapter. Skip it and every boundary case breaks.

The real superpower comes when you stop *computing* a sum and start *asking a question about* the running total — **"have I seen a prefix like this before?"** — and answer it with a hash map. That's section 6.

---

## 2. When to Use It — Pattern Triggers

- **Repeated range-sum queries** on a fixed array → *"return the sum of elements between index `i` and `j`"*, asked many times, possibly thousands. Build once, answer each in `O(1)`.
- **"How many subarrays sum to K"** / **"does a subarray sum to K exist"** — the flagship **prefix sum + hash map** problem, *especially when numbers can be negative* (sliding window can't handle that — see [05-sliding-window.md](05-sliding-window.md)).
- **"Subarray with equal number of X and Y"** — map one symbol to `+1` and the other to `-1`; now "equal counts" becomes "**subarray sums to 0**", a prefix-sum question. (E.g. equal 0s and 1s; equal vowels and consonants.)
- **"Subarray sum divisible by K"** → track `prefix % k` in a map instead of the raw prefix.
- **2D version:** *"sum of a rectangle in a matrix"*, many queries → 2D prefix sum.
- **Product / min / xor variants:** the same "precompute a running fold" idea works for prefix products (LeetCode 238), prefix XOR, etc.

**Example phrasings that mean "prefix sum":**
> "Given an integer array, find the **total number of continuous subarrays whose sum equals `k`**."
> "Implement a class that returns the **sum of the elements between indices `left` and `right`**, called many times."
> "Find the **longest contiguous subarray with an equal number of 0 and 1**."
> "Count subarrays whose **sum is divisible by `k`**."

**Trigger vs. sibling patterns:**
- Range **sum** / "sums to K" with **possible negatives** ⇒ **prefix sum** (this chapter).
- Contiguous, **all non-negative**, "longest/shortest window" ⇒ **sliding window** ([05-sliding-window.md](05-sliding-window.md)) — often simpler when it applies.
- The "have I seen this value before?" lookups that power prefix sum live in **hash maps** ([03-hashing-maps-and-sets.md](03-hashing-maps-and-sets.md)).

---

## 3. Picture It (Diagram)

**Range sum via subtraction** — `nums = [3, 1, 4, 1, 5]`, want `sum(nums[1..3]) = 1 + 4 + 1 = 6`:

```
index:      0   1   2   3   4
nums:     [ 3   1   4   1   5 ]
                └───────┘  = the range we want (i=1 .. j=3)

prefix:  [ 0   3   4   8   9   14 ]
           0   1   2   3   4   5      ← prefix indices (shifted by 1)

sum(1..3) = prefix[3+1] - prefix[1]
          = prefix[4]   - prefix[1]
          =    9        -    3      = 6   ✔

visually:  prefix[4] covers  [3 1 4 1]
           prefix[1] covers  [3]
           subtract        →     [1 4 1]  = 6
```

**Prefix + hash map** — the running total as a "have I seen `current − k` before?" lookup. `nums = [1, 2, 3]`, `k = 3`:

```
running total:  0 → 1 → 3 → 6
                     ↑    ↑
   at total 3: have we seen (3 - 3 = 0) before?  yes, once (the empty prefix) → 1 subarray
   at total 6: have we seen (6 - 3 = 3) before?  yes, once (after index 1)    → 1 subarray
                                                             total = 2 subarrays: [1,2] and [3]
```

---

## 4. The Template (TypeScript)

### Template A — Build the prefix array (1-D range queries)

```ts
/** prefix[k] = sum of nums[0..k-1]; prefix[0] = 0 (empty prefix). */
function buildPrefix(nums: number[]): number[] {
  const prefix = new Array<number>(nums.length + 1).fill(0);
  for (let i = 0; i < nums.length; i++) {
    prefix[i + 1] = prefix[i] + nums[i];
  }
  return prefix;
}

/** Sum of nums[i..j] INCLUSIVE, in O(1). */
function rangeSum(prefix: number[], i: number, j: number): number {
  return prefix[j + 1] - prefix[i];
}
```

### Template B — Prefix sum + hash map ("count subarrays summing to k")

```ts
/** Number of contiguous subarrays whose sum equals k. Handles negatives. */
function countSubarraysSummingTo(nums: number[], k: number): number {
  const seen = new Map<number, number>(); // prefixSum -> how many indices produced it
  seen.set(0, 1);                          // SEED: the empty prefix has sum 0, once
  let running = 0;
  let count = 0;

  for (const n of nums) {
    running += n;
    // every earlier prefix equal to (running - k) starts a subarray summing to k
    count += seen.get(running - k) ?? 0;
    seen.set(running, (seen.get(running) ?? 0) + 1);
  }
  return count;
}
```

### Template C — 2D prefix sum (rectangle queries), briefly

```ts
/** pre[r][c] = sum of the sub-rectangle from (0,0) to (r-1, c-1). */
function build2D(matrix: number[][]): number[][] {
  const rows = matrix.length, cols = matrix[0].length;
  const pre: number[][] = Array.from({ length: rows + 1 }, () =>
    new Array<number>(cols + 1).fill(0)
  );
  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      // inclusion–exclusion: add cell, add top & left blocks, subtract the double-counted corner
      pre[r + 1][c + 1] = matrix[r][c] + pre[r][c + 1] + pre[r + 1][c] - pre[r][c];
    }
  }
  return pre;
}

/** Sum of rectangle (r1,c1)..(r2,c2) inclusive, in O(1). */
function rectSum(pre: number[][], r1: number, c1: number, r2: number, c2: number): number {
  return pre[r2 + 1][c2 + 1] - pre[r1][c2 + 1] - pre[r2 + 1][c1] + pre[r1][c1];
}
```

> **The two flavours of the map matter.** For **counting** subarrays you store a **frequency** and seed `{0: 1}`. For a **longest span** (like equal 0s/1s) you store the **earliest index** each prefix appeared and seed `{0: -1}`. Same idea, different bookkeeping — see the pitfalls.

---

## 5. Complexity

| Task | Precompute | Per query | Space |
|---|---|---|---|
| 1-D range sum (Template A) | `O(n)` once | `O(1)` | `O(n)` |
| Count subarrays sum = k (Template B) | — | `O(n)` total | `O(n)` map |
| 2-D rectangle sum (Template C) | `O(rows · cols)` once | `O(1)` | `O(rows · cols)` |

Compare to the naïve *recompute-every-time* approach: `q` range queries cost `O(q · n)` without prefix sums, but `O(n + q)` with them. For "count subarrays summing to k", brute force is `O(n²)` (every start × every end); the hash-map version is a single `O(n)` pass — the classic space-for-time trade that defines this pattern (and [03-hashing-maps-and-sets.md](03-hashing-maps-and-sets.md)).

---

## 6. Worked Example — Subarray Sum Equals K (LeetCode 560)

> **Problem:** Given an integer array `nums` (values may be **negative**) and an integer `k`, return the **total number of contiguous subarrays** whose sum equals `k`. E.g. `nums = [1, 1, 1], k = 2` → `2`.

### Brute force (the baseline)

Try every subarray: for each start `i`, extend `j` and accumulate the sum, counting hits. `O(n²)` time. Fine to state, too slow to submit for large `n`. And note: because values can be **negative**, you **cannot** use a sliding window (adding an element might *decrease* the sum, so there's no valid "shrink when too big"). That observation is worth saying out loud.

### Key insight

Let `prefix[x]` be the running total after `x` elements. The sum of the subarray `(i, j]` is `prefix[j] - prefix[i]`. We want that to equal `k`:

```
prefix[j] - prefix[i] = k   ⟺   prefix[i] = prefix[j] - k
```

So as you sweep `j` left to right maintaining `running = prefix[j]`, the number of subarrays ending at `j` that sum to `k` is **how many earlier prefixes equalled `running - k`**. Keep those counts in a `Map`. Seed it with `{0: 1}` for the **empty prefix**, so subarrays that start at index `0` are counted (their `prefix[i]` is the empty `0`).

### Optimal (commented TypeScript)

```ts
function subarraySum(nums: number[], k: number): number {
  const seen = new Map<number, number>(); // prefix sum -> times it has occurred
  seen.set(0, 1);                          // empty prefix: sum 0 seen once (CRUCIAL)
  let running = 0;
  let count = 0;

  for (const n of nums) {
    running += n;                          // running = prefix up to & including this element
    count += seen.get(running - k) ?? 0;   // earlier prefixes that complete a sum-k subarray
    seen.set(running, (seen.get(running) ?? 0) + 1); // record this prefix
  }
  return count;
}
```

### Trace — `nums = [1, 2, 3]`, `k = 3`

| element | `running` | need `running - k` | `seen` has it? → add | `count` | `seen` after |
|---|---|---|---|---|---|
| — | 0 | — | — | 0 | `{0: 1}` |
| 1 | 1 | `1 - 3 = -2` | no → +0 | 0 | `{0:1, 1:1}` |
| 2 | 3 | `3 - 3 = 0` | yes (×1) → +1 | 1 | `{0:1, 1:1, 3:1}` |
| 3 | 6 | `6 - 3 = 3` | yes (×1) → +1 | 2 | `{0:1, 1:1, 3:1, 6:1}` |

Answer **2** — the subarrays `[1, 2]` and `[3]`. ✔

**Why the `{0: 1}` seed and frequency (not a Set) matter** — trace `nums = [1, -1, 0], k = 0`:

| element | `running` | need `-0` | add | `count` | `seen` after |
|---|---|---|---|---|---|
| — | 0 | — | — | 0 | `{0: 1}` |
| 1 | 1 | `1` | no | 0 | `{0:1, 1:1}` |
| -1 | 0 | `0` | yes (×1) → +1 | 1 | `{0:2, 1:1}` |
| 0 | 0 | `0` | yes (×**2**) → +2 | 3 | `{0:3, 1:1}` |

Answer **3** — `[1,-1]`, `[1,-1,0]`, and `[0]`. This only comes out right because we (a) seeded the empty prefix and (b) stored **how many times** each prefix occurred, not just whether it appeared. A plain `Set` would undercount.

### Must-know cousin

**Range Sum Query — Immutable (LeetCode 303)** — the pure Template-A use case: build `prefix` in the constructor, answer each `sumRange(left, right)` in `O(1)`.

```ts
class NumArray {
  private prefix: number[];

  constructor(nums: number[]) {
    this.prefix = new Array<number>(nums.length + 1).fill(0);
    for (let i = 0; i < nums.length; i++) {
      this.prefix[i + 1] = this.prefix[i] + nums[i];
    }
  }

  sumRange(left: number, right: number): number {
    return this.prefix[right + 1] - this.prefix[left]; // inclusive [left, right]
  }
}
```

And the **+1/−1 mapping** trick — **Contiguous Array (LeetCode 525)**, longest subarray with equal 0s and 1s. Treat `0` as `-1`, `1` as `+1`; an equal count means the mapped subarray **sums to 0**. Here we want the *longest span*, so store each prefix's **earliest index** and seed `{0: -1}`:

```ts
function findMaxLength(nums: number[]): number {
  const firstSeen = new Map<number, number>(); // prefix sum -> earliest index
  firstSeen.set(0, -1);                         // empty prefix sits "before" index 0
  let running = 0;
  let best = 0;

  for (let i = 0; i < nums.length; i++) {
    running += nums[i] === 1 ? 1 : -1;
    const prev = firstSeen.get(running);
    if (prev !== undefined) {
      best = Math.max(best, i - prev);          // same prefix twice ⇒ zero-sum span between
    } else {
      firstSeen.set(running, i);                // keep EARLIEST index → longest span
    }
  }
  return best;
}
```

---

## 7. Practice Set (curated LeetCode)

| # | Name | Difficulty | Trigger it teaches |
|---|---|---|---|
| 1480 | Running Sum of 1d Array | Easy | literally build the prefix array |
| 303 | Range Sum Query — Immutable | Easy | precompute → `O(1)` queries |
| 724 | Find Pivot Index | Easy | left sum vs. right sum |
| 560 | Subarray Sum Equals K | Medium | prefix + hash map count |
| 525 | Contiguous Array | Medium | map 0→−1, longest zero-sum span |
| 238 | Product of Array Except Self | Medium | prefix × suffix products |
| 974 | Subarray Sums Divisible by K | Medium | count on `prefix % k` |
| 523 | Continuous Subarray Sum | Medium | `prefix % k` first-seen index |
| 304 | Range Sum Query 2D — Immutable | Medium | 2-D prefix sum rectangle |
| 1248 | Count Number of Nice Subarrays | Medium | prefix on odd-count parity |
| 325 | Maximum Size Subarray Sum Equals k | Medium | prefix + earliest index |
| 1658 | Minimum Operations to Reduce X to Zero | Medium | reframe as a subarray-sum target |

---

## 8. Common Pitfalls & Interview Tips

- **Forgetting the empty prefix (`seed {0: 1}` or `{0: -1}`).** Without it you miss every subarray that starts at index `0`, and your answer is silently too low. This is *the* prefix-sum bug. Seed value depends on the job: **`{0: 1}`** when counting occurrences (Template B); **`{0: -1}`** when tracking the earliest index for a longest span.

- **Off-by-one on inclusive vs. exclusive ranges.** With a length-`(n+1)` prefix array (`prefix[0] = 0`), the **inclusive** sum of `nums[i..j]` is `prefix[j + 1] - prefix[i]`. Decide your convention up front and write it as a comment — flipping the `+1` between the two ends is the second most common bug.

- **Count vs. longest ⇒ frequency vs. earliest index.** Counting subarrays (560, 974): store **how many times** each prefix occurred. Longest span (525, 325): store the **earliest index** and never overwrite it. Using the wrong one gives plausible-but-wrong numbers.

- **A `Set` can't count.** For "how many subarrays", you need a `Map<number, number>` (frequency), not a `Set` — otherwise repeated prefixes are undercounted (see the `[1,-1,0]` trace). Map fundamentals: [03-hashing-maps-and-sets.md](03-hashing-maps-and-sets.md).

- **Use `?? 0`, not `|| 0`, when reading counts.** `map.get(x) ?? 0` treats only `undefined` as missing. `map.get(x) || 0` would also wipe a legitimately stored `0`-valued count. And a raw `map.get(x)` is `number | undefined` — TS will (correctly) complain if you add it without a fallback.

- **Divisibility: normalise negative remainders.** For "sum divisible by k" (974/523), JS's `%` can return a negative result (`-1 % 5 === -1`). Canonicalise with `((running % k) + k) % k` so `-1` and `4` map to the same bucket.

- **Prefix sums shine with negatives — that's their edge over sliding window.** If a problem is "subarray sum = K" and numbers can be negative, don't even try a window; reach straight for prefix + map ([05-sliding-window.md](05-sliding-window.md) explains why the window breaks).

- **What to say out loud:** *"I'll keep a running prefix sum. A subarray sums to k exactly when some earlier prefix equals `current − k`, so I count those with a hash map seeded at `{0: 1}` for the empty prefix. One pass, O(n) time and space — and unlike a sliding window it's correct with negative numbers."*

---

## 9. TL;DR

- **Prefix sum = a running cumulative total.** Build once in `O(n)`; then `sum(nums[i..j]) = prefix[j+1] - prefix[i]` in `O(1)`.
- **Always include the empty prefix** (`prefix[0] = 0`, or map seed `{0: 1}`). This one detail fixes all the boundary cases.
- **Prefix + hash map** counts subarrays summing to `k` in a single `O(n)` pass: at each step, add `seen.get(running - k)`, then record `running`. Works with **negatives**, where sliding window fails.
- **Count → store frequencies, seed `{0: 1}`. Longest span → store earliest index, seed `{0: -1}`.**
- **"Equal number of X and Y"** → map to `+1 / -1` and hunt for a **zero-sum** subarray. **2-D** rectangles → 2-D prefix via inclusion–exclusion.
- **JS/TS traps:** `?? 0` not `|| 0`; normalise `%` for divisibility; mind the inclusive/exclusive `+1`. The map machinery lives in [03-hashing-maps-and-sets.md](03-hashing-maps-and-sets.md).
