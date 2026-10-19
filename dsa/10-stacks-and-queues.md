# 10. Stacks & Queues

> **One-liner:** A **stack** is last-in-first-out (a pile of plates); a **queue** is first-in-first-out (a line at the billing counter). Same job — hold things in order — opposite exit doors.
> **Pattern power:** Once you spot "matching pairs", "nesting/undo", or "next greater/smaller element", a stack turns an O(n²) brute force into a clean **O(n)** one-pass — that's the **monotonic stack**.

---

## 1. The Idea in Plain Words

### Stack — Last In, First Out (LIFO)

Picture a stack of plates. You add to the **top** and you remove from the **top**. The last plate you put on is the first one you take off. Two operations, both touching only the top:

- **push** — put an item on top
- **pop** — take the top item off (and return it)
- (**peek/top** — look at the top without removing it)

You already live inside a stack every day: the **JS call stack**. When function `a()` calls `b()` which calls `c()`, the engine *pushes* a frame for each call; when `c` returns it *pops*. Recurse too deep and you get the famous `RangeError: Maximum call stack size exceeded` — a literal **stack overflow**. Your browser's Back button is a stack too: each page you visit is pushed; Back pops the most recent.

### Queue — First In, First Out (FIFO)

Picture the line at a Zepto delivery counter. You join at the **back**; you get served from the **front**. First to arrive, first to leave. Fair ordering.

- **enqueue** — add to the back
- **dequeue** — remove from the front (and return it)
- (**front/peek** — look at the front without removing)

You also already use a queue every day: the **JS event loop's callback queue**. When a `setTimeout` fires or a click handler is ready, its callback joins the back of the task queue and the event loop pulls from the front, one at a time, in arrival order. That FIFO discipline is why your handlers run in the order events happened.

Queues are the engine of **BFS (breadth-first search)** — exploring a graph or tree level by level. When you get to graphs, the queue *is* the algorithm → cross-link **[16-graphs-bfs-dfs.md](16-graphs-bfs-dfs.md)**.

```
STACK (LIFO)                 QUEUE (FIFO)
 push ▲  ▼ pop                enqueue ▶        ▶ dequeue
     ┌───┐                            ┌───┬───┬───┐
 top │ C │  ← last in, first out      │ A │ B │ C │
     ├───┤                            └───┴───┴───┘
     │ B │                             ▲front    ▲back
     ├───┤                          first in, first out
     │ A │  ← first in, last out
     └───┘
```

---

## 2. When to Use It — Pattern Triggers

| If the problem says… | Reach for | Chapter link |
|---|---|---|
| **"matching pairs"**, "valid parentheses", "balanced brackets" | **Stack** (push openers, pop on closers) | this chapter |
| **"nesting"**, "undo/redo", "innermost", "most recent" | **Stack** (LIFO is literally "most recent") | this chapter |
| **"next greater / next smaller element"**, "warmer day", "span" | **Monotonic stack** | this chapter |
| **"largest rectangle in histogram"**, "trapping rain water" | **Monotonic stack** | this chapter |
| **"evaluate an expression"** / Reverse Polish Notation | **Stack** (operands wait, operators fire) | this chapter |
| **"simulate recursion"** without recursing (avoid stack overflow) | **Explicit stack** | [11-recursion-and-backtracking.md](11-recursion-and-backtracking.md) |
| **"level by level"**, "shortest path (unweighted)", "spread/flood" | **Queue** (BFS) | [16-graphs-bfs-dfs.md](16-graphs-bfs-dfs.md) |

> **The tell for a monotonic stack:** the phrase **"next greater / next smaller"** (or "previous greater/smaller"). If your brain jumps to "nested loop comparing every pair" — O(n²) — pause. A monotonic stack almost always collapses it to O(n).

---

## 3. Picture It (Diagram)

**Valid parentheses** — push each opener; when a closer arrives, the top *must* be its match, then pop. Empty stack at the end = balanced.

```
Input:  ( [ { } ] )

read '('  push        stack: [ (        ]
read '['  push        stack: [ ( [      ]
read '{'  push        stack: [ ( [ {    ]
read '}'  top is '{' ✓ pop  stack: [ ( [      ]
read ']'  top is '[' ✓ pop  stack: [ (        ]
read ')'  top is '(' ✓ pop  stack: [          ]   ← empty ⇒ VALID ✅
```

**Monotonic (decreasing) stack** — we keep the stack's values **decreasing top-to-bottom**. When a new, bigger value arrives, it "resolves" everything smaller than it by popping — each popped element just found its **next greater element**:

```
nums = [2, 1, 3]         stack holds INDICES; values below are decreasing

i=0 (val 2)  push 0            stack(vals): [2]
i=1 (val 1)  1 < 2, push 1     stack(vals): [2,1]
i=2 (val 3)  3 > 1 → pop → ans[1]=3
             3 > 2 → pop → ans[0]=3
             push 2            stack(vals): [3]
```

The genius: every index is **pushed once and popped once**. That's why the whole scan is O(n), not O(n²), even though there's a loop inside a loop.

---

## 4. The Template (TypeScript)

### Stack — just use a `number[]` with `push` / `pop`

This is the big JS/TS win: an array *is* a ready-made stack, and both ends operations you need are O(1).

```typescript
const stack: number[] = [];

stack.push(10);                       // O(1) — add to top
stack.push(20);
const top = stack[stack.length - 1];  // O(1) — peek (does NOT remove) → 20
const out = stack.pop();              // O(1) — remove top → 20
const empty = stack.length === 0;     // check before you peek/pop!
```

> `push` and `pop` operate on the **end** of the array, which needs no re-indexing → **O(1)**. This is why "array as a stack" is free.

### Queue — do NOT use `shift()`; use an index pointer

Here's the trap the other guides skip. You *can* write a queue as `queue.push(x)` + `queue.shift()`, but **`array.shift()` is O(n)**: removing element 0 forces JS to slide every remaining element down one index. In a loop that's a hidden O(n²). (Same reason `unshift()` is O(n).) `push`/`pop` are fine; `shift`/`unshift` are the slow ones.

**Fix: never actually remove the front — just move a `head` index forward.** This makes dequeue O(1):

```typescript
class Queue<T> {
  private items: T[] = [];
  private head = 0; // index of the current front

  enqueue(x: T): void {
    this.items.push(x);            // O(1)
  }

  dequeue(): T | undefined {
    if (this.head >= this.items.length) return undefined; // empty
    const x = this.items[this.head];
    this.head++;                   // O(1) — just advance, don't splice
    // Optional: reclaim memory once we've consumed a lot of the array.
    if (this.head > 1000 && this.head * 2 >= this.items.length) {
      this.items = this.items.slice(this.head);
      this.head = 0;
    }
    return x;
  }

  peek(): T | undefined {
    return this.head < this.items.length ? this.items[this.head] : undefined;
  }

  get size(): number { return this.items.length - this.head; }
  get isEmpty(): boolean { return this.size === 0; }
}
```

> **Interview line to say out loud:** "I'll back the queue with an array plus a head index, because `Array.prototype.shift()` is O(n) and would make this O(n²)." That one sentence signals you know the JS internals. (In some ecosystems you'd reach for a **deque** / doubly linked list — see **[09-linked-lists.md](09-linked-lists.md)** — but the index trick is simplest and enough for interviews.)

### Monotonic stack — the reusable skeleton

Keep indices on the stack; maintain a monotonic order; pop when the new element breaks it. This one skeleton solves "next greater", "daily temperatures", and friends:

```typescript
function nextGreaterElements(nums: number[]): number[] {
  const n = nums.length;
  const res = new Array<number>(n).fill(-1); // default: no greater element
  const stack: number[] = [];                // INDICES; values decreasing top→bottom

  for (let i = 0; i < n; i++) {
    // While current value beats the value at the top index, we've found ITS answer.
    while (stack.length > 0 && nums[i] > nums[stack[stack.length - 1]]) {
      const idx = stack.pop()!;
      res[idx] = nums[i];
    }
    stack.push(i);
  }
  return res; // indices left on the stack keep -1 (nothing greater to their right)
}
```

> **Store indices, not values.** You can always read the value back with `nums[idx]`, but the index also gives you *distances* (as in Daily Temperatures) and positions. Storing raw values throws that away — a classic bug.

---

## 5. Complexity

| Operation | Structure | Time | Note |
|---|---|---|---|
| push / pop / peek | Stack (array) | **O(1)** | end-of-array ops, no shifting |
| enqueue | Queue (array) | **O(1)** | `push` |
| dequeue via **`shift()`** | Queue (array) | **O(n)** ⚠️ | re-indexes everything — avoid |
| dequeue via **head index** | Queue | **O(1)** ✅ | just move a pointer |
| Valid parentheses | Stack | O(n) time, O(n) space | one pass |
| Monotonic stack scan | Stack | **O(n)** time, O(n) space | each index pushed & popped once |

> **Why the monotonic stack is O(n) despite a nested `while`:** amortized analysis. Across the *entire* run, every index is pushed exactly once and popped at most once — that's at most `2n` stack operations total, no matter how the `while` loop bunches up. The inner loop can't run more times overall than there are elements. (More on amortized cost in [01-big-o-and-problem-solving-framework.md](01-big-o-and-problem-solving-framework.md).)

---

## 6. Worked Example (classic LeetCode)

### 6a. Valid Parentheses — [LeetCode 20](https://leetcode.com/problems/valid-parentheses/) (Easy)

> Given a string of `()[]{}`, decide if every bracket is closed by the correct type in the correct order.

```typescript
function isValid(s: string): boolean {
  const stack: string[] = [];
  const match: Record<string, string> = { ')': '(', ']': '[', '}': '{' };

  for (const ch of s) {
    if (ch === '(' || ch === '[' || ch === '{') {
      stack.push(ch);                          // opener → remember it
    } else {
      // closer: the top MUST be its matching opener
      if (stack.pop() !== match[ch]) return false; // covers empty stack too:
    }                                              // pop() on [] is undefined ≠ opener
  }
  return stack.length === 0; // leftover openers ⇒ not balanced
}
```

Trace on **`"{[()]}"`** (should be `true`):

| Read | Action | Stack (bottom→top) |
|---|---|---|
| `{` | push | `{` |
| `[` | push | `{ [` |
| `(` | push | `{ [ (` |
| `)` | pop `(`, matches `match[')']='('` ✓ | `{ [` |
| `]` | pop `[`, matches ✓ | `{` |
| `}` | pop `{`, matches ✓ | *(empty)* |

End: stack empty → **`true`** ✅

Now trace the failure **`"(]"`**: push `(`; read `]`, pop `(`, but `match[']'] === '['` and `'(' !== '['` → **`false`** immediately. And **`"("`** alone ends with `[ ( ]` still on the stack → `stack.length !== 0` → **`false`**. The two failure modes — *wrong match* and *leftovers* — are both covered.

> Why a stack and not a counter? A single counter works for one bracket type, but `([)]` needs to know the **most recent** unclosed opener — that's LIFO, exactly what a stack gives you.

### 6b. Daily Temperatures — [LeetCode 739](https://leetcode.com/problems/daily-temperatures/) (Medium)

> For each day, how many days until a **warmer** temperature? If none, `0`. This is "next greater element" wearing a costume.

The brute force is a nested loop: for each day, scan forward for a warmer one — **O(n²)**. A monotonic stack does it in **O(n)**. Keep a stack of **indices** whose temperatures are still waiting for a warmer day (decreasing temps top-to-bottom). When today is warmer than the day on top, we've resolved that day — the distance is `today - thatDay`.

```typescript
function dailyTemperatures(temps: number[]): number[] {
  const n = temps.length;
  const answer = new Array<number>(n).fill(0);
  const stack: number[] = []; // indices of days awaiting a warmer day

  for (let i = 0; i < n; i++) {
    while (stack.length > 0 && temps[i] > temps[stack[stack.length - 1]]) {
      const prevDay = stack.pop()!;
      answer[prevDay] = i - prevDay; // distance in days
    }
    stack.push(i);
  }
  return answer;
}
```

Trace on **`[73, 74, 75, 71, 69, 72, 76, 73]`**:

| i | temp | Pops (day → answer) | Stack after (indices) |
|---|---|---|---|
| 0 | 73 | — | `[0]` |
| 1 | 74 | 0 → `ans[0]=1` | `[1]` |
| 2 | 75 | 1 → `ans[1]=1` | `[2]` |
| 3 | 71 | — | `[2,3]` |
| 4 | 69 | — | `[2,3,4]` |
| 5 | 72 | 4 → `ans[4]=1`, 3 → `ans[3]=2` | `[2,5]` |
| 6 | 76 | 5 → `ans[5]=1`, 2 → `ans[2]=4` | `[6]` |
| 7 | 73 | — | `[6,7]` |

Days 6 and 7 never get a warmer day, so they keep `0`. Result: **`[1, 1, 4, 2, 1, 1, 0, 0]`** ✅ — and every index was pushed and popped at most once, so it's a single O(n) sweep.

---

## 7. Practice Set (curated LeetCode)

| # | Problem | Difficulty | Skill it drills |
|---|---|---|---|
| 1 | [20. Valid Parentheses](https://leetcode.com/problems/valid-parentheses/) | Easy | Stack for matching pairs |
| 2 | [155. Min Stack](https://leetcode.com/problems/min-stack/) | Medium | Track the min in O(1) with a second stack |
| 3 | [739. Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) | Medium | Monotonic stack (indices) |
| 4 | [496. Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/) | Easy | Monotonic stack + a hash map |
| 5 | [150. Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/) | Medium | Stack as an evaluator |
| 6 | [232. Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks/) | Easy | Two stacks → a FIFO queue |
| 7 | [84. Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/) | Hard | Monotonic stack, the boss level |

**Min Stack** — push the running minimum alongside each value so `getMin()` is O(1):

```typescript
class MinStack {
  private stack: number[] = [];
  private mins: number[] = []; // mins[i] = smallest value in stack[0..i]

  push(x: number): void {
    this.stack.push(x);
    const prevMin = this.mins.length === 0 ? x : this.mins[this.mins.length - 1];
    this.mins.push(Math.min(x, prevMin));
  }
  pop(): void { this.stack.pop(); this.mins.pop(); }
  top(): number { return this.stack[this.stack.length - 1]; }
  getMin(): number { return this.mins[this.mins.length - 1]; }
}
```

**Evaluate RPN** — operands wait on the stack; an operator pops two and pushes the result. Note the **pop order** (`b` first, then `a`) — it matters for `-` and `/`:

```typescript
function evalRPN(tokens: string[]): number {
  const stack: number[] = [];
  const ops = new Set(['+', '-', '*', '/']);

  for (const t of tokens) {
    if (!ops.has(t)) { stack.push(Number(t)); continue; }
    const b = stack.pop()!; // second operand popped FIRST
    const a = stack.pop()!;
    if (t === '+') stack.push(a + b);
    else if (t === '-') stack.push(a - b);
    else if (t === '*') stack.push(a * b);
    else stack.push(Math.trunc(a / b)); // truncate toward zero, not Math.floor
  }
  return stack[0];
}
```

**Implement Queue using Stacks** — an `in` stack for pushes and an `out` stack for pops; move everything over only when `out` runs dry, which makes each element cross once (amortized O(1)):

```typescript
class MyQueue {
  private inStack: number[] = [];
  private outStack: number[] = [];

  push(x: number): void { this.inStack.push(x); }

  private transfer(): void {
    if (this.outStack.length === 0) {
      while (this.inStack.length > 0) this.outStack.push(this.inStack.pop()!);
    }
  }
  pop(): number { this.transfer(); return this.outStack.pop()!; }
  peek(): number { this.transfer(); return this.outStack[this.outStack.length - 1]; }
  empty(): boolean { return this.inStack.length === 0 && this.outStack.length === 0; }
}
```

---

## 8. Common Pitfalls & Interview Tips

- **`array.shift()` / `unshift()` are O(n).** The single most important takeaway. Using `shift()` to dequeue inside a loop silently makes your solution O(n²). Use a **head index** (or a deque). `push`/`pop` are O(1) — those are safe.
- **Peeking or popping an empty stack.** `[].pop()` returns `undefined` (no error), which can quietly poison your logic. Always guard `if (stack.length > 0)` before you rely on the top — or exploit it deliberately, like `stack.pop() !== match[ch]` in Valid Parentheses where `undefined` correctly fails the match.
- **Monotonic stack: indices vs values.** Push **indices** by default. You can recover the value via `nums[idx]`, but only the index gives you distances (Daily Temperatures) and positions (histogram widths). Decide *up front* which you store and stay consistent.
- **Which direction is the stack monotonic?** "Next greater" → keep values **decreasing** and pop when a bigger one arrives. "Next smaller" → keep **increasing**, pop when a smaller one arrives. Getting this backwards is the classic monotonic bug — write the two-line invariant in a comment before you code.
- **RPN operand order.** Pop `b` then `a`, and compute `a - b` / `a / b` (not `b - a`). For integer division use `Math.trunc` (toward zero), because `Math.floor(-7 / 2)` gives `-4`, while most RPN specs want `-3`.
- **Recursion IS a stack.** If a recursive solution risks `Maximum call stack size exceeded` on large input, you can convert it to an **explicit stack** loop — same traversal, heap memory instead of the call stack. This bridges directly to DFS in [11-recursion-and-backtracking.md](11-recursion-and-backtracking.md).
- **Say the pattern name.** "This is a monotonic stack" or "I'll use a stack because I need the most-recent unmatched item." Naming the tool up front is worth real points — it shows pattern recognition, which is the whole game.

---

## 9. TL;DR

- **Stack = LIFO** (plates, Back button, the JS call stack). **Queue = FIFO** (billing line, the event loop's task queue, BFS).
- In TS, an **array is a stack for free**: `push`/`pop`/`arr[arr.length-1]` are all **O(1)**.
- **Never dequeue with `shift()`** — it's O(n). Back a queue with an array + a **head index** for O(1) dequeue (or use a deque).
- **Monotonic stack** is the star pattern: it turns "next greater/smaller element", "daily temperatures", and "largest rectangle" from O(n²) into **O(n)**, because each index is pushed and popped once. **Store indices**, and pin down the increasing-vs-decreasing invariant before coding.
- **Stacks parse and match** (valid parentheses, RPN); **queues spread level-by-level** (BFS).
- Related structures: linked lists (a deque can be a doubly linked list) → **[09-linked-lists.md](09-linked-lists.md)**; the queue powers graph traversal → **[16-graphs-bfs-dfs.md](16-graphs-bfs-dfs.md)**.
