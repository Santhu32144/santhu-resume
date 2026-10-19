# 03. Hashing: Maps & Sets

> **One-liner:** After this you'll reach for a `Map` or `Set` on instinct to turn slow `O(n²)` "search the array again and again" solutions into fast `O(n)` ones — the single most reusable move in coding interviews.
> **Pattern power:** Hashing is the **most-used tool in the entire interview canon.** "Seen before?", "count of", "group by", "does a complement exist?" — it unlocks a huge fraction of Easy/Medium problems outright and is a building block inside sliding window, prefix sum, graphs, and more.

If you learn one pattern deeply before your first interview, make it this one. It's also the most *intuitive* for a JS dev: you already use objects as lookup tables every day. We're just going to do it deliberately, with the right tool, and know exactly why it's fast.

---

## 1. The Idea in Plain Words

A **hash map** is a dictionary: you store **key → value** pairs and look any key up in roughly **one step**, no matter how many entries it holds. A **hash set** is the same machine minus the values — a bag of **unique keys** that answers "**is X in here?**" in one step. In TypeScript these are the built-in `Map` and `Set`.

How is lookup instant? A **hash function** turns your key into a number (a "hash"), which points to a **bucket** (a slot in an internal array). To find a key you re-hash it and jump straight to its bucket — no scanning the whole collection. That's why `map.get(k)` and `set.has(k)` are **O(1) on average**, versus `arr.indexOf(k)` which is **O(n)** (it walks the whole array).

**The one big idea: trade space for time.** You spend `O(n)` extra memory to remember things, and in return you turn repeated `O(n)` searches into `O(1)` lookups — collapsing an `O(n²)` brute force into `O(n)`. That trade is the heartbeat of this chapter.

*A Habex analogy that genuinely fits:* to show a running total per category, you don't re-scan every expense once per category (that's `O(categories × expenses)`). You keep one `Map<category, total>` and update it in a single pass over expenses — `O(expenses)`. Same answer, dramatically less work. That's a frequency map, the exact pattern below.

---

## 2. When to Use It — Pattern Triggers

Hashing is the answer astonishingly often. Train your ear for these signals:

| The problem says / implies… | Why hashing wins | Concrete phrasing you'll see |
|---|---|---|
| "have I **seen** this before?" | O(1) membership vs O(n) scan | "find the duplicate", "is there a cycle of values" |
| "**count** / frequency of" | one pass builds counts | "most frequent element", "first non-repeating char" |
| "**group by** some key" | bucket items under a computed key | "group anagrams", "group by domain" |
| "**dedupe** / distinct" | `Set` drops repeats for free | "number of unique …", "remove duplicates" |
| "does a **complement / pair** exist?" | store what you've seen, look up the partner | "two sum", "pair with difference k" |
| "**map** one thing to another / bijection" | key→value relationship | "isomorphic strings", "word pattern" |
| "constant-time **membership** in a set" | precompute a lookup set | "is this a valid word?", "is it a vowel?" |

Rule of thumb: **the moment you notice yourself wanting to search the array again from the start, stop — a hash map/set almost certainly removes that inner search.** This is the [README trigger map](00-README.md) row *"seen before?, count of, group by, dedupe → Hash Map/Set."*

**When hashing does *not* help** (equally important to say out loud): when you need **sorted order**, the **nearest/closest** value, or **range** queries — hashing scatters keys, so it can't answer those. Then reach for **sorting** ([08](08-sorting.md)), **two pointers** ([04](04-two-pointers.md)), **binary search** ([07](07-binary-search.md)), or **trees** ([13](13-binary-search-trees.md)). And when the interviewer demands **O(1) extra space**, a hash map (O(n) space) may be disallowed.

---

## 3. Picture It (Diagram)

**Keys hashing into buckets** — why lookup skips the scan:

```
  key "swiggy"  --hash()-->  3
  key "zepto"   --hash()-->  0        buckets (internal array)
  key "cred"    --hash()-->  1     +---+------------------+
                                   | 0 | zepto -> ...     |
  get("cred"):  hash("cred")=1     | 1 | cred  -> ...     |  <-- jump straight here
  jump to bucket 1, done. O(1).    | 2 |                  |
                                   | 3 | swiggy -> ...    |
                                   +---+------------------+
```

**Building a frequency map** in one pass over `["a","b","a","c","b","a"]`:

```
 read 'a' -> {a:1}
 read 'b' -> {a:1, b:1}
 read 'a' -> {a:2, b:1}
 read 'c' -> {a:2, b:1, c:1}
 read 'b' -> {a:2, b:2, c:1}
 read 'a' -> {a:3, b:2, c:1}      one O(n) pass, counts ready to query in O(1)
```

---

## 4. The Template (TypeScript)

The four hashing skeletons that cover most problems.

```ts
// ---------- 1. Frequency map: count occurrences of anything ----------
function buildFrequency<T>(items: T[]): Map<T, number> {
  const freq = new Map<T, number>();
  for (const x of items) {
    // get() returns undefined for a missing key; ?? 0 seeds the first count.
    freq.set(x, (freq.get(x) ?? 0) + 1);
  }
  return freq;
}

// ---------- 2. Seen-set: membership / dedupe / "have I seen it?" ----------
function firstDuplicate(nums: number[]): number | undefined {
  const seen = new Set<number>();
  for (const n of nums) {
    if (seen.has(n)) return n; // O(1) membership check
    seen.add(n);
  }
  return undefined;
}

// ---------- 3. Group-by: bucket items under a computed key ----------
function groupBy<T>(items: T[], keyOf: (item: T) => string): Map<string, T[]> {
  const groups = new Map<string, T[]>();
  for (const item of items) {
    const key = keyOf(item);
    if (!groups.has(key)) groups.set(key, []); // create the bucket once
    groups.get(key)!.push(item);               // "!": we just ensured it exists
  }
  return groups;
}

// ---------- 4. Complement / seen-before: build the answer as you scan ----------
function hasPairWithSum(nums: number[], target: number): boolean {
  const seen = new Set<number>();
  for (const n of nums) {
    if (seen.has(target - n)) return true; // did the partner already go by?
    seen.add(n);
  }
  return false;
}
```

> **Idiom to internalize:** `map.set(k, (map.get(k) ?? 0) + 1)` is *the* counter increment. And `if (!map.has(k)) map.set(k, [])` then `map.get(k)!.push(x)` is *the* group-append. You'll type these hundreds of times.

---

## 5. Complexity

| Operation | Average | Worst case | Note |
|---|---|---|---|
| `map.get(k)` / `set(k, v)` / `has(k)` / `delete(k)` | **O(1)** | O(n) | worst case = many hash collisions (rare; adversarial) |
| `set.has(x)` / `add(x)` / `delete(x)` | **O(1)** | O(n) | the fast membership test |
| Iterating a `Map`/`Set` of n items | O(n) | O(n) | preserves **insertion order** |
| Building a map/set of n items | O(n) | — | n inserts × O(1) |
| **Space for a map/set of n items** | O(n) | — | this is the memory you "spend" to buy speed |

Say **"O(1) average"** in interviews — it's precise and signals you know collisions exist. The whole value proposition: **+O(n) space ⇒ −O(n) time factor** (turning `O(n²)` into `O(n)`).

---

## 6. Worked Example — Two Sum, the canonical space-for-time win

> **LeetCode 1 · Two Sum (Easy).** Given `nums: number[]` and `target: number`, return the indices of the two numbers that sum to `target`. Exactly one solution; can't reuse an element.

**Brute force.** Try every pair:
```ts
function twoSumBrute(nums: number[], target: number): number[] {
  for (let i = 0; i < nums.length; i++) {
    for (let j = i + 1; j < nums.length; j++) {
      if (nums[i] + nums[j] === target) return [i, j];
    }
  }
  return [];
}
```
**O(n²)** time. The waste: for every `i`, the inner loop **re-scans** the array hunting for `target - nums[i]`. That repeated search is exactly what a hash map deletes.

**Key insight.** As I walk the array, for the current number `x` I need its **complement** `target - x`. If I've stored every earlier number in a `Map` (value → its index), I can ask "did the complement already go by?" in **O(1)**. No inner loop.

**Optimal solution.**
```ts
function twoSum(nums: number[], target: number): number[] {
  const seen = new Map<number, number>(); // value -> index it appeared at

  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];

    if (seen.has(complement)) {            // O(1): partner already recorded?
      return [seen.get(complement)!, i];   // "!": has() guaranteed it exists
    }
    seen.set(nums[i], i);                   // record current AFTER checking,
    // ...so we never pair an element with itself.
  }
  return [];
}
```

**Trace** `nums = [3, 2, 4]`, `target = 6`:

| `i` | `nums[i]` | `complement` | `seen` before | has complement? | action |
|---|---|---|---|---|---|
| 0 | 3 | 3 | `{}` | no | set `3 → 0` |
| 1 | 2 | 4 | `{3:0}` | no | set `2 → 1` |
| 2 | 4 | 2 | `{3:0, 2:1}` | **yes** (idx 1) | return `[1, 2]` ✅ |

Notice we correctly *don't* pair the `3` with itself: at `i=0` its complement `3` wasn't stored yet, because we check *before* we insert.

**Evaluate.** **O(n)** time (one pass, O(1) map ops), **O(n)** space (the map). We spent memory to erase the inner loop — `O(n²) → O(n)`. This *is* the space-for-time trade in its purest form.

**Same idea, one level up — Group Anagrams (LeetCode 49).** Anagrams share a **canonical signature** (their letters sorted). Use that signature as a map key and bucket words under it — a group-by:
```ts
function groupAnagrams(strs: string[]): string[][] {
  const groups = new Map<string, string[]>();
  for (const s of strs) {
    const key = s.split("").sort().join(""); // "eat","tea","ate" -> "aet"
    if (!groups.has(key)) groups.set(key, []);
    groups.get(key)!.push(s);
  }
  return [...groups.values()];
}
```
`["eat","tea","tan","ate","nat","bat"]` → `[["eat","tea","ate"], ["tan","nat"], ["bat"]]`. Time `O(n · k log k)` (k = word length, for the per-word sort); space `O(n · k)`. Whenever you think *"things that share a property belong together,"* that's a hash-map group-by.

---

## 7. Practice Set (curated LeetCode)

Easy → hard. This set alone covers a big slice of what Zepto/Swiggy/Groww ask.

| # | Problem | Difficulty | What it teaches / the trigger |
|---|---|---|---|
| 217 | Contains Duplicate | Easy | `Set` membership — the "seen before?" reflex |
| 1 | Two Sum | Easy | Complement map — the canonical space-for-time win |
| 242 | Valid Anagram | Easy | Frequency map; counting beats sorting |
| 349 | Intersection of Two Arrays | Easy | Set intersection; dedupe both sides |
| 387 | First Unique Character in a String | Easy | Frequency map, then a second pass to find count 1 |
| 205 | Isomorphic Strings | Easy | Two maps enforcing a consistent bijection |
| 49 | Group Anagrams | Medium | Group-by with a canonical signature key |
| 347 | Top K Frequent Elements | Medium | Frequency map + bucketing (bridge to heaps, [14](14-heaps-and-priority-queues.md)) |
| 560 | Subarray Sum Equals K | Medium | Prefix-sum counts in a map (bridge to [06](06-prefix-sum.md)) |
| 128 | Longest Consecutive Sequence | Medium | `Set` + O(n) sequence walk; the clever "start of a run" trick |

Start with 217 → 1 → 242. If you can write those three cold with the templates in §4, hashing has clicked.

---

## 8. Common Pitfalls & Interview Tips

**Why `Map`/`Set` over a plain object for DSA (say this if asked):**

- **Object keys are coerced to strings.** `obj[1]` and `obj["1"]` are the *same* key; `obj[true]` becomes `"true"`. A `Map` keeps keys as their real type (number, boolean, even object references).
- **Prototype-key landmines.** `"constructor" in {}` is `true`, and `"__proto__"`, `"hasOwnProperty"`, `"toString"` can collide with inherited members or corrupt the object. A `Map` has none of this. (If you *must* use an object as a dictionary, create it with `Object.create(null)` and read via `Object.hasOwn(obj, k)`.)
- **`Map` knows its size** (`map.size`) and **preserves insertion order** reliably; object integer-like keys get reordered numerically (`{2:.., 1:..}` iterates as `1, 2`). Insertion order matters for things like LRU caches and "first unique."
- **`Map` is built for frequent add/delete** and any key type. For DSA, default to `Map`/`Set`. Reserve plain objects for fixed, known string keys.

**Correctness traps:**

- **`map.get(k)` returns `undefined` for missing keys** — always seed counters with `?? 0`: `map.set(k, (map.get(k) ?? 0) + 1)`.
- **Truthiness bug:** `if (map.get(k))` is wrong when the stored value can legitimately be `0`, `""`, or `false` — those are falsy but *present*. Use `map.has(k)` or `map.get(k) !== undefined` to test existence.
- **Objects as keys compare by reference.** `new Set([{id:1}, {id:1}])` has size **2** (two different object references). To dedupe by value, key on a **string signature** (e.g. `JSON.stringify` or a joined field), not the object.
- **`NaN` as a key works** in `Map`/`Set` (they use SameValueZero, treating `NaN` equal to itself) even though `NaN === NaN` is `false`. Occasionally handy, occasionally surprising.
- **Worst-case O(n) collisions** exist but are effectively never the point in interviews — quote "O(1) average" and move on unless asked about adversarial hashing.
- **Don't forget the space cost.** Hashing is O(n) memory. If the problem insists on O(1) space, pivot to sorting + two pointers ([04](04-two-pointers.md)) or in-place tricks.

**What to say out loud:** "A brute force re-searches the array — O(n²). I'll trade O(n) space for a hash map so each lookup is O(1), bringing it to O(n). I'm using a `Map` rather than an object to avoid string-coercion and prototype-key issues, and I'll test with `.has()` since a stored value could be 0." That sentence alone lands the pattern *and* the JS depth interviewers probe for.

---

## 9. TL;DR

- A **`Map`** is a key→value dictionary; a **`Set`** is a bag of unique keys. Both give **O(1)-average** `get`/`set`/`has` — versus `indexOf`'s O(n).
- **The move: trade O(n) space for time**, collapsing "search again and again" `O(n²)` into `O(n)`.
- **Triggers:** seen-before, count/frequency, group-by, dedupe, complement/pair, key→value mapping, constant-time membership.
- Four templates: **frequency map**, **seen-set**, **group-by**, **complement scan**. Memorize `map.set(k, (map.get(k) ?? 0) + 1)`.
- **Prefer `Map`/`Set` over plain objects** for DSA: any key type, real `size`, insertion order, no prototype/string-coercion bugs.
- Test existence with **`.has()`**, not truthiness (a stored `0`/`false`/`""` is still present).
- **Hashing can't do order/range/nearest** — for those use sorting ([08](08-sorting.md)), two pointers ([04](04-two-pointers.md)), binary search ([07](07-binary-search.md)), or trees ([13](13-binary-search-trees.md)).
- Built on the fundamentals in [01-big-o-and-problem-solving-framework.md](01-big-o-and-problem-solving-framework.md) and the array/string reflexes in [02-arrays-and-strings.md](02-arrays-and-strings.md).
