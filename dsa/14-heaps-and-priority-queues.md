# 14. Heaps & Priority Queues

> **One-liner:** A heap is a "**always know the min-or-max in O(1), remove it in O(log n)**" structure, secretly just an array with clever index math — and a **priority queue is literally a heap**.
> **Pattern power:** The moment a problem says **"top K", "K largest/smallest/closest/frequent", "median of a stream", "merge K sorted", or "process by priority"**, a heap turns an `O(n log n)` sort (or worse) into `O(n log k)` — and it's the *only* structure that lets you keep a running answer over data you haven't fully seen yet.

> ⚠️ **The #1 JS/TS interview trap in this whole track:** **JavaScript/TypeScript has NO built-in heap or priority queue.** Python has `heapq`, Java has `PriorityQueue`, C++ has `priority_queue`. You have… nothing. So you **must** be able to write a ~30-line heap from memory. Interviewers at Zepto/Swiggy/CRED/Razorpay/Groww *know* JS lacks it — casually saying *"JS has no PQ, so here's my min-heap"* and typing it out is a genuine green flag. This chapter's job is to burn that class into your memory.

---

## 1. The Idea in Plain Words

Imagine a hospital ER waiting room. Patients don't get seen in arrival order (that's a normal queue) — the **most critical** patient is seen next, no matter when they arrived. When a new patient walks in, they slot into the right spot; when a doctor frees up, they instantly grab the most urgent case. That "instantly grab the most urgent" machine is a **priority queue**, and the data structure that powers it is a **heap**.

A **binary heap** is a binary tree with two rules:

1. **Shape:** it's a *complete* tree — every level is full except possibly the last, which fills left-to-right. (This is what lets us store it in a flat array with no gaps.)
2. **Heap order:**
   - In a **min-heap**, every parent is **≤** its children → the **smallest** element is always at the **root**.
   - In a **max-heap**, every parent is **≥** its children → the **largest** element is at the root.

Note it is **not** fully sorted — only the path from root down is ordered. That partial order is exactly why a heap is cheaper than a sorted array: you pay only to keep the *extreme* correct, not everything.

**The magic trick — no pointers, just an array.** Because the tree is always complete, you can flatten it into an array and compute family relationships with arithmetic:

```
For the node at index i:
  left child  = 2*i + 1
  right child = 2*i + 2
  parent      = (i - 1) >> 1     // >> 1 is integer divide-by-2, i.e. floor((i-1)/2)
```

That's the whole secret. A heap is an array plus two helper motions:

- **`siftUp` (a.k.a. bubble-up):** after adding at the end, swap the newcomer *upward* with its parent while it's out of order.
- **`siftDown` (a.k.a. bubble-down / heapify-down):** after removing the root, drop the last element into the root slot and swap it *downward* with its smaller (min-heap) child until order is restored.

Both motions walk the *height* of the tree, which is `log n` — that's where `O(log n)` push/pop comes from. Peeking the min/max is just reading index `0`: `O(1)`.

> **Heap vs. sorted structures:** A balanced BST ([13-binary-search-trees.md](13-binary-search-trees.md)) also gives you the min/max, and so does sorting ([08-sorting.md](08-sorting.md)) — but if all you need is *"repeatedly give me the extreme,"* a heap is simpler and has smaller constant factors. And unlike sorting, a heap works on a **stream** you can't see all of at once. Also note: **JS has no built-in `TreeMap`/sorted-set either**, so the heap is often your only ordered tool.

---

## 2. When to Use It — Pattern Triggers

Scan the problem for these signals:

| If the problem says… | Reach for | Why a heap wins |
|---|---|---|
| "**top K** / K **largest** / K **smallest**" | size-K heap | `O(n log k)` beats sorting's `O(n log n)` |
| "K **closest** points / K **most frequent**" | size-K heap (+ freq map) | keep only K candidates alive at once |
| "**median** of a data **stream**" | **two heaps** | balance a max-heap + min-heap around the middle |
| "**merge K sorted** lists/arrays" | min-heap of K heads | always pull the global smallest next |
| "**Kth** largest/smallest" | size-K heap (or QuickSelect) | root of a size-K heap = the answer |
| "**schedule / process by priority**", "next task due" | priority queue | the heap *is* the scheduler |
| "shortest path, weighted graph" (Dijkstra, Prim) | min-heap of `(dist, node)` | greedily expand the closest frontier — see [17](17-graphs-advanced.md) |
| "**running / online**" + "at any time report the extreme" | heap | streaming; you can't pre-sort the future |

**The meta-trigger:** you need the **best-so-far** *repeatedly*, and the set keeps **changing** (things arrive or get removed). Sorting gives a one-shot order; a heap gives a *living* one.

**Anti-triggers (don't force a heap):**
- You need the data **fully sorted** at the end → just sort ([08](08-sorting.md)).
- You need **exact-value lookup / dedupe / counting** → hash map/set ([03](03-hashing-maps-and-sets.md)).
- You need **arbitrary-position** access or range queries → not a heap's job.

---

## 3. Picture It (Diagram)

A min-heap as a tree, and the exact array behind it:

```
Min-heap (root is always the smallest)

              (1)                index 0
             /   \
          (3)     (2)            index 1, 2
          / \     /
       (7) (4)  (5)              index 3, 4, 5

Backing array:   [ 1 , 3 , 2 , 7 , 4 , 5 ]
index:             0   1   2   3   4   5

Index math check — node (3) sits at i = 1:
  left  child = 2*1+1 = 3  -> (7)   ✓
  right child = 2*1+2 = 4  -> (4)   ✓
  parent      = (1-1)>>1 = 0 -> (1) ✓
Heap order holds: every parent ≤ its children.
```

**`push(0)` → siftUp** (the new value bubbles up until its parent is smaller):

```
append 0 at the end (index 6):   [1,3,2,7,4,5, 0]
                                              ↑ compare with parent (2) at index 2
0 < 2  → swap:                   [1,3,0,7,4,5, 2]
                                    ↑ compare with parent (1) at index 0
0 < 1  → swap:                   [0,3,1,7,4,5, 2]
                                  ↑ parent-less (root). Stop. New min = 0.
```

**`pop()` → siftDown** (remove root, move last to top, sink it back down):

```
Start:  [1,3,2,7,4,5]      root 1 is the min we return
move last (5) to root:  [5,3,2,7,4]
                         ↑ compare with children 3 (idx1) and 2 (idx2); smallest child is 2
5 > 2  → swap with 2:   [2,3,5,7,4]
                             ↑ 5 now has no children in-bounds. Stop.
Returned 1; new root 2 is the next-smallest.  ✓
```

Notice both motions only ever touch **one root-to-leaf path** = height = `log n` swaps. That is the entire cost model.

---

## 4. The Template (TypeScript)

Memorize **one** generic engine; everything else is a comparator. This is the class to be able to type in your sleep.

```ts
// ── The one class to memorize ──────────────────────────────────────────────
// A binary heap. `compare(a, b) < 0` means "a comes out BEFORE b".
class Heap<T> {
  private heap: T[] = [];

  // Optional `items` are heapified in O(n) (see §5 — cheaper than n pushes).
  constructor(private compare: (a: T, b: T) => number, items: T[] = []) {
    this.heap = items.slice();
    for (let i = (this.heap.length >> 1) - 1; i >= 0; i--) this.siftDown(i);
  }

  size(): number { return this.heap.length; }
  isEmpty(): boolean { return this.heap.length === 0; }
  peek(): T | undefined { return this.heap[0]; }          // O(1)

  push(value: T): void {                                   // O(log n)
    this.heap.push(value);
    this.siftUp(this.heap.length - 1);
  }

  pop(): T | undefined {                                   // O(log n)
    const n = this.heap.length;
    if (n === 0) return undefined;
    const top = this.heap[0];
    const last = this.heap.pop()!;   // remove the physical last slot
    if (n > 1) {                     // if that wasn't the only element,
      this.heap[0] = last;           // move it to the root and sink it
      this.siftDown(0);
    }
    return top;
  }

  private siftUp(i: number): void {
    while (i > 0) {
      const parent = (i - 1) >> 1;
      if (this.compare(this.heap[i], this.heap[parent]) >= 0) break; // in order
      [this.heap[i], this.heap[parent]] = [this.heap[parent], this.heap[i]];
      i = parent;
    }
  }

  private siftDown(i: number): void {
    const n = this.heap.length;
    while (true) {
      let best = i;
      const l = 2 * i + 1, r = 2 * i + 2;
      if (l < n && this.compare(this.heap[l], this.heap[best]) < 0) best = l;
      if (r < n && this.compare(this.heap[r], this.heap[best]) < 0) best = r;
      if (best === i) break;         // parent already smallest of the trio
      [this.heap[i], this.heap[best]] = [this.heap[best], this.heap[i]];
      i = best;
    }
  }
}
```

Now the **MinHeap / MaxHeap** for numbers are one-liners — proof that *the comparator is the only difference*:

```ts
// ── Min & Max heaps for numbers: just pick the comparator ──────────────────
class MinHeap extends Heap<number> {
  constructor(items: number[] = []) { super((a, b) => a - b, items); } // asc  → min at top
}
class MaxHeap extends Heap<number> {
  constructor(items: number[] = []) { super((a, b) => b - a, items); } // desc → max at top
}

const mn = new MinHeap();
mn.push(5); mn.push(1); mn.push(3);
mn.peek(); // 1
mn.pop();  // 1  (then 3, then 5)

// Objects? Pass a field comparator to the generic Heap directly:
type Task = { name: string; priority: number };
const pq = new Heap<Task>((a, b) => a.priority - b.priority); // min-priority first
pq.push({ name: "email", priority: 5 });
pq.push({ name: "deploy", priority: 1 });
pq.peek(); // { name: "deploy", priority: 1 }
```

> **The comparator mirrors the `sort()` gotcha** from [08-sorting.md](08-sorting.md): `(a, b) => a - b` is ascending (min-heap), `(a, b) => b - a` is descending (max-heap). Same mental model, so it's one rule to remember, not two.
>
> **The "negation" hack (know it, but prefer the comparator):** if you only have a number `MinHeap` handy, you can fake a max-heap by pushing `-x` and negating on pop. It's a nice one-liner but breaks for objects — the comparator is the general answer.

**A priority queue *is* this class.** In your own products the analogy is exact: a reminders/scheduler for **[Habex](https://habex.santhu.site)** could keep a `Heap<Habit>((a,b) => a.dueAt - b.dueAt)` so the *soonest-due* habit is always `peek()`-able in O(1), and firing + rescheduling it is O(log n) — no re-sorting the whole list on every tick.

---

## 5. Complexity

| Operation | Time | Note |
|---|---|---|
| `peek` (min/max) | **O(1)** | just read index 0 |
| `push` | **O(log n)** | one siftUp along the height |
| `pop` | **O(log n)** | one siftDown along the height |
| Build from `n` items via **n pushes** | **O(n log n)** | fine, but not optimal |
| Build via **heapify** (constructor loop) | **O(n)** | ⭐ cheaper — siftDown from the middle back to root |
| Space | **O(n)** | just the backing array |

**The killer pattern — size-K heap → `O(n log k)`:** to find the top/closest/most-frequent **K** out of **n**, you never hold more than K items. Each of n elements costs `log k` (not `log n`) to push/pop, so total is **`O(n log k)` time, `O(k)` space**. When `k ≪ n`, that crushes a full `O(n log n)` sort — and it works even if the n items are *streaming in* and can't all fit in memory.

Why heapify is O(n) (not O(n log n)): most nodes are near the bottom and sink at most a tiny distance; the math sums to a linear total. If you're handed an array up front, prefer `new MinHeap(arr)` over pushing one-by-one.

---

## 6. Worked Example (a classic LeetCode problem)

### Kth Largest Element in an Array — [LeetCode 215](https://leetcode.com/problems/kth-largest-element-in-an-array/)

> Given `nums` and `k`, return the **k-th largest** element (in sorted order, not the k-th distinct).

**The counterintuitive key:** to track the K **largest**, use a **MIN**-heap of size K. Its root is the *smallest of the K biggest-so-far* — i.e. the "weakest survivor." When a bigger number arrives, it kicks out that weakest survivor. After you've seen everything, the K items left are the K largest, and the **root is exactly the Kth largest.**

```ts
function findKthLargest(nums: number[], k: number): number {
  const heap = new MinHeap();      // holds the k largest seen so far
  for (const num of nums) {
    heap.push(num);
    if (heap.size() > k) heap.pop(); // evict the smallest — it can't be top-k
  }
  return heap.peek()!;             // root = smallest of the k largest = Kth largest
}
```

**Full trace** on `nums = [3, 2, 1, 5, 6, 4]`, `k = 2` (answer should be `5`, since sorted-desc is `6,5,4,3,2,1`):

| Incoming | After push & (pop if size>2) | Heap (min-heap, root first) | Root = current Kth-largest guess |
|---|---|---|---|
| `3` | push | `[3]` | — (not enough yet) |
| `2` | push | `[2, 3]` | 2 |
| `1` | push then pop `1` | `[2, 3]` | 2 |
| `5` | push then pop `2` | `[3, 5]` | 3 |
| `6` | push then pop `3` | `[5, 6]` | 5 |
| `4` | push then pop `4` | `[5, 6]` | **5** |

Final `heap.peek()` = **5**. ✓ We only ever held 2 numbers → `O(n log k)` time, `O(k)` space.

> **Interview bonus — QuickSelect:** the average `O(n)` alternative is QuickSelect (partition around a pivot, recurse into one side only). Mention it to show range — but the heap solution is cleaner to write correctly under pressure, and it's the one that generalizes to *streaming* Kth-largest ([LeetCode 703]).

### Also know: Top K Frequent Elements — [LeetCode 347](https://leetcode.com/problems/top-k-frequent-elements/)

Same size-K heap, now combined with a **frequency map** ([03-hashing-maps-and-sets.md](03-hashing-maps-and-sets.md)) — count first, then keep the K highest counts:

```ts
function topKFrequent(nums: number[], k: number): number[] {
  const freq = new Map<number, number>();
  for (const n of nums) freq.set(n, (freq.get(n) ?? 0) + 1);   // step 1: count (ch 03)

  // step 2: min-heap keyed by frequency, capped at size k
  const heap = new Heap<[number, number]>((a, b) => a[1] - b[1]); // compare by count
  for (const [num, count] of freq) {
    heap.push([num, count]);
    if (heap.size() > k) heap.pop();          // drop the least frequent
  }

  const out: number[] = [];
  while (!heap.isEmpty()) out.push(heap.pop()![0]);
  return out;                                  // the k most frequent (any order)
}
```

`O(n)` to count + `O(m log k)` to keep the top K of `m` distinct values.

### The two-heaps pattern: Median from a Data Stream — [LeetCode 295](https://leetcode.com/problems/find-median-from-data-stream/)

Keep the smaller half in a **max-heap** (`low`) and the larger half in a **min-heap** (`high`). The two tops straddle the median. Keep sizes balanced (`low` holds the extra when the count is odd):

```ts
class MedianFinder {
  private low = new MaxHeap();   // smaller half; its MAX is just below the median
  private high = new MinHeap();  // larger half;  its MIN is just above the median

  addNum(num: number): void {
    this.low.push(num);
    this.high.push(this.low.pop()!);                 // funnel through to keep order right
    if (this.high.size() > this.low.size()) {        // rebalance so low >= high in size
      this.low.push(this.high.pop()!);
    }
  }

  findMedian(): number {
    if (this.low.size() > this.high.size()) return this.low.peek()!;       // odd count
    return (this.low.peek()! + this.high.peek()!) / 2;                     // even count
  }
}
```

Every `addNum` is `O(log n)`; every `findMedian` is `O(1)`. No sorting a growing stream — impossible to beat with a plain array.

### The K-way merge: Merge K Sorted Lists — [LeetCode 23](https://leetcode.com/problems/merge-k-sorted-lists/)

Put the **head of each list** in a min-heap; repeatedly pop the global smallest and push its `.next`:

```ts
class ListNode {
  constructor(public val: number = 0, public next: ListNode | null = null) {}
}

function mergeKLists(lists: Array<ListNode | null>): ListNode | null {
  const heap = new Heap<ListNode>((a, b) => a.val - b.val);
  for (const node of lists) if (node) heap.push(node);   // K heads

  const dummy = new ListNode();
  let tail = dummy;
  while (!heap.isEmpty()) {
    const node = heap.pop()!;         // smallest current head across all lists
    tail.next = node;
    tail = node;
    if (node.next) heap.push(node.next);
  }
  return dummy.next;
}
```

`N` total nodes, at most `k` in the heap → **`O(N log k)`**. The same idea drives **Dijkstra's shortest path** ([17-graphs-advanced.md](17-graphs-advanced.md)): a min-heap of `(distance, node)` always expands the closest unfinished node next.

---

## 7. Practice Set (curated LeetCode)

Do them in this order — each adds one twist.

| # | Problem | LC | Difficulty | Heap flavor |
|---|---|---|---|---|
| 1 | Kth Largest Element in an Array | 215 | Medium | size-K **min**-heap (the template) |
| 2 | Last Stone Weight | 1046 | Easy | **max**-heap, repeatedly pop 2 |
| 3 | K Closest Points to Origin | 973 | Medium | size-K **max**-heap by distance² |
| 4 | Top K Frequent Elements | 347 | Medium | freq map ([03](03-hashing-maps-and-sets.md)) + size-K heap |
| 5 | Sort Characters By Frequency | 451 | Medium | count then heap/bucket |
| 6 | Kth Largest Element in a **Stream** | 703 | Easy | size-K min-heap kept live |
| 7 | Task Scheduler | 621 | Medium | max-heap of counts (greedy) — see [20](20-greedy-and-intervals.md) |
| 8 | Merge K Sorted Lists | 23 | Hard | K-way merge, min-heap of heads |
| 9 | Find Median from Data Stream | 295 | Hard | **two heaps** (max + min) |
| 10 | Reorganize String | 767 | Medium | max-heap by remaining count |

> **For "K closest" / "K smallest"** flip the intuition from the worked example: keep a size-K **MAX**-heap and pop the *largest* when it overflows — so the K *smallest/closest* survive.

---

## 8. Common Pitfalls & Interview Tips

| Pitfall | Fix / What to say |
|---|---|
| **"I'll just use the built-in heap."** There isn't one in JS/TS. | Say it out loud and *write the class*. This is the signal interviewers are fishing for. |
| **Min vs. max backwards.** Using a max-heap for "K largest." | For **K largest → size-K MIN-heap** (evict the smallest). For **K smallest/closest → size-K MAX-heap**. Reason about the root = "weakest survivor." |
| **Comparator for objects.** `(a, b) => a - b` on objects gives `NaN`. | Compare a field: `(a, b) => a.priority - b.priority`. Same as the `sort()` trap in [08](08-sorting.md). |
| **Pushing n items one-by-one when you have them all.** `O(n log n)`. | Pass the array to the constructor → **heapify in `O(n)`**. |
| **Forgetting to cap the size-K heap.** Holding all n. | Add `if (heap.size() > k) heap.pop()` — that's what makes it `O(n log k)` and `O(k)` space. |
| **`peek()`/`pop()` on empty.** Returns `undefined`, then `undefined.val` crashes. | Guard with `isEmpty()`; use `!` only when you've checked. |
| **Off-by-one in index math.** parent = `i/2`. | It's **`(i - 1) >> 1`**. Children are `2i+1`, `2i+2`. Write it once, trust it. |
| **Comparing tuples by the wrong slot.** `[num, count]` sorted by `num`. | Be explicit: `(a, b) => a[1] - b[1]` to key by count. |

**When a heap beats sorting (say this to justify your choice):**
- **Streaming / online data** — you must answer *before* seeing everything; you can't sort the future.
- **Partial order is enough** — you need only the top K, so `O(n log k)` < `O(n log n)`.
- **Repeated extract-min/max with insertions interleaved** — merges, Dijkstra, schedulers.

**Talking track:** *"JS has no built-in priority queue, so I'll drop in a min-heap. For the K largest I keep a size-K min-heap — each element is O(log k), total O(n log k), O(k) space — which beats sorting when k is small and works even if the input is a stream."* That single sentence hits the gotcha, the pattern, and the complexity.

---

## 9. TL;DR

- A **heap** is a complete binary tree stored in a **plain array**; **min-heap** keeps the smallest at the root, **max-heap** the largest.
- Index math is the whole trick: **left `2i+1`, right `2i+2`, parent `(i-1)>>1`.** `siftUp` on push, `siftDown` on pop, each `O(log n)`; **`peek` is `O(1)`.**
- **JS/TS has no built-in heap or priority queue** — memorize the ~30-line `Heap<T>` class. A **priority queue *is* a heap**; min vs max is *just the comparator* (`a-b` vs `b-a`), same as the `sort()` rule ([08](08-sorting.md)).
- **Top-K pattern:** a **size-K heap** solves "K largest/smallest/closest/frequent" in **`O(n log k)`**. For K *largest* use a size-K **min**-heap; for K *smallest/closest* use a size-K **max**-heap.
- **Two-heaps** (max-heap + min-heap) tracks a **streaming median** in `O(log n)` add / `O(1)` query.
- Heaps power **K-way merge** (merge K sorted lists, `O(N log k)`) and **Dijkstra** ([17](17-graphs-advanced.md)).
- **Trigger words:** "top K", "K largest/smallest/closest/frequent", "median of a stream", "merge K sorted", "by priority", "shortest path (weighted)".
- Pair it with a **frequency map** ([03](03-hashing-maps-and-sets.md)) for "top K frequent." Choose a heap over a **sort** ([08](08-sorting.md)) when data **streams** or you need only a **partial** order.

**Next:** [15 — Tries (Prefix Trees)](15-tries.md) — the other "specialized tree" that turns prefix/autocomplete questions into O(length) lookups.
