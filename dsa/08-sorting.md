# 08. Sorting

> **One-liner:** Put data in order first, and a shocking number of "hard" problems collapse — sorting turns O(n²) brute force into **O(n log n)**, and switches on [two pointers](04-two-pointers.md), [greedy](20-greedy-and-intervals.md), and [binary search](07-binary-search.md).
> **Pattern power:** Half of medium interview problems have the same secret first line: *"…what if I sort it?"* Learning to reach for sorting — and knowing when **not** to — is one of the highest-leverage habits you can build.

---

## 1. The Idea in Plain Words

Sorting means **arranging elements in order** (numbers ascending, strings A→Z, objects by some field). You already do it every day: a leaderboard, a "sort by price" dropdown, files by date.

For interviews the real lesson isn't *how* the library sorts — it's that **order is structure**, and structure kills brute force:

- **"Are there two numbers that sum to K?"** Unsorted: check every pair, O(n²). Sorted: walk two pointers from both ends, O(n).
- **"Any duplicates?"** Sorted: duplicates sit next to each other — one linear pass.
- **"Merge overlapping intervals?"** Sorted by start: overlaps become adjacent, so one sweep does it.

So sorting is rarely the *answer* by itself — it's the **setup move** that makes the real technique cheap. You pay O(n log n) once and unlock an O(n) (or O(log n)) finish.

Two new terms you'll hear constantly:

- **In-place:** rearranges the original array using O(1) extra memory (no big second array).
- **Stable:** equal elements keep their original relative order. Crucial when you sort by one key and want ties broken by a *previous* order — e.g. sort users by score, and among equal scores keep them alphabetical from an earlier sort.

---

## 2. When to Use It — Pattern Triggers

| Signal in the problem | Sorting unlocks… |
|---|---|
| "find a **pair/triplet** summing to X", "closest pair" | [Two pointers](04-two-pointers.md) on the sorted array → O(n) / O(n²) instead of O(n²) / O(n³). |
| "**merge / insert intervals**", "meeting rooms", "non-overlapping" | Sort by start, then sweep once → [Greedy & Intervals](20-greedy-and-intervals.md). |
| "**minimum number of** …", "schedule to maximize/minimize" | Greedy usually needs a sorted order to make each locally-best choice. |
| "does X exist" on data you'll **query many times** | Sort once (O(n log n)), then each query is a [binary search](07-binary-search.md) O(log n). |
| "**k-th largest / smallest**", "top K" | Sorting works (O(n log n)); but a [heap](14-heaps-and-priority-queues.md) is often better (O(n log k)) or Quickselect O(n) avg. |
| "group **anagrams / duplicates** together" | Sort each string / the array so equal things become adjacent. |
| "arrange numbers to form the largest/smallest result" | A **custom comparator** *is* the whole solution. |

> **Rule of thumb:** if a problem feels O(n²) and the *positions* of elements don't matter to the final answer, try sorting first. It's the cheapest thing that changes the shape of a problem.

---

## 3. Picture It (Diagram)

**Merge sort** — the "divide & conquer" you should be able to draw. Split until pieces are size 1 (trivially sorted), then **merge** pieces back together in order.

```
                    [ 5 , 2 , 8 , 1 ]
                   /                  \          DIVIDE  (keep splitting in half)
          [ 5 , 2 ]                    [ 8 , 1 ]
          /       \                    /       \
       [ 5 ]     [ 2 ]              [ 8 ]     [ 1 ]     ← size 1: already "sorted"
          \       /                    \       /
          [ 2 , 5 ]                    [ 1 , 8 ]        MERGE  (weave two sorted lists)
                   \                  /
                    [ 1 , 2 , 5 , 8 ]                   ← fully sorted
```

There are `log n` levels of splitting (halving each time), and merging one full level costs O(n) — so **O(n log n)** total. Every O(n log n) comparison sort has this same "log n levels × n work" shape.

---

## 4. The Template (TypeScript)

### 4a. The comparator — where JS/TS burns everyone

`Array.prototype.sort()` with **no comparator converts elements to strings and sorts lexicographically.** This is *the* most common TS interview bug:

```typescript
[10, 2, 1, 100].sort();          // ❌ → [1, 10, 100, 2]   ("10" < "2" as strings!)
[10, 2, 1, 100].sort((a, b) => a - b);  // ✅ → [1, 2, 10, 100]
```

The comparator `(a, b) => number` contract: **negative** → `a` before `b`; **positive** → `b` before `a`; **0** → treat as equal. So `a - b` sorts ascending, `b - a` descending. Every pattern you need:

```typescript
const nums = [10, 2, 1, 100];

nums.sort((a, b) => a - b);   // ascending  → [1, 2, 10, 100]
nums.sort((a, b) => b - a);   // descending → [100, 10, 2, 1]

// By an object field
type User = { name: string; score: number };
users.sort((a, b) => a.score - b.score);          // lowest score first

// Multi-key / tie-break: score DESC, then name A→Z when scores tie
users.sort((a, b) => b.score - a.score || a.name.localeCompare(b.name));
//                    └ primary key ┘    └ runs only when primary is 0 (tie) ┘

// Strings: use localeCompare, NOT a - b (subtracting strings gives NaN → no sorting)
words.sort((a, b) => a.localeCompare(b));
```

> **Two more traps:**
> 1. **`sort()` mutates the array in place** *and returns it.* If you must keep the original (e.g. it's a function argument you don't own), copy first: `const s = [...arr].sort((a,b)=>a-b)`.
> 2. **Stability:** V8 (Node/Chrome) has been stable since ES2019, so equal elements keep their order there. But don't *assume* stability across every runtime historically — if correctness depends on it, encode the tie-breaker explicitly in the comparator (as above).

### 4b. Merge sort — the implementation interviewers ask you to write

Stable, always O(n log n), and the clearest divide-and-conquer to code:

```typescript
function mergeSort(arr: number[]): number[] {
  if (arr.length <= 1) return arr;             // base case: 0 or 1 element is sorted

  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));   // sort the left half
  const right = mergeSort(arr.slice(mid));     // sort the right half
  return merge(left, right);                   // weave the two sorted halves
}

function merge(left: number[], right: number[]): number[] {
  const out: number[] = [];
  let i = 0, j = 0;

  while (i < left.length && j < right.length) {
    // '<=' takes from the LEFT on ties → equal elements keep original order → STABLE
    if (left[i] <= right[j]) out.push(left[i++]);
    else out.push(right[j++]);
  }
  // exactly one side has leftovers; append them (already sorted)
  while (i < left.length) out.push(left[i++]);
  while (j < right.length) out.push(right[j++]);
  return out;
}
```

### 4c. Quick sort — the in-place idea (partition)

Pick a **pivot**, **partition** so everything smaller sits left of it and everything larger sits right, then recurse on each side. The pivot lands in its final position every round.

```typescript
function quickSort(arr: number[], lo = 0, hi = arr.length - 1): number[] {
  if (lo >= hi) return arr;               // 0 or 1 element in this segment

  const pivot = arr[hi];                  // simple pivot = last element
  let i = lo;                             // boundary: arr[lo..i-1] are all < pivot
  for (let j = lo; j < hi; j++) {
    if (arr[j] < pivot) {
      [arr[i], arr[j]] = [arr[j], arr[i]]; // swap the smaller value into the left zone
      i++;
    }
  }
  [arr[i], arr[hi]] = [arr[hi], arr[i]];  // drop the pivot into its final slot
  quickSort(arr, lo, i - 1);              // sort the left part (< pivot)
  quickSort(arr, i + 1, hi);              // sort the right part (>= pivot)
  return arr;                             // mutated IN PLACE
}
```

The catch: if the pivot is always the min or max (e.g. an already-sorted array with last-element pivot), partitions are size 0 and n−1 → **O(n²)**. Real libraries fix this by choosing a **random** or **median-of-three** pivot. (V8's built-in sort is actually **Timsort** — a merge/insertion hybrid — so you rarely hand-roll quicksort in JS; know the idea for the whiteboard.)

---

## 5. Complexity

Comparison sorts (compare pairs of elements) — the "log n" family:

| Algorithm | Best | Average | Worst | Space | Stable? | Why you'd care |
|---|---|---|---|---|---|---|
| Bubble | O(n) | O(n²) | O(n²) | O(1) | Yes | Teaching only. Best case O(n) with an early-exit flag. |
| Selection | O(n²) | O(n²) | O(n²) | O(1) | No | Fewest swaps (n−1). Rarely useful. |
| Insertion | O(n) | O(n²) | O(n²) | O(1) | Yes | **Great on tiny or nearly-sorted** input; used inside Timsort. |
| **Merge** | O(n log n) | O(n log n) | O(n log n) | O(n) | **Yes** | Guaranteed n log n + stable. The safe default to *implement*. |
| **Quick** | O(n log n) | O(n log n) | **O(n²)** | O(log n) | No | Fast constants, in-place. Worst case on bad pivots. |
| Heap | O(n log n) | O(n log n) | O(n log n) | O(1) | No | n log n with O(1) space; see [14 — Heaps](14-heaps-and-priority-queues.md). |

Non-comparison sorts (bucket keys directly) — beat the O(n log n) "comparison lower bound" **only when keys are bounded integers**:

| Algorithm | Time | Space | Stable? | Use when |
|---|---|---|---|---|
| Counting sort | O(n + k) | O(k) | Yes | Integer keys in a **small** range `[0, k]` (e.g. ages, grades 0–100). |
| Radix sort | O(d · (n + k)) | O(n + k) | Yes | Fixed-width keys — integers/strings of `d` digits. |
| Bucket sort | O(n + k) avg, O(n²) worst | O(n) | Yes | Values **uniformly distributed** over a range (e.g. floats in [0,1)). |

> **Why can't a comparison sort beat O(n log n)?** With only pairwise comparisons there are `n!` possible orderings to distinguish, and a decision tree needs `log₂(n!) ≈ n log n` comparisons. Counting/radix dodge this by *not comparing* — they use the key as an array index. Say this in an interview and you sound like you actually understand sorting.

**What does your `Array.prototype.sort()` cost?** O(n log n) time, stable, via Timsort in V8. That's what you'll use 99% of the time — you implement merge/quick sort only when explicitly asked.

---

## 6. Worked Example (a classic LeetCode problem)

### LC 56 "Merge Intervals" — sorting *is* the insight

> Given intervals like `[[1,3],[2,6],[8,10],[15,18]]`, merge all overlapping ones. (Deep dive lives in [20 — Greedy & Intervals](20-greedy-and-intervals.md); here we focus on *why sorting cracks it*.)

**The key realization:** if intervals are **sorted by start time**, any interval that overlaps the current one must come *immediately after* it. That turns "does this overlap with any other interval?" (O(n²)) into "does this overlap the *previous* one?" (O(n)). Sorting is the entire unlock.

```typescript
function merge(intervals: number[][]): number[][] {
  if (intervals.length === 0) return [];

  // THE move: sort by start. (Default sort would compare [1,3] as the string "1,3" — wrong.)
  intervals.sort((a, b) => a[0] - b[0]);

  const result: number[][] = [intervals[0].slice()]; // copy so we don't mutate the input row
  for (let i = 1; i < intervals.length; i++) {
    const last = result[result.length - 1];
    const [start, end] = intervals[i];
    if (start <= last[1]) {
      last[1] = Math.max(last[1], end);  // overlap → extend the current interval's end
    } else {
      result.push([start, end]);         // gap → begin a fresh interval
    }
  }
  return result;
}
```

**Trace** with input `[[1,3],[2,6],[8,10],[15,18]]` (already sorted by start):

| Looking at | last block | `start <= last.end`? | Action | `result` after |
|---|---|---|---|---|
| — | — | — | seed with first | `[[1,3]]` |
| `[2,6]` | `[1,3]` | `2 <= 3` ✅ | extend end to `max(3,6)=6` | `[[1,6]]` |
| `[8,10]` | `[1,6]` | `8 <= 6` ❌ | gap → push new | `[[1,6],[8,10]]` |
| `[15,18]` | `[8,10]` | `15 <= 10` ❌ | gap → push new | `[[1,6],[8,10],[15,18]]` |

Result: `[[1,6],[8,10],[15,18]]`. Without the sort, `[8,10]` and `[2,6]` could be in any order and the single-pass sweep would miss merges. **O(n log n)** for the sort dominates; the sweep is O(n).

### Bonus comparator flex — LC 179 "Largest Number"

Sometimes the *comparator* is the whole algorithm. Arrange numbers so their concatenation is the largest possible: for two pieces `a`, `b`, put `a` first iff `a+b > b+a` (compare the two concatenations as strings — they're the same length, so string `>` is exactly the right test, and it never overflows).

```typescript
function largestNumber(nums: number[]): string {
  const parts = nums.map(String).sort((a, b) => {
    const ab = a + b, ba = b + a;         // two ways to glue them
    return ab === ba ? 0 : ab > ba ? -1 : 1; // bigger concatenation goes first
  });
  return parts[0] === '0' ? '0' : parts.join(''); // all-zeros edge case → "0", not "000"
}
// [3, 30, 34, 5, 9]  →  "9534330"
```

---

## 7. Practice Set (curated LeetCode)

| # | Problem | Difficulty | The sorting insight |
|---|---|---|---|
| 912 | Sort an Array | Medium | Implement merge/quick sort yourself (built-in banned). |
| 88 | Merge Sorted Array | Easy | The `merge` step of merge sort — merge in place from the back. |
| 75 | Sort Colors | Medium | Counting sort *or* Dutch-national-flag 3-way partition. |
| 169 | Majority Element | Easy | Sort → the middle element is the majority. |
| 217 | Contains Duplicate | Easy | Sort → duplicates are adjacent (or just use a [Set](03-hashing-maps-and-sets.md)). |
| 268 | Missing Number | Easy | Sort → find the first index that ≠ its value. |
| 1 | Two Sum (variant) | Easy | Sort + [two pointers](04-two-pointers.md) if indices aren't required. |
| 15 | 3Sum | Medium | **Sort first**, then two pointers per anchor. Canonical. |
| 56 | Merge Intervals | Medium | Sort by start, sweep (worked above). |
| 179 | Largest Number | Medium | Custom concatenation comparator. |
| 49 | Group Anagrams | Medium | Sort each word → sorted form is the group key. |
| 215 | Kth Largest Element | Medium | Sort is O(n log n); compare vs [heap](14-heaps-and-priority-queues.md) / Quickselect. |

---

## 8. Common Pitfalls & Interview Tips

| Pitfall | Fix |
|---|---|
| **Default lexicographic sort.** `[10, 2, 1].sort()` → `[1, 10, 2]`, not `[1, 2, 10]`. | *Always* pass a comparator: `.sort((a, b) => a - b)`. This is the #1 TS sorting bug — say it out loud in interviews. |
| **`sort()` mutates in place.** You reorder an array a caller still needs. | Copy before sorting: `[...arr].sort(cmp)` or `arr.slice().sort(cmp)`. |
| **Sorting strings/objects with `a - b`.** `"b" - "a"` is `NaN`; a `NaN` comparator leaves the array unsorted. | Strings → `a.localeCompare(b)`; objects → subtract a numeric field, e.g. `a.age - b.age`. |
| **Assuming stability everywhere.** Historically not all engines were stable. | V8 is stable now, but if correctness depends on tie order, **encode the tie-break in the comparator** (`primary || secondary`). |
| **Sorting when you didn't need to.** Pay O(n log n) for something O(n) via hashing/heap. | Only need membership/counts? → [hash Set/Map](03-hashing-maps-and-sets.md), O(n). Only need top-K? → [heap](14-heaps-and-priority-queues.md), O(n log k). Already sorted? → don't re-sort. |
| **Comparator returns a boolean.** `(a, b) => a > b` returns `true`/`false`, not a number → undefined behavior. | Return a **number**: `(a, b) => a - b`. |
| **Big-number comparator overflow** (e.g. Largest Number via `Number(a+b)`). | Compare as **strings** of equal length, or subtract only when values fit in `Number.MAX_SAFE_INTEGER` (2⁵³). |

**Interview tips**

- **Reach for sorting to break O(n²).** When a brute force pairs everything up and positions don't matter, "let me sort first" is often the pivot to the optimal solution. Then two pointers / greedy / binary search finishes it.
- **State the cost.** Sorting is O(n log n); if the rest of your algorithm is O(n), the sort dominates — say so.
- **Know when NOT to sort.** If order already matters, if you only need counts (hash) or the top-K (heap), or if an O(n) counting sort applies, sorting is the wrong tool. Naming the alternative signals maturity.
- A real-world tie-in: in **Habex** you might render habits sorted by streak with `habits.sort((a, b) => b.streak - a.streak)` (descending). Forget the comparator and a streak of `10` sorts *before* `2` because `"10" < "2"` as strings — the exact interview bug, live in your own UI.

---

## 9. TL;DR

- **Sorting is a setup move, not usually the answer.** It converts O(n²) into O(n log n) and turns on [two pointers](04-two-pointers.md), [greedy/intervals](20-greedy-and-intervals.md), and [binary search](07-binary-search.md).
- **The TS trap:** `sort()` is **lexicographic by default** and **mutates in place**. Always pass `(a, b) => a - b` (numeric) / `b - a` (desc) / `a.localeCompare(b)` (strings); copy first if you must preserve the input.
- **Tie-breaks:** `(a, b) => primary || secondary`. V8 is stable, but encode order-sensitive logic explicitly.
- **Algorithms to know:** merge sort (stable, always O(n log n), the one to *implement*), quick sort (in-place, avg O(n log n), worst O(n²), partition idea), and non-comparison sorts (counting/radix/bucket → O(n) when keys are bounded integers).
- **Don't sort when:** you only need membership/counts ([hash](03-hashing-maps-and-sets.md), O(n)), top-K ([heap](14-heaps-and-priority-queues.md), O(n log k)), or the data is already ordered.
- Worked example: **Merge Intervals** — sort by start, then a single O(n) sweep merges neighbors.

Next up: [09 — Linked Lists](09-linked-lists.md). Or revisit [07 — Binary Search](07-binary-search.md) to see what a freshly sorted array lets you do in O(log n).
