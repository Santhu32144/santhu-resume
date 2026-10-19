# 18. Dynamic Programming I — Memoization, Tabulation & 1D DP

> **One-liner:** Dynamic Programming (DP) is just **recursion that remembers** — you break a problem into smaller versions of itself, and you write down each answer the first time so you never solve the same subproblem twice.
> **Pattern power:** DP turns exponential brute-force (O(2ⁿ)) into linear or polynomial time by *reusing* work. Once you internalise the 5-step recipe here, "scary DP" becomes a fill-in-the-blanks exercise — and it unlocks a huge slice of medium/hard interview questions at CRED, Razorpay, Groww and FAANG.

---

Take a breath. **DP is the topic that scares people the most, and it's mostly a marketing problem** — the name "dynamic programming" sounds like rocket science. It isn't. If you're comfortable with recursion (from [11 — Recursion & Backtracking](11-recursion-and-backtracking.md)), you are 80% of the way there. The other 20% is one idea: *remember answers you've already computed.* That's it. This chapter proves it to you.

---

## 1. The Idea in Plain Words

Here is the entire concept in one sentence, and you should tattoo it on your brain:

> **Dynamic Programming = Recursion + Memory.** Solve a big problem by solving smaller sub-versions of the *same* problem, and store each sub-answer so you compute it only once.

Two conditions must be true for DP to apply. Interviewers love asking "why is this DP?" — the answer is always these two:

| Property | Plain-English meaning | Quick test |
|---|---|---|
| **Optimal substructure** | The best answer to the big problem is built from the best answers to smaller problems. | "If I knew the answer for `n-1`, could I get `n` cheaply?" |
| **Overlapping subproblems** | The naive recursion solves the *same* smaller problem again and again. | "Does my recursion tree repeat the same call?" |

If **both** hold, DP will save you. If subproblems *don't* overlap (every branch is unique), memory buys you nothing and it's plain divide-and-conquer (like merge sort in [08 — Sorting](08-sorting.md)). If there's no optimal substructure, it's probably [backtracking](11-recursion-and-backtracking.md) or [greedy](20-greedy-and-intervals.md) territory.

**The mental model that makes it click.** Imagine climbing a staircase and someone keeps asking "how many ways to reach step 5?" A naive you re-counts from scratch every time. A smart you writes each answer on a sticky note stuck to that step. Next time anyone asks about step 5, you just read the note. **The sticky notes are your DP table.** Memoization is nothing more fancy than that.

Think of it like React's `useMemo`: you don't recompute an expensive value if its inputs haven't changed — you return the cached result. DP is `useMemo` for recursive subproblems.

---

## 2. When to Use It — Pattern Triggers

You do **not** need to invent DP from nothing. You need to *recognise the smell*. Scan the problem statement for these signals:

| Trigger phrase in the problem | Why it's DP |
|---|---|
| "**Count the number of ways** to…" | Ways = sum of ways of sub-choices. Classic DP. |
| "**Minimum / maximum** cost / value / length…" with **choices** at each step | Optimize over choices → compare sub-answers. |
| "Can you **reach / make / form** X?" (true/false) | Boolean DP: reachable if some sub-state is reachable. |
| "**Longest / shortest** … subsequence / path" | Build the best answer from smaller prefixes. |
| You wrote a recursion and it **times out** / recomputes the same call | Add memory → it's DP. |
| Choices look like "**take it or skip it**" at each index | The take/skip template (House Robber, Knapsack — see [19](19-dynamic-programming-2.md)). |

**The two big families you'll meet 90% of the time:**

1. **"Number of ways"** → you *add* the ways from each choice. (Climbing Stairs, Decode Ways, Coin Change II.)
2. **"Min / max of something with choices"** → you take `Math.min`/`Math.max` over choices. (House Robber, Coin Change, Min Cost Climbing Stairs.)

**Anti-trigger (when it's NOT DP):** if a simple locally-optimal choice provably works and you never need to reconsider it, that's **greedy** ([20](20-greedy-and-intervals.md)), which is cheaper. DP is what you use when greedy is *wrong* because a choice now affects choices later.

---

## 3. Picture It (Diagram)

The whole reason DP exists is visible in one picture: the **naive Fibonacci recursion tree**. Watch how `fib(2)` and `fib(3)` get recomputed over and over.

```
                          fib(5)
                 /                     \
             fib(4)                    fib(3)          ← fib(3) computed here...
           /        \                 /       \
       fib(3)      fib(2)          fib(2)    fib(1)    ← ...AND again here
       /    \       /   \          /   \
   fib(2)  fib(1) fib(1) fib(0) fib(1) fib(0)
   /    \
 fib(1) fib(0)
```

Count the repeats: `fib(3)` twice, `fib(2)` three times, `fib(1)` five times. For `fib(50)` the naive tree has **~2⁵⁰ ≈ 10¹⁵ nodes** — it will hang your interview laptop. Every one of those repeats is wasted work.

**Memoization prunes the tree.** The first time we compute `fib(3)` we cache it; the second time we return the cached value instantly (the whole subtree below it never runs):

```
   Naive tree (exponential)          Memoized (linear)
   -------------------------         ------------------------------
            fib(5)                   fib(5) → needs fib(4), fib(3)
           /      \                  fib(4) → computed once, cached
      fib(4)      fib(3) ⟳ repeat    fib(3) → CACHE HIT ✓ (no subtree)
      /    \      (whole subtree     fib(2) → computed once, cached
   fib(3)  ...     recomputed)       fib(1), fib(0) → base cases
     ⟳
```

We go from an exploding tree to a straight line of `n` unique subproblems. **That collapse — from O(2ⁿ) to O(n) — is the entire payoff of DP.**

---

## 4. The Template (TypeScript)

### The DP Recipe (memorise these 5 steps — say them out loud in interviews)

> **1. Define the state in words.** What does `dp[i]` (or `f(i)`) *mean*? e.g. "`dp[i]` = the number of ways to reach step `i`." Be precise — a fuzzy state definition is the #1 cause of DP bugs.
> **2. Write the recurrence (transition).** How does `dp[i]` depend on smaller states? e.g. `dp[i] = dp[i-1] + dp[i-2]`.
> **3. Base cases.** The smallest states you can answer directly. e.g. `dp[0] = 1`, `dp[1] = 1`.
> **4. Choose memoization (top-down) or tabulation (bottom-up).** Same recurrence, two ways to run it.
> **5. Answer location.** Which cell holds the final answer? e.g. `dp[n]`. (Sometimes it's `Math.max(...dp)`, not the last cell — decide consciously.)

### Style A — Top-Down (memoization): write the recursion, then cache it

```typescript
// Generic memoization skeleton — this shape solves most 1D DP problems.
function solveTopDown(n: number): number {
  const memo = new Map<number, number>(); // the "sticky notes"

  function f(i: number): number {
    // 3. Base cases
    if (i <= 1) return i;

    // Cache check — the ONE line that turns O(2^n) into O(n)
    if (memo.has(i)) return memo.get(i)!;

    // 2. Recurrence
    const result = f(i - 1) + f(i - 2);

    memo.set(i, result); // write the sticky note
    return result;
  }

  return f(n); // 5. answer
}
```

Top-down is the closest to how you *think*: write the recursion naturally, then drop in a cache. It only computes the subproblems it actually needs (lazy).

### Style B — Bottom-Up (tabulation): fill an array from the base cases up

```typescript
function solveBottomUp(n: number): number {
  if (n <= 1) return n;
  const dp = new Array<number>(n + 1).fill(0); // 1D array init — see gotcha below
  dp[0] = 0;
  dp[1] = 1;               // 3. base cases
  for (let i = 2; i <= n; i++) {
    dp[i] = dp[i - 1] + dp[i - 2]; // 2. recurrence, filled in order
  }
  return dp[n];            // 5. answer
}
```

Bottom-up has **no recursion**, so no stack-overflow risk, and it's usually a touch faster. The price: you must figure out the *fill order* (small states before the states that depend on them).

### Style C — Space-optimised (rolling variables)

When `dp[i]` only looks back a fixed distance (here, 2 cells), you don't need the whole array — just carry the last few values:

```typescript
function solveRolling(n: number): number {
  if (n <= 1) return n;
  let prev2 = 0, prev1 = 1;       // dp[i-2], dp[i-1]
  for (let i = 2; i <= n; i++) {
    const cur = prev1 + prev2;    // dp[i]
    prev2 = prev1;                // slide the window forward
    prev1 = cur;
  }
  return prev1;
}
```

This drops space from O(n) to **O(1)**. Interviewers love it: "can you do it in constant space?" — for take/skip and Fibonacci-shaped DPs, the answer is yes.

> ⚠️ **JS/TS gotcha — 1D array init.** Use `new Array(n + 1).fill(0)` to get a real, writable array of zeros. **Never** rely on `new Array(n + 1)` alone — that gives you `n+1` *empty slots*, and `arr[i] += 1` becomes `undefined + 1 = NaN`, which silently poisons your whole table. Always `.fill(...)` a 1D DP array before you read from it. (2D arrays have a nastier version of this trap — see [19 — DP II](19-dynamic-programming-2.md).)

---

## 5. Complexity

The beautiful thing about DP complexity: it's almost mechanical to compute.

> **Time = (number of distinct states) × (work per state/transition).**
> **Space = (size of the memo/table)**, before any rolling-variable optimisation.

| Version of Fibonacci | Time | Space | Why |
|---|---|---|---|
| Naive recursion | **O(2ⁿ)** | O(n) stack | Recomputes everything (see the tree) |
| Top-down memo | **O(n)** | O(n) map + O(n) stack | `n` distinct states, O(1) each |
| Bottom-up table | **O(n)** | O(n) | Same states, iterative |
| Rolling variables | **O(n)** | **O(1)** | Only keep the last 2 values |

For 1D DP problems in this chapter: **n states × O(1) transition = O(n) time**. When a transition itself loops (e.g. Coin Change tries every coin), it's **O(n × k)** where `k` is the number of choices.

> **Recursion-depth caveat (JS gotcha).** Top-down memo on a large `n` can blow the call stack — V8's default is roughly ~10⁴–10⁵ frames. If `n` can be big (say 10⁶), prefer **bottom-up tabulation**, which uses no recursion stack at all. This is a real, testable difference, not a nitpick.

---

## 6. Worked Example (a classic LeetCode problem)

We'll do three, building from the "hello world" of DP up to the take/skip template you'll reuse constantly.

### 6a. Fibonacci — the full journey (LeetCode 509)

We already saw all four versions above. The point of Fibonacci isn't the numbers — it's watching **the exact same recurrence** (`f(n) = f(n-1) + f(n-2)`) get faster and leaner as you apply the recipe: naive O(2ⁿ) → memo O(n) → tabulation O(n) → rolling O(1) space. Every DP problem is this same journey. Internalise it here where the math is trivial, so the machinery is automatic when the problem is hard.

### 6b. Climbing Stairs (LeetCode 70) — "count the ways"

> You climb a staircase of `n` steps. Each move you go up **1 or 2** steps. How many distinct ways to reach the top?

Run the recipe:

1. **State:** `dp[i]` = number of distinct ways to reach step `i`.
2. **Recurrence:** to land on step `i`, your last move came from step `i-1` (a 1-step) or step `i-2` (a 2-step). So `dp[i] = dp[i-1] + dp[i-2]`. (It's Fibonacci in disguise — that's why "count the ways" is a DP trigger.)
3. **Base cases:** `dp[0] = 1` (one way to stand at the bottom: do nothing), `dp[1] = 1`.
4. **Style:** tabulation, then rolling.
5. **Answer:** `dp[n]`.

```typescript
function climbStairs(n: number): number {
  if (n <= 2) return n;
  let oneStepBelow = 2; // ways to reach step 2
  let twoStepsBelow = 1; // ways to reach step 1
  for (let i = 3; i <= n; i++) {
    const cur = oneStepBelow + twoStepsBelow;
    twoStepsBelow = oneStepBelow;
    oneStepBelow = cur;
  }
  return oneStepBelow;
}
// climbStairs(5) === 8
```

**Trace for `n = 5`:** `dp = [_, 1, 2, 3, 5, 8]` → **8 ways.** Each cell is literally the sum of the two before it. If you can see why, you understand 1D DP.

### 6c. House Robber (LeetCode 198) — the take/skip template

> Houses in a row hold `nums[i]` cash. You **can't rob two adjacent houses** (alarms link neighbours). Maximise the loot.

This is the archetypal **"min/max with a take-or-skip choice"** DP — you will see this shape a dozen times.

1. **State:** `dp[i]` = the most money robbable from houses `0..i`.
2. **Recurrence — the choice:** at house `i` you either
   - **skip** it → you keep `dp[i-1]`, or
   - **rob** it → you get `nums[i]` **plus** `dp[i-2]` (you had to skip `i-1`).
   Take the better: `dp[i] = Math.max(dp[i-1], nums[i] + dp[i-2])`.
3. **Base cases:** `dp[0] = nums[0]`, `dp[1] = max(nums[0], nums[1])`.
4. **Style:** rolling variables (only look back 2).
5. **Answer:** `dp[n-1]`.

```typescript
function rob(nums: number[]): number {
  let prev2 = 0; // best loot up to house i-2
  let prev1 = 0; // best loot up to house i-1
  for (const cash of nums) {
    const cur = Math.max(prev1, prev2 + cash); // skip vs rob
    prev2 = prev1;
    prev1 = cur;
  }
  return prev1;
}
```

**Trace for `nums = [2, 7, 9, 3, 1]`:**

| House `i` | `nums[i]` | skip = `prev1` | rob = `prev2 + nums[i]` | `dp[i] = max` |
|---|---|---|---|---|
| 0 | 2 | 0 | 0 + 2 = 2 | **2** |
| 1 | 7 | 2 | 0 + 7 = 7 | **7** |
| 2 | 9 | 7 | 2 + 9 = 11 | **11** |
| 3 | 3 | 11 | 7 + 3 = 10 | **11** |
| 4 | 1 | 11 | 11 + 1 = 12 | **12** |

Answer: **12** (rob houses 0, 2, 4 → 2 + 9 + 1). Notice how at house 3 we *correctly skip* even though 3 is money on the table — greedy would have grabbed it and lost. **That's exactly why this needs DP, not greedy.**

---

## 7. Practice Set (curated LeetCode)

Do them in this order — each one reinforces the recipe with a slightly new twist. Every one is 1D DP.

| # | Problem | LeetCode | Difficulty | Family / twist |
|---|---|---|---|---|
| 1 | Climbing Stairs | 70 | Easy | Count-ways (Fibonacci shape) |
| 2 | Min Cost Climbing Stairs | 746 | Easy | Min-cost variant of #1 |
| 3 | House Robber | 198 | Medium | Take/skip template |
| 4 | House Robber II | 213 | Medium | Circular → run #3 twice (see note) |
| 5 | Decode Ways | 91 | Medium | Count-ways with validity conditions |
| 6 | Coin Change | 322 | Medium | **Min** coins (unbounded choices) |
| 7 | Coin Change II | 518 | Medium | **Count** combinations (order of loops matters!) |
| 8 | Word Break | 139 | Medium | Boolean DP over string prefixes |

**Note on House Robber II (circular street).** The first and last house are now adjacent, so you can't rob both. Trick: the max is either the best from houses `[0 … n-2]` **or** the best from `[1 … n-1]` — run your plain `rob` on each slice and take the larger. Reusing a solved subproblem twice — very interview-friendly.

**Coin Change (min) — a second take/min example to lock in the pattern:**

```typescript
function coinChange(coins: number[], amount: number): number {
  // dp[a] = fewest coins to make amount `a`; Infinity = impossible
  const dp = new Array<number>(amount + 1).fill(Infinity);
  dp[0] = 0; // base: 0 coins make amount 0
  for (let a = 1; a <= amount; a++) {
    for (const coin of coins) {
      if (coin <= a) dp[a] = Math.min(dp[a], dp[a - coin] + 1);
    }
  }
  return dp[amount] === Infinity ? -1 : dp[amount];
}
// coinChange([1,2,5], 11) === 3   (5 + 5 + 1)
```

Same recipe: state = "fewest coins for amount `a`", recurrence = "best over each coin you could add last", base = `dp[0] = 0`, answer = `dp[amount]`. Coin Change is the bridge to the **knapsack family** in [19 — DP II](19-dynamic-programming-2.md).

---

## 8. Common Pitfalls & Interview Tips

| Pitfall | What goes wrong | Fix |
|---|---|---|
| **Fuzzy state definition** | You can't write a correct recurrence because you're not sure what `dp[i]` *means*. This is the #1 DP bug. | Write the state in one English sentence *before* any code. Say it out loud. |
| **Wrong base cases** | Off-by-one at the boundary; e.g. `dp[0]` should be `1` (empty way) but you set `0`. | Hand-compute the two smallest inputs and make the table match them. |
| **Recomputation (forgot the cache)** | Top-down TLEs because you never `memo.set` / never check the cache. | The cache-check line is mandatory. No cache = it's not DP, it's the exponential tree. |
| **1D array init** | `new Array(n)` leaves empty slots; `arr[i] += x → NaN`. | Always `new Array(n).fill(0)` (or `false`/`Infinity`). |
| **Wrong answer location** | Returning `dp[n]` when the answer is `Math.max(...dp)` (e.g. LIS) — or vice-versa. | In step 5, consciously decide *which cell* holds the answer. |
| **Loop order in "count combinations"** | Coin Change II: looping amounts outer vs coins outer gives permutations, not combinations. | For *combinations*, loop **coins outer, amount inner**. |
| **Stack overflow on big `n`** | Deep top-down recursion crashes. | Switch to bottom-up tabulation. |

**Interview scripts that score points:**
- **Always start with the brute-force recursion out loud.** "Let me define `f(i)`… the naive version is exponential because it recomputes subproblems — so I'll memoize." This shows the interviewer you *understand* why DP, not just that you memorised a table.
- **Name the two properties:** "This has optimal substructure and overlapping subproblems, so it's DP."
- **Offer the space optimisation** as a follow-up: "I used O(n) space, but since I only look back two cells I can do O(1) with rolling variables."
- Reach for **top-down first** if the recurrence is easier to see recursively; convert to bottom-up if they ask about stack depth or you want the constant-space version.

> **Real-project analogy (Habex).** Computing your monthly spend-by-category on [habex.santhu.site](https://habex.santhu.site) is exactly memoization: compute the July summary once, cache it, and re-serve the cached value until the data changes — instead of re-summing every transaction on every dashboard load. Same instinct as DP: *don't redo work you've already done.*

---

## 9. TL;DR

- **DP = recursion + remembering.** Break the problem into smaller copies of itself, and cache each sub-answer so you solve it once.
- It applies when you have **optimal substructure** (big answer built from small answers) **and** **overlapping subproblems** (naive recursion repeats work).
- **Triggers:** "count the number of ways", "min/max with choices", "can you reach/form X", or a recursion that times out.
- **The 5-step recipe:** ① define the state in words → ② write the recurrence → ③ base cases → ④ memo (top-down) or table (bottom-up) → ⑤ locate the answer.
- **Top-down memoization** mirrors your thinking and is lazy; **bottom-up tabulation** avoids the recursion stack; **rolling variables** cut space to O(1) when you only look back a fixed distance.
- **Complexity:** time = states × work-per-state; 1D DP is usually **O(n)**.
- **JS gotchas:** always `.fill()` a 1D array; watch recursion depth on huge `n` (prefer tabulation).
- The **take/skip** template (House Robber) — `dp[i] = max(skip = dp[i-1], take = value + dp[i-2])` — recurs everywhere; know it cold.

**Where to go next:** [19 — Dynamic Programming II](19-dynamic-programming-2.md) takes the exact same recipe into **2D tables** (grid paths, two-string problems), the **knapsack** family, and **subsequence** DP (LCS, LIS). And if any of the recursion here felt shaky, warm up with [11 — Recursion & Backtracking](11-recursion-and-backtracking.md) first — DP *is* memoized recursion.
