# 20. Greedy & Intervals

> **One-liner:** A **greedy** algorithm makes the choice that looks best *right now* and never looks back — and the **intervals** pattern is the most common place it pays off: sort the ranges, then sweep through them once, merging or scheduling as you go.
> **Pattern power:** When greedy is *valid*, it's the cheapest tool you own — often just a sort plus one O(n) pass, no table like [DP](18-dynamic-programming-1.md) needed. Interval problems (merge, insert, non-overlapping, meeting rooms) are a whole genre of interview question that yields to this single "sort-then-sweep" move. The hard part — and the interview skill — is knowing *when greedy is right versus when it silently gives a wrong answer.*

---

Greedy is the mirror image of [Dynamic Programming](18-dynamic-programming-1.md). DP is careful: it explores every choice and *remembers* results because a choice now might hurt you later. Greedy is bold: it **commits to the locally-best option and never reconsiders.** When that boldness is *justified*, greedy is dramatically simpler and faster. When it isn't, greedy confidently returns garbage. This chapter teaches you both the technique and — more importantly — the judgment to tell the two apart.

---

## 1. The Idea in Plain Words

> **Greedy = at each step, grab the choice that looks best *locally*, commit to it, and never undo it — hoping the locally-best choices add up to the globally-best answer.**

The catch is in the word *hoping*. Sometimes local best = global best (greedy works, beautifully). Sometimes a small sacrifice now unlocks a much bigger win later, and greedy — which never sacrifices — misses it (greedy fails, and you need [DP](18-dynamic-programming-1.md)).

**A picture of the difference:**

- **Coin Change with coins `[1, 5, 10, 25]`, make 30:** greedily grab the biggest coin ≤ remaining → `25 + 5` = 2 coins. Correct! These coin denominations are "canonical," so greedy is provably optimal.
- **Coin Change with coins `[1, 3, 4]`, make 6:** greedy grabs `4 + 1 + 1` = 3 coins. But the real best is `3 + 3` = **2 coins**. Greedy *fails* here — grabbing the 4 blocked the better path. This is why Coin Change in [DP I](18-dynamic-programming-1.md) is DP, not greedy.

Same problem shape, opposite verdicts. **You cannot tell by looking — you must justify it.** Two ways interviewers accept:

| Justification | What it means |
|---|---|
| **Exchange argument** | Show that any optimal solution can be transformed, step by step, into the greedy one without getting worse. If you can "exchange" your way from optimal → greedy, greedy is optimal. |
| **"Greedy stays ahead"** | Prove that after each step, greedy's partial solution is at least as good as any other's. |

You don't need a formal proof in an interview, but you **do** need to *say why you believe greedy is safe here* — or admit "greedy looks tempting but can fail, so I'll use DP." That sentence is worth real points.

> **Rule of thumb:** if the problem asks for an **optimum** and choices *interact* (taking one changes what's optimal later), be suspicious of greedy → reach for [DP](18-dynamic-programming-1.md). If choices are **independent** or there's a clean sort order that makes the best choice obvious, greedy usually wins.

---

## 2. When to Use It — Pattern Triggers

| Trigger in the problem | Reach for | Note |
|---|---|---|
| "**merge / insert** overlapping **intervals / ranges**" | Intervals: sort by **start**, sweep | Merge Intervals, Insert Interval |
| "**minimum number of** rooms / arrows / platforms" | Intervals: sort by **end** (or a [heap](14-heaps-and-priority-queues.md)) | Meeting Rooms II, Non-overlapping |
| "**maximum number of non-overlapping** activities" | Greedy by **earliest end time** | Classic activity selection |
| "**can you reach** the end / the last index" | Greedy reachability | Jump Game I |
| "**minimum jumps / steps** to reach the end" | Greedy BFS-by-levels | Jump Game II |
| "assign / match to **maximise count**" | Sort both sides, two pointers | Assign Cookies |
| "**partition** a string/array into as many pieces as possible" | Greedy with a running boundary | Partition Labels |
| "one pass, some **running resource** (fuel/balance) must stay ≥ 0" | Greedy with a reset | Gas Station |

**The meta-trigger:** the phrase **"intervals" or "ranges" plus "overlap / merge / schedule"** almost always means *sort first, then sweep in one pass.* And **"minimum number of X to cover/schedule everything"** is the greedy-by-end-time signature.

---

## 3. Picture It (Diagram)

Two pictures — the two greedy moves you'll use most.

**(a) Merge Intervals — sort by *start*, then sweep and absorb overlaps.**

```
Input (unsorted):   [1,3]  [8,10]  [2,6]  [15,18]

After sort by start: [1,3] [2,6] [8,10] [15,18]

Number line:
   1   2   3   4   5   6   7   8   9  10      15   16   17   18
   ●───────●                                                        [1,3]
       ●───────────────●                                            [2,6]   overlaps [1,3] (2 ≤ 3) → merge
                           ●───────●                                [8,10]  gap after 6  → new group
                                                ●──────────────●    [15,18] gap          → new group

Result:  [1,6]  [8,10]  [15,18]
          └ 1&2 merged because 2 ≤ 3 (they touch/overlap)
```

The rule you read off the picture: **walk left to right; if the next interval starts at or before the current group's end, stretch the group's end to the max of the two; otherwise close the group and start a new one.**

**(b) Meeting Rooms II — how many rooms at the busiest moment?** Sort *starts* and *ends* separately, then sweep a timeline: `+1` room on each start, `−1` on each end. The peak concurrent count is the answer.

```
Meetings: [0,30] [5,10] [15,20]

timeline →   0    5    10    15    20        30
 start [0,30] +1
 start [5,10]      +1
 end   [5,10]            -1
 start [15,20]                +1
 end   [15,20]                     -1
 end   [0,30]                                -1
 rooms:  1    2     1     2     1         0
                    ↑ peak = 2  → need 2 rooms
```

Both pictures are the same underlying move — **sort, then sweep once** — which is why "intervals" is really a single pattern wearing different hats.

---

## 4. The Template (TypeScript)

### Skeleton A — Merge Intervals (sort by start, sweep)

```typescript
function merge(intervals: number[][]): number[][] {
  // 1. Sort by start. Number comparator, NOT default lexicographic sort!
  intervals.sort((a, b) => a[0] - b[0]);

  const result: number[][] = [];
  for (const [start, end] of intervals) {
    const last = result[result.length - 1];
    // 2. Overlap if this interval starts at/before the last one's end.
    if (last && start <= last[1]) {
      last[1] = Math.max(last[1], end); // absorb: extend the end
    } else {
      result.push([start, end]);        // no overlap: start a new group
    }
  }
  return result;
}
// merge([[1,3],[2,6],[8,10],[15,18]]) → [[1,6],[8,10],[15,18]]
```

> ⚠️ **JS/TS gotcha — always pass a numeric comparator.** `intervals.sort()` sorts **lexicographically as strings**, so `[[10,12],[2,5]]` becomes `[[10,12],[2,5]]` (because `"10" < "2"` as text). Use `.sort((a, b) => a[0] - b[0])`. This exact bug has cost people interviews — it's in the [00-README](00-README.md) trap list for a reason.

### Skeleton B — Interval scheduling (greedy by *end time*, "erase the fewest")

```typescript
// Non-overlapping Intervals: remove the MINIMUM number so the rest don't overlap.
function eraseOverlapIntervals(intervals: number[][]): number {
  if (intervals.length === 0) return 0;
  // Sort by END time — the key insight for scheduling.
  intervals.sort((a, b) => a[1] - b[1]);

  let kept = 0;
  let prevEnd = -Infinity;
  for (const [start, end] of intervals) {
    if (start >= prevEnd) {  // no overlap with the last kept interval
      kept++;
      prevEnd = end;         // greedily keep the one that ends earliest
    }
    // else: overlaps → we "remove" it (do nothing, don't advance prevEnd)
  }
  return intervals.length - kept; // removals = total − kept
}
```

**Why sort by end, not start?** The interval that **finishes earliest** leaves the most room for everything after it. Keeping it is *provably* optimal (a clean exchange argument: swapping in the earliest-finishing interval never reduces how many you can fit). This "**earliest deadline first**" instinct is the heart of interval scheduling — memorise it.

### Skeleton C — Meeting Rooms II (min rooms via sorted starts/ends)

```typescript
function minMeetingRooms(intervals: number[][]): number {
  const n = intervals.length;
  if (n === 0) return 0;
  const starts = intervals.map(i => i[0]).sort((a, b) => a - b);
  const ends   = intervals.map(i => i[1]).sort((a, b) => a - b);

  let rooms = 0, maxRooms = 0, e = 0;
  for (let s = 0; s < n; s++) {
    if (starts[s] < ends[e]) rooms++;      // a meeting starts before one ends → need a room
    else { rooms--; e++; }                 // a meeting freed up → reuse the room
    maxRooms = Math.max(maxRooms, rooms);
  }
  return maxRooms;
}
```

The **min-heap** alternative (push end times, pop when a room frees) is O(n log n) too and reads naturally — see [14 — Heaps & Priority Queues](14-heaps-and-priority-queues.md). Mention both in an interview.

---

## 5. Complexity

Almost every problem here is **dominated by the sort**: O(n log n) time, O(n) (or O(1)) extra space. The sweep itself is a single O(n) pass.

| Problem | Time | Space | Bottleneck |
|---|---|---|---|
| Merge Intervals | O(n log n) | O(n) output | the sort |
| Insert Interval (input already sorted) | **O(n)** | O(n) | just the linear scan |
| Non-overlapping Intervals | O(n log n) | O(1) | the sort |
| Meeting Rooms I (can attend all?) | O(n log n) | O(1) | the sort |
| Meeting Rooms II (min rooms) | O(n log n) | O(n) | sort / [heap](14-heaps-and-priority-queues.md) |
| Jump Game I / II | **O(n)** | O(1) | single greedy pass |
| Gas Station | **O(n)** | O(1) | single pass |
| Assign Cookies | O(n log n) | O(1) | sort both arrays |
| Partition Labels | O(n) | O(1) | last-seen map (26 letters) |

Note the pure-greedy problems (Jump Game, Gas Station) are **O(n) with no sort at all** — that's greedy at its most elegant. See [08 — Sorting](08-sorting.md) for why the comparator sorts are O(n log n).

---

## 6. Worked Example (a classic LeetCode problem)

### Merge Intervals (LeetCode 56) — fully traced

> Given intervals, merge all that overlap and return the non-overlapping result.

Using Skeleton A on `[[1,3],[2,6],[8,10],[15,18]]` (already sorted by start):

| Step | Interval | `result` before | Overlap test | Action | `result` after |
|---|---|---|---|---|---|
| 1 | `[1,3]` | `[]` | no last | push | `[[1,3]]` |
| 2 | `[2,6]` | `[[1,3]]` | `2 ≤ 3` ✅ | extend end → `max(3,6)=6` | `[[1,6]]` |
| 3 | `[8,10]` | `[[1,6]]` | `8 ≤ 6`? ❌ | push new | `[[1,6],[8,10]]` |
| 4 | `[15,18]` | `[[1,6],[8,10]]` | `15 ≤ 10`? ❌ | push new | `[[1,6],[8,10],[15,18]]` |

Result: **`[[1,6],[8,10],[15,18]]`** — matches the number-line picture in §3. The whole algorithm is "sort, then absorb-or-append." Once you've traced it once, every interval problem feels familiar.

### Non-overlapping Intervals (LeetCode 435) — greedy by end time

> Remove the minimum number of intervals so the rest don't overlap.

On `[[1,2],[2,3],[3,4],[1,3]]`, sort by **end** → `[[1,2],[2,3],[1,3],[3,4]]`:

| Interval | `start ≥ prevEnd`? | Decision | `prevEnd` | `kept` |
|---|---|---|---|---|
| `[1,2]` | `1 ≥ -∞` ✅ | keep | 2 | 1 |
| `[2,3]` | `2 ≥ 2` ✅ | keep | 3 | 2 |
| `[1,3]` | `1 ≥ 3`? ❌ | remove | 3 | 2 |
| `[3,4]` | `3 ≥ 3` ✅ | keep | 4 | 3 |

Kept 3 of 4 → **remove 1.** Sorting by *end* is what makes this optimal: by always keeping the earliest-finishing compatible interval, we leave maximum room for the rest. Try sorting by *start* instead and you'll get a wrong answer on `[[1,100],[2,3],[3,4]]` — proof that the sort key is load-bearing, not a detail.

### Meeting Rooms II (LeetCode 253) — the concurrency count

Covered by Skeleton C and the §3 timeline diagram: the answer is the **maximum number of meetings happening at the same instant**. Sweep sorted starts against sorted ends; every start before the next end needs a fresh room, every end frees one, and the peak is your answer. The [heap](14-heaps-and-priority-queues.md) version stores active end times and pops the earliest when a room frees — same O(n log n), often cleaner to explain.

---

## 7. Practice Set (curated LeetCode)

Do the intervals block first (they share one skeleton), then the pure-greedy block.

| # | Problem | LeetCode | Difficulty | Pattern |
|---|---|---|---|---|
| 1 | Merge Intervals | 56 | Medium | Sort by start, sweep |
| 2 | Insert Interval | 57 | Medium | Three-phase scan (before / overlap / after) |
| 3 | Non-overlapping Intervals | 435 | Medium | Greedy by **end** time |
| 4 | Meeting Rooms I | 252 | Easy | Sort, check adjacent overlap |
| 5 | Meeting Rooms II | 253 | Medium | Sweep starts/ends, or [min-heap](14-heaps-and-priority-queues.md) |
| 6 | Jump Game | 55 | Medium | Greedy reachability |
| 7 | Jump Game II | 45 | Medium | Greedy level-by-level (min jumps) |
| 8 | Gas Station | 134 | Medium | Greedy with reset |
| 9 | Partition Labels | 763 | Medium | Greedy running boundary |
| 10 | Assign Cookies | 455 | Easy | Sort both, two pointers |

**Jump Game (55)** — a clean O(n) greedy, no sort:

```typescript
function canJump(nums: number[]): boolean {
  let maxReach = 0;
  for (let i = 0; i < nums.length; i++) {
    if (i > maxReach) return false;              // stuck: can't even step onto i
    maxReach = Math.max(maxReach, i + nums[i]);  // greedily extend our reach
  }
  return true;
}
```

**Gas Station (134)** — greedy with a restart, and a slick correctness argument:

```typescript
function canCompleteCircuit(gas: number[], cost: number[]): number {
  let total = 0, tank = 0, start = 0;
  for (let i = 0; i < gas.length; i++) {
    const diff = gas[i] - cost[i];
    total += diff;
    tank += diff;
    if (tank < 0) {           // can't reach i+1 from `start`
      start = i + 1;          // so no station in [start..i] can be the answer → jump past
      tank = 0;
    }
  }
  return total >= 0 ? start : -1; // a valid start exists iff total gas ≥ total cost
}
```

> The greedy leap in Gas Station — "if you run dry between `start` and `i`, no station in that range can be the start" — is the kind of exchange-style insight interviewers want you to *state*, not just code. Practise verbalising it.

---

## 8. Common Pitfalls & Interview Tips

| Pitfall | What goes wrong | Fix |
|---|---|---|
| **Default `.sort()`** | Lexicographic string sort mangles numbers (`[10] < [2]`). | Always `.sort((a, b) => a[0] - b[0])` (or `a[1] - b[1]`). |
| **Sorting by the wrong key** | Merge needs sort by **start**; scheduling ("keep the most") needs sort by **end**. Swap them and you get subtle wrong answers. | Merge/insert → **start**. "Max non-overlapping / min removals / min rooms" → **end**. |
| **Greedy that's actually wrong** | Committing locally when choices interact (e.g. Coin Change `[1,3,4]`). | If a choice now changes what's optimal later, suspect it → use [DP](18-dynamic-programming-1.md). Justify greedy before trusting it. |
| **Boundary equality (`<` vs `<=`)** | Do intervals that merely *touch* (`[1,3]` and `[3,5]`) count as overlapping? Depends on the problem. | Read whether endpoints are inclusive. For Meeting Rooms, `[a,b]` and `[b,c]` usually **don't** conflict (`start < end`), so use `<`, not `<=`. |
| **Mutating input intervals** | `last[1] = ...` edits the caller's array. | Push fresh copies (`result.push([start, end])`) if the caller reuses the input. |
| **Empty input** | `intervals[intervals.length-1]` on `[]` throws / misbehaves. | Guard `length === 0` up front. |
| **Off-by-one in the sweep** | Meeting Rooms II mis-advancing the `ends` pointer. | Only advance `e` when you free a room; recompute `maxRooms` each step. |

**Interview scripts that score points:**
- **State the sort key and *why*:** "I'll sort by end time because keeping the earliest-finishing meeting leaves the most room for the rest" — this shows you understand the greedy, not just memorised it.
- **Pre-empt the greedy-vs-DP question.** Say: "Greedy works here because [exchange reason]. If choices interacted — like Coin Change with arbitrary denominations — I'd switch to DP." Naming the boundary between the two ([18](18-dynamic-programming-1.md)/[19](19-dynamic-programming-2.md) vs greedy) is senior-level signal.
- **Clarify endpoint semantics early:** "Do touching intervals `[1,3]` and `[3,5]` overlap?" One question saves a wrong submission.
- **Offer the heap alternative** for Meeting Rooms II to show range → [14 — Heaps](14-heaps-and-priority-queues.md).

> **Real-project analogy (Habex).** A calendar/habit-scheduling view on [habex.santhu.site](https://habex.santhu.site) that overlays your daily time blocks is *literally* Meeting Rooms II: "what's the maximum number of overlapping commitments, and can they all fit?" Sort the blocks, sweep the day, and the peak concurrency tells you if you've double-booked yourself.

---

## 9. TL;DR

- **Greedy = grab the locally-best choice, commit, never look back.** Cheap and fast *when it's valid* — usually a sort plus one O(n) pass.
- **The whole skill is knowing when greedy is safe.** Justify it with an **exchange argument** or "greedy stays ahead." If choices *interact* (a pick now changes later optimums, like Coin Change `[1,3,4]`), greedy fails → use [DP](18-dynamic-programming-1.md). Say which and why in the interview.
- **Intervals = one pattern: sort, then sweep once.** Sort by **start** to **merge/insert**; sort by **end** to **schedule** ("keep the most", "remove the fewest", "minimum rooms").
- **Merge Intervals:** if `start ≤ lastEnd`, extend the end; else append a new group.
- **Interval scheduling:** earliest-finishing-first is provably optimal — that's why the sort key is by **end**, not start.
- **Meeting Rooms II:** answer = peak concurrent meetings; sweep sorted starts vs ends, or use a [min-heap](14-heaps-and-priority-queues.md) of end times.
- **Pure greedy gems (O(n), no sort):** Jump Game (track max reach), Gas Station (reset start when the tank goes negative), Partition Labels (extend a running boundary to each char's last occurrence).
- **JS gotchas:** never trust default `.sort()` on numbers/tuples — pass a comparator; watch `<` vs `<=` at touching endpoints; guard empty input; don't mutate the caller's intervals.
- **Complexity** is almost always **O(n log n)** (dominated by the sort) — see [08 — Sorting](08-sorting.md).

**Where you are now:** you've completed the core optimisation trio — [18 — DP I](18-dynamic-programming-1.md), [19 — DP II](19-dynamic-programming-2.md), and greedy/intervals here. You can now recognise, for any "make the best sequence of choices" problem, whether to *remember everything* (DP) or *commit and move on* (greedy). That judgment is exactly what separates a strong interview from a stuck one.
