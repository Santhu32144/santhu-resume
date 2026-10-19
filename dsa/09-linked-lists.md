# 09. Linked Lists

> **One-liner:** A chain of little boxes where each box holds a value and a pointer to the *next* box — cheap to splice in the middle, expensive to jump to the 500th item.
> **Pattern power:** Master three moves — the **dummy head**, **iterative reversal (pointer rewiring)**, and the **fast & slow pointers** trick — and you can solve the vast majority of linked-list interview questions.

---

## 1. The Idea in Plain Words

A **linked list** is a sequence of **nodes**. Each node holds two things:

1. a **value** (the data — a number, a string, whatever), and
2. a **`next`** pointer — a reference to the following node (or `null` if it's the last one).

You only ever hold onto the **head** (the first node). To reach any other node, you follow `next` pointers one hop at a time, like following a treasure hunt where each clue points to the next clue.

```
head
 │
 ▼
[ 3 | ● ]──▶[ 7 | ● ]──▶[ 1 | ● ]──▶ null
  val next    val next    val next
```

### "But I already have arrays — why bother?"

Great instinct. Here's the honest comparison, and it comes down to **memory layout**.

An **array** is one solid, contiguous block of memory. Because every slot is the same size and they sit side by side, the computer can compute the address of index `i` instantly (`base + i * size`). That's why `arr[500]` is **O(1)** — random access is free.

A **linked list** is *scattered*. Node 0 might live at address `0x100`, node 1 at `0x2F8`, node 2 at `0x0A0` — wherever there was free space. The only thing tying them together is the `next` pointer inside each node. So to reach the 500th node you must walk 500 hops: **O(n)** random access. There's no formula to jump.

```
Array — contiguous block, index math works:
 idx:   0     1     2     3
      [ 3 ] [ 7 ] [ 1 ] [ 9 ]      ← one solid slab in RAM

Linked list — scattered, only pointers connect them:
   0x100         0x2F8         0x0A0
 [3|0x2F8] ───▶ [7|0x0A0] ───▶ [1|null]
```

So what do you *gain*? **Cheap insertion and deletion once you're standing at the right spot.** To insert a node between two nodes you already hold, you rewire two pointers — **O(1)**. No shifting. In an array, inserting at the front means sliding every other element over by one — **O(n)**.

| Operation | Array | Linked List |
|---|---|---|
| Access by index (`get(i)`) | **O(1)** ✅ | O(n) |
| Insert / delete at the **front** | O(n) (`unshift`/`shift`) | **O(1)** ✅ |
| Insert / delete at a node you **already hold** | O(n) (shift the rest) | **O(1)** ✅ (rewire) |
| Search for a value | O(n) | O(n) |
| Memory overhead | low (just data) | higher (a pointer per node) |

> **This is the mental model:** arrays win at *reading by position*; linked lists win at *stitching things in and out* when you're already there. Interviews test the second skill because pointer rewiring is where bugs hide.

### Singly vs doubly linked

- **Singly linked:** each node points only **forward** (`next`). Lightweight. You can't walk backward, and to delete a node you usually need its *predecessor*.
- **Doubly linked:** each node has **`next`** *and* **`prev`**. Twice the pointers, but you can walk both directions and delete a node you're standing on in true O(1). This is the backbone of an **LRU cache** (a classic interview problem — hash map + doubly linked list).

```
Doubly linked:
 null ◀──[ prev | 3 | next ]⇄[ prev | 7 | next ]⇄[ prev | 1 | next ]──▶ null
```

> **React aside (why this matters to you):** React's Fiber tree isn't an array — each fiber links to its `child`, `sibling`, and `return` (parent) via pointers, a linked structure. That pointer-based design is *exactly* what lets React pause, resume, and bail out of rendering work. You've been using linked lists this whole time.

A **JavaScript array is NOT a linked list.** `number[]` is a dynamic array (contiguous-ish, index-addressable). There is no built-in linked list in JS/TS — you build it from a tiny class, which is the next section.

---

## 2. When to Use It — Pattern Triggers

| If the problem… | Reach for | Why |
|---|---|---|
| Literally hands you a **linked list** (`ListNode`) | The techniques here | It's testing pointer manipulation on purpose |
| Says **"detect a cycle"** / "does it loop?" | **Fast & slow pointers** (Floyd's) | Two speeds must collide inside a loop |
| Says **"find the middle"** | **Fast & slow pointers** | Fast reaches the end when slow is at the middle |
| Says **"reverse"** / "reorder" the list | **Iterative pointer rewiring** | The core O(1)-space move |
| Says **"merge two sorted lists"** | **Dummy head** + two pointers | Splice nodes onto a fresh tail |
| Says **"remove the Nth node from the end"** | **Dummy head** + fast/slow with a gap | One pass, no length count |
| Says **"is it a palindrome?"** | Find middle → reverse half → compare | Combines two techniques |

The two power tools — **dummy head** and **fast & slow** — show up constantly, so learn them cold. Fast & slow is itself a flavor of the two-pointer family, so if that idea is new, skim **[04-two-pointers.md](04-two-pointers.md)** first.

---

## 3. Picture It (Diagram)

**Traversal** — start at `head`, follow `next` until `null`:

```
curr
 │
 ▼
[3|●]──▶[7|●]──▶[1|●]──▶ null
         ▲
    curr = curr.next   (one hop at a time)
```

**Insertion at a known node** — splice `[9]` between `[3]` and `[7]` by rewiring TWO pointers (no shifting anything else):

```
Before:            [3|●]────────▶[7|●]──▶...
                                      
Step 1: 9.next = 3.next    [3|●]   [7|●]
                              │  ┌───▲
                              │  │
                            [9|●]┘

Step 2: 3.next = 9          [3|●]──▶[9|●]──▶[7|●]──▶...
```

Only two pointers changed. That's the **O(1) insert** superpower — impossible to match with an array, which would slide every later element right.

**Fast & slow pointers** — `slow` hops 1, `fast` hops 2. On a straight list `fast` falls off the end first; if there's a loop, `fast` laps `slow` and they *collide*:

```
slow ▶ one step
fast ▶ two steps

[1]──▶[2]──▶[3]──▶[4]──▶[5]
 S,F                              (start together)
        S           F             (after 1 tick)
              S                 F (after 2 ticks → fast at/past end → no cycle)
```

---

## 4. The Template (TypeScript)

There is no linked list in the standard library, so **you define the node**. This exact class is what LeetCode hands you in TS problems:

```typescript
class ListNode {
  val: number;
  next: ListNode | null;

  constructor(val: number = 0, next: ListNode | null = null) {
    this.val = val;
    this.next = next;
  }
}
```

**Build & traverse:**

```typescript
// Build 3 → 7 → 1 → null
const head = new ListNode(3, new ListNode(7, new ListNode(1)));

// Walk it. `curr` is narrowed to non-null inside the loop by the check.
let curr: ListNode | null = head;
while (curr !== null) {
  console.log(curr.val); // 3, 7, 1
  curr = curr.next;
}
```

**Template A — the Dummy (Sentinel) Head.** A throwaway node placed *before* the real head so that "insert" and "delete" never have to special-case the front of the list. You build onto `dummy.next` and return it at the end. This single trick removes a huge class of bugs.

```typescript
function buildWithDummy(head: ListNode | null): ListNode | null {
  const dummy = new ListNode(0);   // sentinel: never holds real data
  let tail = dummy;                // we always append after `tail`

  let curr = head;
  while (curr !== null) {
    tail.next = new ListNode(curr.val); // attach a node
    tail = tail.next;                   // advance the tail
    curr = curr.next;
  }
  return dummy.next; // the REAL head — skip the sentinel
}
```

**Template B — Reverse a Linked List (iterative pointer rewiring).** The single most important linked-list move. Memorize the four lines and their *order*:

```typescript
function reverseList(head: ListNode | null): ListNode | null {
  let prev: ListNode | null = null;
  let curr: ListNode | null = head;

  while (curr !== null) {
    const next: ListNode | null = curr.next; // 1. SAVE next before we lose it
    curr.next = prev;                         // 2. rewire: point backward
    prev = curr;                              // 3. advance prev
    curr = next;                              // 4. advance curr
  }
  return prev; // prev is the new head
}
```

**Template C — Fast & Slow Pointers.** Three of the most common uses:

```typescript
// Find the MIDDLE node (for even length, returns the 2nd of the two middles).
function middleNode(head: ListNode | null): ListNode | null {
  let slow = head;
  let fast = head;
  while (fast !== null && fast.next !== null) {
    slow = slow!.next;      // `!` = "I promise this isn't null" (see pitfalls)
    fast = fast.next.next;
  }
  return slow;
}

// DETECT a cycle (Floyd's Tortoise and Hare).
function hasCycle(head: ListNode | null): boolean {
  let slow: ListNode | null = head;
  let fast: ListNode | null = head;
  while (fast !== null && fast.next !== null) {
    slow = slow!.next;      // slow trails fast, so it can't be null here
    fast = fast.next.next;
    if (slow === fast) return true; // they collided → there's a loop
  }
  return false; // fast fell off the end → no loop
}

// Find where the cycle STARTS (returns the entry node, or null).
function detectCycle(head: ListNode | null): ListNode | null {
  let slow: ListNode | null = head;
  let fast: ListNode | null = head;
  while (fast !== null && fast.next !== null) {
    slow = slow!.next;
    fast = fast.next.next;
    if (slow === fast) {
      // Phase 2: reset one pointer to head, step both by 1, they meet at the start.
      let ptr: ListNode | null = head;
      while (ptr !== slow) {
        ptr = ptr!.next;
        slow = slow!.next;
      }
      return ptr;
    }
  }
  return null;
}
```

---

## 5. Complexity

| Technique | Time | Space | Note |
|---|---|---|---|
| Traverse / search | O(n) | O(1) | one pass |
| Reverse (iterative) | O(n) | **O(1)** | the whole point — no extra list |
| Reverse (recursive) | O(n) | O(n) | call stack — see [11-recursion-and-backtracking.md](11-recursion-and-backtracking.md) |
| Find middle (fast/slow) | O(n) | O(1) | one pass, no length count |
| Cycle detection (Floyd's) | O(n) | **O(1)** | vs O(n) space for a `Set` of seen nodes |
| Merge two sorted lists | O(n + m) | O(1) | rewire, don't copy |

> **The recurring win:** fast & slow gives you **O(1) space** where the "obvious" solution (store every node you've seen in a `Set`, check for repeats) costs **O(n) space**. Interviewers love asking you to cut the space — this is the answer.

---

## 6. Worked Example (classic LeetCode)

### 6a. Reverse Linked List — [LeetCode 206](https://leetcode.com/problems/reverse-linked-list/) (Easy)

> Given the head of a singly linked list, reverse it and return the new head.

Let's trace `reverseList` on **`1 → 2 → 3 → null`**. Watch `prev` and `curr` move, and note how line 1 (*save next*) rescues us every iteration.

**Start:** `prev = null`, `curr = 1`

```
prev   curr
 │      │
null    [1]──▶[2]──▶[3]──▶ null
```

**Iteration 1:** `next = 2`; `1.next = null`; `prev = 1`; `curr = 2`

```
        prev   curr
         │      │
null ◀──[1]    [2]──▶[3]──▶ null
```

**Iteration 2:** `next = 3`; `2.next = 1`; `prev = 2`; `curr = 3`

```
              prev   curr
               │      │
null ◀──[1]◀──[2]    [3]──▶ null
```

**Iteration 3:** `next = null`; `3.next = 2`; `prev = 3`; `curr = null`

```
                    prev    curr
                     │       │
null ◀──[1]◀──[2]◀──[3]     null
```

`curr === null` → **stop.** Return `prev = 3`. The list is now **`3 → 2 → 1 → null`**. ✅

> **The one bug everyone writes first:** setting `curr.next = prev` *before* saving `next`. The instant you do that, the pointer to the rest of the list is gone forever — you've sawed off the branch you're sitting on. **Always save `next` first.**

### 6b. Linked List Cycle — [LeetCode 141](https://leetcode.com/problems/linked-list-cycle/) (Easy)

> Return `true` if the list has a cycle (some node's `next` points back to an earlier node).

Consider **`3 → 2 → 0 → -4`**, where `-4.next` points back to `2` (a loop):

```
 [3]──▶[2]──▶[0]──▶[-4]
         ▲                │
         └────────────────┘
```

Run `hasCycle` (slow +1, fast +2 each tick):

| Tick | slow at | fast at | Collided? |
|---|---|---|---|
| start | 3 | 3 | — |
| 1 | 2 | 0 | no |
| 2 | 0 | 2 | no |
| 3 | -4 | -4 | **yes → return true** |

**Why does this *have* to work?** Think of a circular running track. The hare (fast) gains exactly **one step per tick** on the tortoise (slow), because fast moves 2 and slow moves 1 (net +1). Once both are inside the loop, the gap between them shrinks by exactly 1 every tick: `gap → gap-1 → gap-2 → … → 0`. Because it decreases by **exactly one** (never skips), it *must* eventually hit 0 — the moment they land on the same node. And if there's **no** loop, `fast` simply runs off the end (`fast` or `fast.next` becomes `null`) and we return `false`. There's no third possibility. That's the whole magic.

**Bonus — where does the cycle start?** After they collide, reset one pointer to `head` and advance both by 1; they meet at the cycle's entrance (that's `detectCycle` above — [LeetCode 142](https://leetcode.com/problems/linked-list-cycle-ii/)). The reason is a short piece of algebra: if `a` = distance from head to the loop start and `b` = distance the pointers traveled inside the loop before colliding, Floyd's two-speed setup forces `a` to equal the remaining distance from the collision point back around to the start. So two pointers stepping 1-at-a-time from `head` and from the collision point cover the same distance and meet exactly at the entrance.

---

## 7. Practice Set (curated LeetCode)

Do them in this order — each builds on the last.

| # | Problem | Difficulty | Technique it drills |
|---|---|---|---|
| 1 | [206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/) | Easy | Iterative rewiring (the foundation) |
| 2 | [876. Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/) | Easy | Fast & slow (find middle) |
| 3 | [141. Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/) | Easy | Fast & slow (detect cycle) |
| 4 | [21. Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/) | Easy | Dummy head + two pointers |
| 5 | [19. Remove Nth Node From End](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) | Medium | Dummy head + fast/slow gap |
| 6 | [142. Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/) | Medium | Floyd's, find cycle start |
| 7 | [234. Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/) | Easy | Find middle + reverse half |
| 8 | [143. Reorder List](https://leetcode.com/problems/reorder-list/) | Medium | Middle + reverse + merge (combo) |

**Merge Two Sorted Lists** is worth internalizing — it's the dummy-head pattern in its purest form:

```typescript
function mergeTwoLists(l1: ListNode | null, l2: ListNode | null): ListNode | null {
  const dummy = new ListNode(0);
  let tail = dummy;

  while (l1 !== null && l2 !== null) {
    if (l1.val <= l2.val) {
      tail.next = l1;   // splice the smaller node on
      l1 = l1.next;
    } else {
      tail.next = l2;
      l2 = l2.next;
    }
    tail = tail.next;
  }
  tail.next = l1 !== null ? l1 : l2; // attach whatever's left (already sorted)
  return dummy.next;
}
```

**Remove Nth From End** shows the dummy + gap combo — advance `fast` by `n` first, then move both until `fast` hits the end; `slow` lands right *before* the target:

```typescript
function removeNthFromEnd(head: ListNode | null, n: number): ListNode | null {
  const dummy = new ListNode(0, head); // dummy handles "remove the head" cleanly
  let fast: ListNode | null = dummy;
  let slow: ListNode | null = dummy;

  for (let i = 0; i < n; i++) fast = fast!.next; // open a gap of n
  while (fast!.next !== null) {                  // move both until fast is last
    fast = fast!.next;
    slow = slow!.next;
  }
  slow!.next = slow!.next!.next; // skip the target node
  return dummy.next;
}
```

---

## 8. Common Pitfalls & Interview Tips

- **Losing the `next` pointer before rewiring.** The #1 bug. Before you overwrite `node.next`, save it in a temp variable. In reversal, `const next = curr.next` must come *first*.
- **Null checks / walking off the end.** Always guard `fast !== null && fast.next !== null` before doing `fast.next.next`. Reading `.next` of `null` throws `TypeError: Cannot read properties of null`. Test the empty list (`head === null`) and single-node list.
- **Forgetting the dummy head.** Any time you might insert or delete at the *front*, or you're building a result list, start with `const dummy = new ListNode(0)` and return `dummy.next`. It erases the "what if it's the first node?" special case.
- **The TS non-null assertion `!`.** `slow!.next` tells the compiler "trust me, not null here." It's fine when the loop condition *logically* guarantees it (as with fast/slow), but a lazy `!` can hide a real null bug. Prefer narrowing (`if (curr !== null)`) when the guarantee isn't obvious. A JS runtime won't warn you — only TS will, and only if you don't silence it with `!`.
- **Building a cycle in your own test data.** To test cycle code you must *manually* wire `tail.next = someEarlierNode`. Don't `console.log` a cyclic list — it loops forever (or blows up). Walk a bounded number of steps instead.
- **Reversing recursively blows the stack on long lists.** The recursive reversal is elegant but uses O(n) call-stack space; on a 100k-node list you can hit `Maximum call stack size exceeded`. Prefer the iterative O(1)-space version in interviews. (More on the call stack in [10-stacks-and-queues.md](10-stacks-and-queues.md).)
- **State your approach out loud first.** Say "I'll use a dummy head and a fast/slow pointer" *before* coding. Draw the 3-node box diagram on the whiteboard — interviewers grade whether your pointers stay consistent, and the picture keeps you honest.

---

## 9. TL;DR

- A **linked list** = nodes of `{ val, next }`. Arrays win at **O(1) random access**; linked lists win at **O(1) insert/delete when you already hold the node**. There's no built-in list in JS/TS — you define `class ListNode`.
- **Singly** = forward only; **doubly** = `prev` + `next` (backbone of LRU cache).
- **Three moves solve almost everything:**
  1. **Dummy head** — a sentinel before the real head; kills front-of-list edge cases.
  2. **Iterative reversal** — save `next`, rewire backward, advance both. O(1) space.
  3. **Fast & slow pointers** — one hops 1, the other hops 2. Finds the **middle**, **detects cycles** (they must collide inside a loop), and locates the **cycle start** — all in O(1) space.
- **Never overwrite `next` before saving it.** Always null-check before `.next.next`.
- Fast & slow is a member of the pointer family → **[04-two-pointers.md](04-two-pointers.md)**. Next up, the other linear structures → **[10-stacks-and-queues.md](10-stacks-and-queues.md)**.
