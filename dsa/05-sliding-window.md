# 05. Sliding Window

> **One-liner:** Keep a **contiguous window** `[left, right]` over an array/string, **expand** the right edge to include new elements and **shrink** the left edge when a rule breaks — so you scan the whole thing **once** instead of re-checking every subarray.
> **Pattern power:** Collapses `O(n²)`/`O(n³)` "check every substring/subarray" problems into `O(n)`. This is *the* pattern Zepto and Swiggy love ("longest substring…", "at most K…", "max sum window") — recognise it and you'll one-shot a whole class of mediums.

---

## 1. The Idea in Plain Words

A **window** is just a **contiguous slice** of the array/string — everything between two indices, `left` and `right`, inclusive. "Sliding" it means moving those edges rather than starting over each time.

Sliding window is a **specialised same-direction two-pointer** (you read [04-two-pointers.md](04-two-pointers.md) first — this is its most important child). Both pointers only ever move **forward**. The difference from generic two-pointers is what you carry: a window keeps some **running state** about the elements *currently inside it* — a sum, a character count, a `Set` of what's present.

The brute force for "best contiguous subarray of some kind" is to try every start and every end:

```ts
// O(n²) windows, and often O(n) work each → O(n³). Way too slow.
for (let start = 0; start < n; start++) {
  for (let end = start; end < n; end++) {
    // examine subarray [start..end]
  }
}
```

The insight: when you move `end` forward by one, the new window is *almost* the old window — only **one element entered**. And when you move `start` forward, only **one element left**. So instead of recomputing from scratch, you **update the running state incrementally** (`+ arr[right]` when it enters, `- arr[left]` when it leaves). Each element enters once and leaves once ⇒ `O(n)` total.

Two shapes of window:

| Window type | Size | You control the window by… | Typical question |
|---|---|---|---|
| **Fixed** | constant `k` | sliding a `k`-wide frame one step at a time | "max sum of any subarray of size `k`" |
| **Variable** | grows/shrinks | expanding `right` always; shrinking `left` when a rule breaks | "longest/shortest subarray satisfying a condition" |

> In **Habex**, imagine a "7-day rolling spend" number. Each new day you *add* today's expense and *drop* the day that's now 8 days old — you never re-sum all 7 days. That's a **fixed** window. A **variable** window is more like "the longest recent streak of days where total spend stayed under ₹5000": you keep extending the streak until you bust the budget, then trim old days off the front until you're back under.

---

## 2. When to Use It — Pattern Triggers

The tell is almost always the word **contiguous** (a *subarray* or *substring*, i.e. no gaps) plus an **optimisation or constraint**.

**Fixed-window triggers:**
- The window size is **given**: *"subarray of size `k`"*, *"every window of `k` days"*, *"average of `k` consecutive"*.
- *"Maximum/minimum sum of a subarray of length `k`"*, *"find all anagrams of `p` in `s`"* (window = `p.length`).

**Variable-window triggers (the money ones):**
- *"**Longest** substring/subarray such that <condition>"* → expand greedily, shrink when the condition breaks, track the max length.
- *"**Shortest** / **minimum length** subarray such that <condition holds>"* → expand until valid, then shrink while still valid, track the min length.
- *"**at most K** distinct / zeros / replacements"* → the constraint you shrink against.
- *"longest substring **without repeating** characters"*, *"minimum window substring containing all of T"*.

**Example phrasings that mean "sliding window":**
> "Find the length of the **longest substring without repeating characters**."
> "Return the **minimum length** of a **contiguous** subarray whose sum is `≥ target`."
> "Given `k`, find the **maximum sum of any contiguous subarray of size k**."
> "Longest subarray with **at most K** distinct integers."

**Trigger vs. sibling patterns:**
- Contiguous + a running condition ⇒ **sliding window** (this chapter).
- Two ends of a **sorted** array converging ⇒ **two pointers** ([04-two-pointers.md](04-two-pointers.md)).
- "Does this exact thing exist / how many times" with **no contiguity** ⇒ **hash map/set** ([03-hashing-maps-and-sets.md](03-hashing-maps-and-sets.md)) — which sliding window also *uses* to hold window state.
- Range sums or "how many subarrays sum to K" **with negatives** ⇒ **prefix sum** ([06-prefix-sum.md](06-prefix-sum.md)), because a window can't safely shrink when adding elements might *decrease* the sum.

> ⚠️ **Sliding window needs monotonic behaviour.** Growing the window must move you *one way* on the metric (e.g. adding non-negative numbers only *increases* a sum), so that shrinking is a valid "undo". If numbers can be negative and you're counting subarrays that sum to K, shrinking logic breaks → use prefix sum instead.

---

## 3. Picture It (Diagram)

**Fixed window**, `k = 3`, array `[2, 1, 5, 1, 3, 2]`, tracking window sum:

```
[ 2  1  5 ] 1  3  2      window = 2+1+5 = 8
  └───────┘
   2 [ 1  5  1 ] 3  2     slide: -2 (left leaves), +1 (right enters) → 8-2+1 = 7
     └───────┘
   2  1 [ 5  1  3 ] 2     -1 +3 → 7-1+3 = 9   ← best so far
        └───────┘
   2  1  5 [ 1  3  2 ]    -5 +2 → 9-5+2 = 6
           └───────┘
max window sum = 9
```

Notice we never re-add all three numbers — each slide is one subtraction + one addition.

**Variable window** — longest substring without repeats in `"abcabcbb"`. `right` always advances; when a duplicate enters, `left` jumps past the previous copy:

```
a b c a b c b b
L R                 window "a"     len 1
L   R               window "ab"    len 2
L     R             window "abc"   len 3   ← best
  L     R           'a' repeats → jump L past old 'a' → "bca"  len 3
    L     R         'b' repeats → jump L → "cab"  len 3
      L     R       'c' repeats → jump L → "abc"  len 3
            L R     'b' repeats → jump L → "cb"   len 2
              L R   'b' repeats → jump L → "b"    len 1
longest = 3
```

The window is a **caterpillar**: the head (`right`) reaches forward, the tail (`left`) catches up only when it must.

---

## 4. The Template (TypeScript)

### Template A — Fixed window (size `k`)

```ts
/** Max sum of any contiguous subarray of size k. */
function fixedWindow(arr: number[], k: number): number {
  let windowSum = 0;
  let best = -Infinity;

  for (let right = 0; right < arr.length; right++) {
    windowSum += arr[right];              // 1. element ENTERS on the right

    if (right >= k - 1) {                 // 2. window is exactly size k now
      best = Math.max(best, windowSum);   //    record the answer for this window
      windowSum -= arr[right - (k - 1)];  // 3. element LEAVES on the left
    }
  }
  return best;
}
```

### Template B — Variable window, **LONGEST** valid

```ts
/**
 * Longest window satisfying some condition.
 * Rule of thumb: shrink WHILE the window is INVALID, record AFTER it's valid.
 */
function longestWindow(arr: number[]): number {
  let left = 0;
  let best = 0;
  // ...window state here (a Map count, a Set, a running sum, etc.)

  for (let right = 0; right < arr.length; right++) {
    // 1. expand: fold arr[right] into the window state

    while (/* window is INVALID */ false) {
      // 2. shrink from the left until it's valid again
      //    remove arr[left] from the window state
      left++;
    }

    // 3. here the window [left..right] is VALID → update the answer
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```

### Template C — Variable window, **SHORTEST** valid

```ts
/**
 * Shortest window satisfying a condition (e.g. sum >= target).
 * Mirror image: shrink WHILE the window is VALID, record DURING shrinking.
 */
function shortestWindow(nums: number[], target: number): number {
  let left = 0;
  let sum = 0;
  let best = Infinity;

  for (let right = 0; right < nums.length; right++) {
    sum += nums[right];                       // expand

    while (sum >= target) {                   // window VALID → try to make it smaller
      best = Math.min(best, right - left + 1);
      sum -= nums[left];                      // element leaves
      left++;
    }
  }
  return best === Infinity ? 0 : best;        // 0 if no valid window ever formed
}
```

> **The one distinction to memorise:** *Longest* → `while (invalid) shrink;` then record. *Shortest* → `while (valid) { record; shrink; }`. Get these two skeletons cold and most window problems are just "what's the state and what makes it invalid?"

---

## 5. Complexity

| Variant | Time | Space | Why |
|---|---|---|---|
| Fixed window | `O(n)` | `O(1)` | one pass; add-one/remove-one per step |
| Variable window (Set/sum state) | `O(n)` | `O(1)`–`O(k)` | `right` moves `n` times; `left` moves at most `n` times **total** across the whole run |
| Variable window with a char/count **Map** | `O(n)` | `O(min(n, alphabet))` | Map holds at most one entry per distinct char |

The subtle bit: the inner `while (shrink)` loop looks like it could make things `O(n²)`, but it **can't** — `left` only ever moves forward and never past `right`, so over the *entire* algorithm it advances at most `n` times. That's the **amortised** argument, and it's a great thing to state in an interview.

---

## 6. Worked Example — Longest Substring Without Repeating Characters (LeetCode 3)

> **Problem:** Given a string `s`, return the length of the **longest substring** with **no repeating characters**. E.g. `"abcabcbb"` → `3` (`"abc"`).

### Brute force (the baseline)

Generate every substring and check each for duplicates: `O(n²)` substrings × `O(n)` duplicate-check = `O(n³)`. Correct but hopeless for real input. The phrase *"longest substring such that <condition>"* is a textbook **variable window** trigger.

### Key insight

Keep a window that always holds **only unique characters**. Extend `right` to pull in the next character. If that character is **already inside the window**, slide `left` forward until the window is duplicate-free again. The window is valid at the end of every step, so we record `right - left + 1` each time.

To shrink in `O(1)` instead of stepping `left` one char at a time, store each character's **last-seen index** in a `Map`. When a repeat enters, jump `left` to `lastSeen + 1` — but only if that previous copy is **inside** the current window (guard with `>= left`, or old positions from before `left` will wrongly yank it backward).

### Optimal (commented TypeScript)

```ts
function lengthOfLongestSubstring(s: string): number {
  const lastSeen = new Map<string, number>(); // char -> most recent index
  let left = 0;
  let best = 0;

  for (let right = 0; right < s.length; right++) {
    const ch = s[right];

    // If ch was seen AND that sighting is inside the current window,
    // jump left just past it so the window stays duplicate-free.
    const prev = lastSeen.get(ch);
    if (prev !== undefined && prev >= left) {
      left = prev + 1;
    }

    lastSeen.set(ch, right);                 // record/refresh ch's position
    best = Math.max(best, right - left + 1); // window is valid here
  }
  return best;
}
```

*Simpler `Set` variant* (steps `left` one at a time — easier to reason about, same `O(n)`):

```ts
function lengthOfLongestSubstringSet(s: string): number {
  const inWindow = new Set<string>();
  let left = 0;
  let best = 0;

  for (let right = 0; right < s.length; right++) {
    while (inWindow.has(s[right])) {  // shrink until s[right] is no longer a dup
      inWindow.delete(s[left]);
      left++;
    }
    inWindow.add(s[right]);
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```

### Trace — `s = "abcabcbb"` (Map version)

| `right` | `ch` | `prev` (last-seen) | `prev >= left`? | `left` after | window | `best` |
|---|---|---|---|---|---|---|
| 0 | a | — | — | 0 | `a` | 1 |
| 1 | b | — | — | 0 | `ab` | 2 |
| 2 | c | — | — | 0 | `abc` | **3** |
| 3 | a | 0 | `0 >= 0` ✓ | 1 | `bca` | 3 |
| 4 | b | 1 | `1 >= 1` ✓ | 2 | `cab` | 3 |
| 5 | c | 2 | `2 >= 2` ✓ | 3 | `abc` | 3 |
| 6 | b | 4 | `4 >= 3` ✓ | 5 | `cb` | 3 |
| 7 | b | 6 | `6 >= 5` ✓ | 7 | `b` | 3 |

Answer: **3**.

**Why the `prev >= left` guard matters** — trace `"abba"`: at `right = 3` (`'a'`), `lastSeen['a'] = 0`, but `left` is already `2`. Since `0 >= 2` is **false**, we correctly *do not* drag `left` back to `1`. The answer stays `2` (`"ab"` or `"ba"`). Drop the guard and you'd get a wrong, shrinking window — this is the classic bug.

### Two must-know cousins

**Maximum Sum Subarray of Size K** (fixed) — Template A verbatim: add `arr[right]`, and once the window hits size `k`, record the max and subtract `arr[right - k + 1]`.

**Minimum Size Subarray Sum (LeetCode 209)** (shortest variable) — Template C verbatim: grow `sum` until `sum >= target`, then shrink from the left while it stays `>= target`, tracking the smallest window length.

---

## 7. Practice Set (curated LeetCode)

| # | Name | Difficulty | Trigger it teaches |
|---|---|---|---|
| 643 | Maximum Average Subarray I | Easy | fixed window of size `k` |
| 1456 | Max Vowels in a Substring of Given Length | Easy | fixed window + running count |
| 219 | Contains Duplicate II | Easy | window of size `k` + Set |
| 3 | Longest Substring Without Repeating Characters | Medium | variable window + last-seen Map |
| 209 | Minimum Size Subarray Sum | Medium | **shortest** window ≥ target |
| 1004 | Max Consecutive Ones III | Medium | "at most K zeros" flipped |
| 424 | Longest Repeating Character Replacement | Medium | window − maxFreq ≤ K |
| 904 | Fruit Into Baskets | Medium | at most 2 distinct |
| 567 | Permutation in String | Medium | fixed window + freq match |
| 438 | Find All Anagrams in a String | Medium | fixed window + freq match |
| 340 | Longest Substring with At Most K Distinct | Medium | "at most K distinct" |
| 239 | Sliding Window Maximum | Hard | fixed window + monotonic deque |
| 76 | Minimum Window Substring | Hard | variable window + need/have counts |

---

## 8. Common Pitfalls & Interview Tips

- **Knowing *when* to shrink (longest vs shortest).** *Longest*: shrink **while invalid**, then record. *Shortest*: shrink **while valid**, recording as you go. Swapping these is the most common logic error — say which one you're doing before you code.

- **Updating the answer at the wrong moment.** In the *longest* template, record **after** the shrink loop (window is guaranteed valid). In the *shortest* template, record **inside** the shrink loop (before you remove the element that keeps it valid). Recording in the wrong place gives answers off by exactly one window.

- **Window length is `right - left + 1`.** The `+ 1` (both ends inclusive) is a perennial off-by-one. For a window that must *stay* size `k`, the trigger is `right >= k - 1`, and the element leaving is at index `right - k + 1`.

- **The last-seen Map + the `>= left` guard.** When you jump `left` using a stored index, only jump if that index is **inside the current window** (`prev >= left`). Otherwise stale positions from before `left` drag the window backward. If in doubt, use the `Set` version that steps `left` one at a time — same complexity, harder to get wrong. (Map/Set fundamentals: [03-hashing-maps-and-sets.md](03-hashing-maps-and-sets.md).)

- **Negatives break the shrink invariant.** Sliding window assumes adding an element pushes the metric monotonically (e.g. non-negative numbers only grow a sum). With negative numbers, "sum ≥ target" can flip back and forth, so shrinking isn't a clean undo → use **prefix sum + hash map** instead ([06-prefix-sum.md](06-prefix-sum.md)).

- **`s[i]` returns a string, not a char code.** In JS/TS there's no `char` type — `s[i]` is a length-1 string, fine as a `Map`/`Set` key. If you prefer a fixed `number[26]` frequency array, convert with `s.charCodeAt(i) - 97`.

- **What to say out loud:** *"'Longest contiguous substring with a condition' → variable sliding window. I expand `right` every step; when the window breaks the rule I shrink `left`. Each pointer moves forward at most n times, so it's O(n) time, O(k) space — versus O(n³) brute force."* Naming the amortised `O(n)` argument scores real points.

---

## 9. TL;DR

- **Sliding window = a forward-only two-pointer that maintains running state over a contiguous slice.** Expand `right`; shrink `left` when a rule says so. Each element enters once and leaves once ⇒ **`O(n)`**.
- **Fixed window** (size `k` given): add the entering element, subtract the leaving one, record once the window is size `k`.
- **Variable window:** *Longest* → `while (invalid) shrink;` record after. *Shortest* → `while (valid) { record; shrink; }`.
- **Window length is `right - left + 1`.** Watch this off-by-one.
- Use a **last-seen `Map`** to jump `left` in `O(1)` — but guard with `prev >= left`. Simpler mental model: a `Set` that steps `left` one at a time.
- **Negatives?** Window shrinking breaks — switch to **prefix sum** → [06-prefix-sum.md](06-prefix-sum.md). Window state lives in **maps/sets** → [03-hashing-maps-and-sets.md](03-hashing-maps-and-sets.md). Parent pattern → [04-two-pointers.md](04-two-pointers.md).
