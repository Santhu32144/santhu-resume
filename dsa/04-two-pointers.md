# 04. Two Pointers

> **One-liner:** Walk **two indices** through an array (from the ends toward each other, or both forward at different speeds) so you solve in **one pass** what naïvely needs a nested loop.
> **Pattern power:** Turns a whole family of `O(n²)` "try every pair" problems into `O(n)` — the single biggest "free" speedup you'll learn, and it shows up constantly in Zepto/Swiggy/Razorpay array rounds.

---

## 1. The Idea in Plain Words

A **pointer** here just means *an index variable* — a number that points at a spot in the array. "Two pointers" means you keep **two** such indices and move them intelligently instead of using two nested `for` loops.

The naïve way to find a pair that sums to a target is:

```ts
// O(n²): check every pair — the thing we want to avoid
for (let i = 0; i < arr.length; i++) {
  for (let j = i + 1; j < arr.length; j++) {
    if (arr[i] + arr[j] === target) return [i, j];
  }
}
```

That's `n × n` work. The two-pointer trick removes the inner loop by using **structure in the data** (usually: the array is **sorted**) to decide, at each step, which pointer to move. You never re-examine a pair you've already ruled out.

There are **two flavours**, and 90% of your job is recognising which one a problem wants:

| Flavour | Pointers start… | They move… | Classic use |
|---|---|---|---|
| **Opposite ends** (converging) | one at `0`, one at `n-1` | *toward each other* | sorted-array pair/triplet, palindrome, container, reverse |
| **Same direction** (fast/slow) | both near the start | *both forward*, at different speeds | in-place dedupe, move/partition, merge, "read vs write" |

> Think of **opposite ends** like reading a receipt in Habex from both the top and bottom at once to find two line-items that add up to a total — if the running sum is too big you drop the biggest item (move the high end down); too small, you add a bigger one (move the low end up). **Same direction** is like re-typing a list while skipping blanks: a *fast* finger scans every item, a *slow* finger only advances when it writes a keeper.

---

## 2. When to Use It — Pattern Triggers

Scan the problem statement for these signals. Any one of them should make "two pointers" light up in your head.

**Opposite-ends triggers:**
- The array is **already sorted** (or you're allowed to sort it) **and** you need to **find a pair / triplet with a target sum** → *"return two numbers that add up to `target`"*, *"find all unique triplets that sum to 0"*.
- **Palindrome** check → *"is this string/array the same forwards and backwards?"*
- **Something measured between two ends** → *"container with most water"*, *"trapping rain water"*, *"max area"*.
- **Reverse in place** → *"reverse the string/array without extra space"*.

**Same-direction triggers:**
- **Modify an array in place** and return the new length → *"remove duplicates from a sorted array"*, *"move all zeroes to the end"*, *"remove element X"*.
- **Merge two sorted sequences** → *"merge two sorted arrays / lists"*.
- **Partition** by a rule → *"put evens before odds"*, *"sort colors (0s, 1s, 2s)"*.
- A **"read pointer" and a "write pointer"** naturally exist: you scan everything but only *keep* some of it.

**Example phrasings that mean "two pointers":**
> "Given a **sorted** array, find two numbers such that they add up to a specific target."
> "Find all **unique triplets** `[a, b, c]` such that `a + b + c == 0`."
> "Return the array after **removing duplicates in-place**, and the new length."
> "Given `n` vertical lines, find two that **hold the most water**."

> ⚠️ The opposite-ends money trick **requires sorted data**. If the input isn't sorted and the problem *doesn't* let you sort (e.g. it needs original indices, like plain **Two Sum**), reach for a **hash map** instead → see [03-hashing-maps-and-sets.md](03-hashing-maps-and-sets.md). If it's about a **contiguous** run (a window that grows/shrinks), that's the specialised same-direction cousin → **Sliding Window** in [05-sliding-window.md](05-sliding-window.md).

---

## 3. Picture It (Diagram)

**Opposite ends** — Two Sum on a sorted array, `target = 9`:

```
index:   0    1    2    3
value: [ 2 ,  7 , 11 , 15 ]
         L              R          sum = 2 + 15 = 17  > 9  → too big,  move R left
         ↓              ↓
       [ 2 ,  7 , 11 , 15 ]
         L         R               sum = 2 + 11 = 13  > 9  → too big,  move R left
         ↓         ↓
       [ 2 ,  7 , 11 , 15 ]
         L    R                    sum = 2 +  7 =  9  == 9 → FOUND  ✔
```

Each step throws away a whole set of pairs at once. `15` was too big with the smallest value `2`, so `15` can't pair with *anything* to make 9 — we drop it forever. That's why it's `O(n)`, not `O(n²)`.

**Same direction** — remove duplicates in place from a sorted array. `slow` = last unique slot, `fast` = scanner:

```
       [ 1 , 1 , 2 , 3 , 3 ]
   slow  ^
   fast      ^ (=1, same as slow's 1 → skip, just advance fast)

       [ 1 , 1 , 2 , 3 , 3 ]
   slow  ^
   fast          ^ (=2, new! → slow++, write 2 into slow)

       [ 1 , 2 , 2 , 3 , 3 ]
   slow      ^
   fast              ^ (=3, new! → slow++, write 3 into slow)

       [ 1 , 2 , 3 , 3 , 3 ]
   slow          ^
                              length of unique prefix = slow + 1 = 3
result: first 3 items → [1, 2, 3]
```

---

## 4. The Template (TypeScript)

### Variant A — Opposite ends (converging)

```ts
/**
 * Opposite-ends two pointers.
 * PRECONDITION for the sum/target flavour: `arr` is SORTED ascending.
 */
function oppositeEnds(arr: number[], target: number): [number, number] {
  let left = 0;
  let right = arr.length - 1;

  while (left < right) {          // stop when they meet/cross
    const sum = arr[left] + arr[right];

    if (sum === target) {
      return [left, right];       // hit — record / return
    } else if (sum < target) {
      left++;                     // too small → need a bigger value → move left up
    } else {
      right--;                    // too big  → need a smaller value → move right down
    }
  }
  return [-1, -1];                // no valid pair
}
```

### Variant B — Same direction (fast / slow, "read vs write")

```ts
/**
 * Same-direction two pointers.
 * `slow` is the write cursor; `fast` reads every element.
 * Returns the length of the kept prefix.
 */
function sameDirection(arr: number[]): number {
  let slow = 0;                        // next position to WRITE a keeper
  for (let fast = 0; fast < arr.length; fast++) {
    if (shouldKeep(arr[fast])) {       // your rule for "this element stays"
      arr[slow] = arr[fast];           // write it forward
      slow++;
    }
  }
  return slow;                         // number of kept elements
}

function shouldKeep(_x: number): boolean {
  return true; // replace with the problem's condition
}
```

The two variants share a spine — *two indices, moved by a rule* — but differ in **where they start** and **which direction they head**. Once you internalise that, most array problems become "which template, and what's the move-rule?"

---

## 5. Complexity

| Problem shape | Time | Space | Why |
|---|---|---|---|
| Opposite-ends on already-sorted data | `O(n)` | `O(1)` | each pointer crosses the array once; no extra structure |
| Opposite-ends when **you** must sort first | `O(n log n)` | `O(1)`–`O(n)` | sort dominates; the scan is only `O(n)` — see [08-sorting.md](08-sorting.md) |
| Same-direction (dedupe / partition / merge) | `O(n)` | `O(1)` | one linear pass, in place |
| **3Sum** (sort + fix one + inner two pointers) | `O(n²)` | `O(1)`* | `n` anchors × `O(n)` inner scan; sort is only `O(n log n)` |

\* Excluding the output array. The `O(n²)` for 3Sum beats the brute-force `O(n³)` of three nested loops — a huge win.

The headline: opposite-ends and same-direction are both **linear** once data is sorted. Sorting, when needed, is the only thing that pushes you to `O(n log n)`.

---

## 6. Worked Example — Two Sum II (LeetCode 167)

> **Problem:** Given a **1-indexed**, **sorted** array `numbers` and a `target`, return the indices `[i, j]` (1-based) of the two numbers that add up to `target`. Exactly one solution exists; you may not use the same element twice.

### Brute force (the baseline you say out loud)

Two nested loops testing every pair: `O(n²)` time. Correct, but it **ignores that the array is sorted** — a red flag in interviews. When you notice "sorted" in the prompt, you should immediately think *binary search* ([07-binary-search.md](07-binary-search.md)) or *two pointers*.

### Key insight

Put one pointer at each end. The **sum tells you which pointer to move**, and each move is forced:
- `sum < target` → the sum is too small. The only way to grow it is to include a **bigger** number, so move `left` **up** (the array is sorted, so bigger values are to the right).
- `sum > target` → too big. Move `right` **down** to a smaller number.
- `sum === target` → done.

Because every move permanently eliminates a value that can't be part of *any* answer, you touch each element at most once → `O(n)` time, `O(1)` space.

### Optimal (commented TypeScript)

```ts
function twoSumSorted(numbers: number[], target: number): [number, number] {
  let left = 0;
  let right = numbers.length - 1;

  while (left < right) {
    const sum = numbers[left] + numbers[right];

    if (sum === target) {
      // problem wants 1-based indices
      return [left + 1, right + 1];
    } else if (sum < target) {
      left++;   // need a larger sum → advance the low end
    } else {
      right--;  // need a smaller sum → retreat the high end
    }
  }

  return [-1, -1]; // unreachable given the problem's guarantee, but keep TS happy
}
```

### Trace — `numbers = [2, 7, 11, 15]`, `target = 9`

| Step | `left` | `right` | `numbers[left]+numbers[right]` | vs target | Action |
|---|---|---|---|---|---|
| 1 | 0 | 3 | 2 + 15 = 17 | `> 9` | `right--` |
| 2 | 0 | 2 | 2 + 11 = 13 | `> 9` | `right--` |
| 3 | 0 | 1 | 2 + 7 = 9 | `== 9` | **return `[1, 2]`** |

Three comparisons instead of six pair-checks — and the gap widens fast as `n` grows.

### Two must-know extensions

**3Sum (LeetCode 15)** — *the* interview favourite. Sort, **fix one anchor**, then run opposite-ends two pointers on the rest to find the other two:

```ts
function threeSum(nums: number[]): number[][] {
  const res: number[][] = [];
  nums.sort((a, b) => a - b);        // MUST pass a comparator — see pitfalls!

  for (let i = 0; i < nums.length - 2; i++) {
    if (nums[i] > 0) break;                       // smallest is positive → no triple sums to 0
    if (i > 0 && nums[i] === nums[i - 1]) continue; // skip duplicate anchors

    let left = i + 1;
    let right = nums.length - 1;
    while (left < right) {
      const sum = nums[i] + nums[left] + nums[right];
      if (sum === 0) {
        res.push([nums[i], nums[left], nums[right]]);
        left++;
        right--;
        // skip duplicate second/third values so triplets stay unique
        while (left < right && nums[left] === nums[left - 1]) left++;
        while (left < right && nums[right] === nums[right + 1]) right--;
      } else if (sum < 0) {
        left++;   // sum too small → bigger left value
      } else {
        right--;  // sum too big → smaller right value
      }
    }
  }
  return res;
}
```

**Container With Most Water (LeetCode 11)** — opposite ends again, but the move-rule is *"advance the shorter wall"*, because the shorter wall is what caps the area; moving the taller one can only shrink the width without ever raising the ceiling:

```ts
function maxArea(height: number[]): number {
  let left = 0;
  let right = height.length - 1;
  let best = 0;

  while (left < right) {
    const area = (right - left) * Math.min(height[left], height[right]);
    best = Math.max(best, area);

    if (height[left] < height[right]) left++; // move the limiting (shorter) wall
    else right--;
  }
  return best;
}
```

---

## 7. Practice Set (curated LeetCode)

Do them in order — easy first to lock the mechanics, then the mediums that combine two pointers with sorting.

| # | Name | Difficulty | Trigger it teaches |
|---|---|---|---|
| 344 | Reverse String | Easy | swap opposite ends inward |
| 125 | Valid Palindrome | Easy | opposite ends + char compare |
| 167 | Two Sum II — Input Array Is Sorted | Easy | sorted → pair sum |
| 26 | Remove Duplicates from Sorted Array | Easy | slow/fast in-place write |
| 27 | Remove Element | Easy | slow/fast partition |
| 283 | Move Zeroes | Easy | slow/fast, keep order |
| 88 | Merge Sorted Array | Easy | two pointers **from the back** |
| 977 | Squares of a Sorted Array | Easy | biggest square sits at an end |
| 15 | 3Sum | Medium | sort + fix anchor + two pointers |
| 16 | 3Sum Closest | Medium | track the closest sum seen |
| 11 | Container With Most Water | Medium | move the shorter wall |
| 18 | 4Sum | Medium | sort + two nested + two pointers |
| 75 | Sort Colors | Medium | Dutch-flag *three* pointers |
| 42 | Trapping Rain Water | Hard | two pointers + running max walls |

---

## 8. Common Pitfalls & Interview Tips

- **Forgetting the array must be sorted (opposite-ends).** The converging trick is only valid because "too big → smaller, too small → bigger" relies on order. On unsorted data it silently returns wrong answers. Say out loud: *"opposite-ends needs sorted input."* If you can't sort (e.g. plain **Two Sum** needs original indices), switch to a hash map ([03-hashing-maps-and-sets.md](03-hashing-maps-and-sets.md)).

- **`sort()` is lexicographic by default — the #1 JS/TS trap.** `[10, 2, 1].sort()` gives `[1, 10, 2]` because it sorts by **string**. Always pass a numeric comparator: `arr.sort((a, b) => a - b)`. Forgetting this quietly breaks every sorted-array pattern (see [08-sorting.md](08-sorting.md)).

- **Loop condition `left < right`, not `left <= right`.** For pair problems you need two *distinct* elements; `<=` lets a single element pair with itself. (Some in-place partition problems do use `<=` — know why yours differs.)

- **Off-by-one on the ends.** `right` starts at `arr.length - 1`, **not** `arr.length`. Reading `arr[arr.length]` gives `undefined`, and `undefined + number` is `NaN` — a bug that won't throw, just misbehaves.

- **Skipping duplicates in 3Sum/4Sum.** If you don't skip equal anchors (`nums[i] === nums[i-1]`) *and* equal inner values after a hit, you emit duplicate triplets and fail. This is the single most common reason a 3Sum solution is rejected — mention it proactively.

- **Same-direction: return `slow` vs `slow + 1`.** If `slow` points at the *last written* index, the length is `slow + 1`. If `slow` is the *next write* position, the length is `slow`. Pick one convention and stay consistent — mixing them is a classic off-by-one.

- **What to say out loud (scores communication points):** *"The array's sorted, so I'll use converging two pointers — start at both ends, move the one that brings the sum toward the target. That's O(n) time, O(1) space, versus O(n²) brute force."* Then state your loop invariant: *"Everything outside `[left, right]` has been ruled out."*

---

## 9. TL;DR

- **Two pointers = two indices moved by a rule, replacing a nested loop.** Two flavours: **opposite ends** (converge from `0` and `n-1`) and **same direction** (fast reader + slow writer).
- **Opposite ends** ⇒ sorted array, pair/triplet sum, palindrome, "most water". `O(n)` after sorting; **needs sorted data**.
- **Same direction** ⇒ in-place dedupe/partition/merge, "read vs write". `O(n)`, `O(1)` space.
- **3Sum = sort → fix one → two pointers** on the rest = `O(n²)`. Remember to **skip duplicates**.
- **JS/TS traps:** always `sort((a,b)=>a-b)`; use `left < right`; `right` starts at `length - 1`; watch `slow` vs `slow + 1`.
- Contiguous "window" version → [05-sliding-window.md](05-sliding-window.md). Unsorted "seen it?" version → [03-hashing-maps-and-sets.md](03-hashing-maps-and-sets.md). Sorting mechanics → [08-sorting.md](08-sorting.md).
