# 02. Arrays & Strings

> **One-liner:** After this you'll manipulate arrays and strings *fluently and cheaply* — knowing which operations are secretly O(n), how to modify in place, count characters, and reverse without wasting memory.
> **Pattern power:** Arrays and strings are the raw material of **~70% of all interview problems**. Almost every other pattern in this track (two pointers, sliding window, prefix sum, hashing) runs *on top of* an array or string. Master the substrate and everything above it gets easier.

You already use arrays and strings every day in React — `.map` over a list, template literals in JSX. Good news: that fluency transfers. The *new* part is thinking about **cost** and **in-place** modification, plus a few JS-specific traps that quietly turn fast code slow.

---

## 1. The Idea in Plain Words

An **array** is an ordered list of values, each reachable by its **index** — a position number starting at 0. When we say "a **pointer** into the array," in JS that just means **an index variable** like `let i = 0`; there are no C-style memory pointers here. Reading `arr[i]` is **O(1)**: the engine jumps straight to that slot, no scanning.

A **string** is a sequence of characters — basically a read-only array of characters. The critical difference: **strings in JavaScript are immutable.** You *cannot* do `str[0] = 'H'` — it silently does nothing. Any "change" to a string actually builds a brand-new string. This one fact drives half the string traps in interviews (and half the performance bugs in real apps).

Mental model: an **array is a row of numbered lockers** you can open, read, and swap contents in. A **string is that same row, but the lockers are welded shut** — to "edit" it you must build a new row.

---

## 2. When to Use It — Pattern Triggers

Arrays/strings themselves aren't a "pattern" — they're where patterns *live*. But these concrete signals tell you which **array/string technique** to reach for:

| The problem says… | Technique | Where |
|---|---|---|
| "reverse", "swap", "rearrange **in place**", "O(1) extra space" | **Two-pointer swap** (left/right walking inward) | §4 here + [04](04-two-pointers.md) |
| "count occurrences", "frequency", "anagram", "most common char" | **Frequency counting** (Map or char-code array) | §4 + [03](03-hashing-maps-and-sets.md) |
| "contiguous **subarray**", "longest/shortest window with a condition" | **Sliding window** | [05](05-sliding-window.md) |
| "sum of a range", "how many subarrays sum to K" | **Prefix sum** | [06](06-prefix-sum.md) |
| "find a pair/triplet in a **sorted** array" | **Two pointers from both ends** | [04](04-two-pointers.md) |
| "seen this value before?", "does X exist?" | **Hash Set/Map** (not `indexOf`!) | [03](03-hashing-maps-and-sets.md) |
| "build/transform a string char by char" | **Array buffer + `.join('')`** (never `+=` in a loop) | §4 + §8 |

Example phrasings you'll literally see: *"Reverse the string in place"*, *"Determine if two strings are anagrams"*, *"Move all zeroes to the end without making a copy"*, *"Return the array rotated by k"*. Each maps to a row above.

---

## 3. Picture It (Diagram)

**An array and its indices** — note the front vs. back cost asymmetry:

```
 index:   0     1     2     3     4
        +-----+-----+-----+-----+-----+
 value: |  5  |  8  |  2  |  9  |  1  |
        +-----+-----+-----+-----+-----+
           ^                       ^
       arr[0] O(1)            arr.length-1

  push/pop at the BACK  ......... O(1)  (nothing else moves)
  shift/unshift at the FRONT .... O(n)  (every element re-indexes ->)
```

**In-place reversal with two pointers** — the workhorse move. Walk `left` and `right` toward each other, swapping as you go:

```
 start:   [ h  e  l  l  o ]      swap arr[left] <-> arr[right]
            L           R
 step 1:  [ o  e  l  l  h ]
               L     R
 step 2:  [ o  l  l  e  h ]
                  LR              pointers meet/cross -> done
 result:  [ o  l  l  e  h ]       O(n) time, O(1) extra space
```

No second array allocated — we reused the same lockers. That's what "in place" means, and it's `O(1)` space.

---

## 4. The Template (TypeScript)

Four reusable skeletons you'll adapt constantly.

```ts
// ---------- 1. Straight scan (the baseline O(n) pass) ----------
function scan(arr: number[]): void {
  for (let i = 0; i < arr.length; i++) {
    const x = arr[i];
    // ...do something with x at position i...
  }
  // Prefer `for...of` when you don't need the index:
  //   for (const x of arr) { ... }
  // AVOID `for...in` on arrays: it yields string keys & inherited props.
}

// ---------- 2. In-place two-pointer swap (reverse / rearrange) ----------
function reverseInPlace<T>(arr: T[]): void {
  let left = 0;
  let right = arr.length - 1;
  while (left < right) {
    [arr[left], arr[right]] = [arr[right], arr[left]]; // destructuring swap
    left++;
    right--;
  }
}

// ---------- 3. Frequency counting ----------
// General keys -> use a Map:
function frequency<T>(items: T[]): Map<T, number> {
  const freq = new Map<T, number>();
  for (const x of items) {
    freq.set(x, (freq.get(x) ?? 0) + 1); // ?? handles the "not seen yet" (undefined) case
  }
  return freq;
}
// Lowercase-letters-only -> a 26-slot array is faster & tiny:
function letterCounts(s: string): number[] {
  const counts = new Array(26).fill(0);
  const A = "a".charCodeAt(0); // 97
  for (let i = 0; i < s.length; i++) {
    counts[s.charCodeAt(i) - A]++; // map 'a'..'z' -> 0..25
  }
  return counts;
}

// ---------- 4. Build a string the RIGHT way ----------
function buildString(chars: string[]): string {
  const buffer: string[] = [];
  for (const c of chars) {
    buffer.push(c.toUpperCase()); // push is O(1)
  }
  return buffer.join(""); // one O(n) join at the end
  // DON'T:  let out = ""; for (...) out += c;  // O(n^2) — see §8
}
```

---

## 5. Complexity

The real cost of the operations you'll use — several are traps because they read as cheap:

| Operation | Time | In place? | Note |
|---|---|---|---|
| `arr[i]` read/write | O(1) | — | direct index access |
| `arr.push(x)` / `arr.pop()` | O(1) amortized | yes | back of the array — cheap |
| `arr.shift()` / `arr.unshift(x)` | **O(n)** | yes | ⚠️ front ops re-index everything |
| `arr.splice(i, d, ...items)` | **O(n)** | yes | ⚠️ shifts the tail after `i` |
| `arr.slice(a, b)` | O(k) | no | **copies** k elements into a new array |
| `[...arr]` / `arr.concat(b)` | O(n) | no | **copies** — costly inside a loop |
| `arr.indexOf` / `includes` / `find` | **O(n)** | — | ⚠️ linear scan; a `Set` gives O(1) |
| `arr.sort((a,b)=>a-b)` | O(n log n) | yes | ⚠️ needs a comparator for numbers |
| `arr.reverse()` | O(n) | yes | built-in in-place reverse |
| `str[i]` / `str.charCodeAt(i)` | O(1) | — | strings are indexable, read-only |
| `str.slice` / `substring` | O(k) | no | new string of length k |
| `str.split("")` | O(n) | no | string → char array (then mutate the array) |
| `result += char` in a loop | **O(n²)** | no | ⚠️ immutability: each `+=` copies the whole string |

**The two headline traps:** front-of-array ops (`shift`/`unshift`/`splice(0,...)`) are `O(n)`, and string `+=` in a loop is `O(n²)`. Both are invisible until your input gets big — exactly what interviewers probe.

---

## 6. Worked Example — Valid Anagram

> **LeetCode 242 · Valid Anagram (Easy).** Given two strings `s` and `t`, return `true` if `t` is an anagram of `s` (same letters, same counts, rearranged). E.g. `s = "anagram"`, `t = "nagaram"` → `true`; `s = "rat"`, `t = "car"` → `false`.

**Understand / clarify:** *Case-sensitive?* Assume yes (lowercase only for the simple version). *Unicode or just a–z?* Ask — it changes the tool. *Different lengths?* Then it's an instant `false`.

**Brute force.** Two strings are anagrams iff they're equal *after sorting*. Sort both, compare.
```ts
function isAnagramBrute(s: string, t: string): boolean {
  if (s.length !== t.length) return false;
  const norm = (str: string) => str.split("").sort().join(""); // "anagram" -> "aaagmnr"
  return norm(s) === norm(t);
}
```
Correct, and totally acceptable to state first. Cost: **O(n log n)** time (the sort), **O(n)** space (the split arrays).

**Key insight.** We don't actually need the letters *ordered* — we only need to know **each letter appears the same number of times** in both. Counting is `O(n)`, cheaper than sorting. So: count `s`'s letters, then "spend" them against `t`. If `t` ever needs a letter that's exhausted, they're not anagrams.

**Optimal solution.**
```ts
function isAnagram(s: string, t: string): boolean {
  if (s.length !== t.length) return false; // different lengths => impossible

  // Count how many of each character s contains.
  const counts = new Map<string, number>();
  for (const ch of s) {
    counts.set(ch, (counts.get(ch) ?? 0) + 1);
  }

  // Consume one of each character as we walk t.
  for (const ch of t) {
    const c = counts.get(ch);
    if (c === undefined || c === 0) return false; // t needs a letter s doesn't have (enough of)
    counts.set(ch, c - 1);
  }

  // Lengths matched and nothing went negative => identical multisets of letters.
  return true;
}
```

If it's guaranteed lowercase `a–z`, the char-code array version is even tighter and avoids Map overhead — great to mention as a follow-up:
```ts
function isAnagramAZ(s: string, t: string): boolean {
  if (s.length !== t.length) return false;
  const freq = new Array(26).fill(0);
  const A = "a".charCodeAt(0);
  for (let i = 0; i < s.length; i++) {
    freq[s.charCodeAt(i) - A]++; // add for s
    freq[t.charCodeAt(i) - A]--; // subtract for t, in the same pass
  }
  return freq.every((f) => f === 0); // all balanced => anagram
}
```

**Trace** `s = "rat"`, `t = "car"` with the Map version:
1. Count `s`: `{ r:1, a:1, t:1 }`.
2. Walk `t`: `c` → `counts.get('c')` is `undefined` → **return `false`**. ✅ Correct — `t` has a `c` that `s` never had.

**Trace** `s = "aab"`, `t = "aba"`: counts `{a:2, b:1}`. Walk `t`: `a`→2→1, `b`→1→0, `a`→1→0. Never hit `undefined`/`0`-block, lengths equal → **`true`**. ✅

**Evaluate.** Optimal: **O(n) time** (two single passes), **O(1) space** for the 26-array version (fixed size regardless of `n`) or **O(k) space** for the Map where `k` = distinct characters. We beat the sort-based `O(n log n)` — the classic "counting beats sorting when you only need frequencies" insight. It also bridges directly into [03-hashing-maps-and-sets.md](03-hashing-maps-and-sets.md).

---

## 7. Practice Set (curated LeetCode)

Easy → hard. Each teaches an array/string reflex you'll reuse everywhere.

| # | Problem | Difficulty | What it teaches / the trigger |
|---|---|---|---|
| 344 | Reverse String | Easy | In-place two-pointer swap; why strings need a char array first |
| 217 | Contains Duplicate | Easy | "seen before?" → `Set`, not `indexOf` (bridge to [03](03-hashing-maps-and-sets.md)) |
| 242 | Valid Anagram | Easy | Frequency counting; counting beats sorting |
| 125 | Valid Palindrome | Easy | Two pointers + filtering non-alphanumerics; `charCodeAt` |
| 283 | Move Zeroes | Easy | In-place **write pointer** — rearrange without a copy |
| 121 | Best Time to Buy & Sell Stock | Easy | Single pass tracking a running minimum |
| 14 | Longest Common Prefix | Easy | Column-wise string scan; early exit |
| 189 | Rotate Array | Medium | The reversal trick; in-place with O(1) space |
| 238 | Product of Array Except Self | Medium | Prefix/suffix products, no division (bridge to [06](06-prefix-sum.md)) |
| 49 | Group Anagrams | Medium | Canonical-key grouping with a `Map` (bridge to [03](03-hashing-maps-and-sets.md)) |
| 53 | Maximum Subarray | Medium | Kadane's running sum (bridge to DP, [18](18-dynamic-programming-1.md)) |

Do 344, 217, 242 today — they cement in-place swapping, set membership, and frequency counting, the three reflexes the rest of the track leans on.

---

## 8. Common Pitfalls & Interview Tips

**JS/TS-specific traps (these fail real interviews):**

- **`shift` / `unshift` / `splice(0, …)` are O(n).** Doing them inside a loop makes your solution `O(n²)` by accident. Building a result? `push` to the end and `reverse()` once, or use a write-pointer index.
- **Building strings with `+=` in a loop is O(n²).** Strings are immutable, so each `+=` copies the whole thing. Collect chars in an array, then `arr.join("")` once (O(n)).
- **`sort()` is lexicographic.** `[10, 9, 100, 2].sort()` → `[10, 100, 2, 9]`. For numbers always `.sort((a, b) => a - b)`. This is the single most common JS DSA bug.
- **`slice` vs `splice`.** `slice` **copies** and doesn't mutate; `splice` **mutates** in place and returns removed items. Mixing them up corrupts data or wastes memory.
- **Sparse arrays.** `new Array(5)` makes 5 *empty holes*, not zeros — `.map` skips them. Use `new Array(5).fill(0)` or `Array.from({length: 5}, () => 0)`.
- **`arr[-1]` is `undefined`,** not the last element (no negative indexing like Python). Use `arr.at(-1)` or `arr[arr.length - 1]`.
- **`for...in` on an array** iterates **string** keys and inherited properties — a real bug source. Use `for...of` (values) or a classic `for` (indices).
- **`===` on arrays compares references, not contents.** `[1,2] === [1,2]` is `false`. Compare element-by-element or via a serialized key.
- **`charCodeAt` returns `NaN`** for out-of-range indices. Remember `'a'` = 97, `'A'` = 65, `'0'` = 48. Zero-base a lowercase letter with `ch.charCodeAt(0) - 97`.
- **Hidden copies.** `[...arr]`, `slice`, `concat`, `str.split("")` each allocate O(n). One is fine; one *per loop iteration* is an accidental `O(n²)`.

**What to say out loud:**
- "I'll swap in place with two pointers, so this is `O(1)` extra space."
- "I'm avoiding `shift` here because it's `O(n)` — I'll use a write pointer instead."
- "Strings are immutable in JS, so I'll build into an array and `join` at the end to stay `O(n)`."

Naming these unprompted signals senior-level awareness — exactly what CRED/Razorpay-style "clean code + edge cases" interviews reward.

---

## 9. TL;DR

- **Array** = numbered lockers, `arr[i]` is O(1). **String** = welded-shut lockers: **immutable**, so every "edit" builds a new string.
- **Back is cheap, front is expensive:** `push`/`pop` = O(1); `shift`/`unshift`/`splice` = **O(n)**.
- **Never `+=` strings in a loop** (O(n²)) — push chars to an array and `.join("")` once.
- **Always `.sort((a,b)=>a-b)`** for numbers; the default is string order.
- Core reflexes: **in-place two-pointer swap** (O(1) space), **frequency counting** (Map or 26-array), **char-code math** (`charCodeAt`/`fromCharCode`).
- `indexOf`/`includes` are O(n) — when you're asking "seen before?", reach for a `Set` instead → [03-hashing-maps-and-sets.md](03-hashing-maps-and-sets.md).
- Foundations recap and complexity rules live in [01-big-o-and-problem-solving-framework.md](01-big-o-and-problem-solving-framework.md). Next patterns that run on arrays: [04-two-pointers.md](04-two-pointers.md), [05-sliding-window.md](05-sliding-window.md), [06-prefix-sum.md](06-prefix-sum.md).
