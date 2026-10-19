# 19. Dynamic Programming II — 2D Grids, Knapsack & Subsequences

> **One-liner:** The same "recursion + remembering" idea from [18 — DP I](18-dynamic-programming-1.md), now with a **2D table** — because these problems have *two* things that vary (a row index and a column index), so a single number `dp[i]` isn't enough state to remember.
> **Pattern power:** 2D DP is the workhorse of hard interviews. Master three families — **grid paths**, **knapsack**, and **subsequence (two-string) DP** — and you can recognise and set up the table for almost any DP a product company (CRED, Razorpay, Groww, FAANG) throws at you.

---

You already own the engine. [DP I](18-dynamic-programming-1.md) gave you the **5-step recipe** (state → recurrence → base cases → memo/table → answer location) and the take/skip template. Nothing new is coming. The *only* upgrade in this chapter is: **the state now has two coordinates instead of one**, so `dp` becomes a grid. If you can fill a 1D array left-to-right, you can fill a 2D grid row-by-row. Let's make it routine.

---

## 1. The Idea in Plain Words

In [DP I](18-dynamic-programming-1.md), one index captured the whole subproblem: "ways to reach step `i`." But many problems have **two moving parts** at once:

- Walking a grid → you're at **row `i`, column `j`**.
- Comparing two strings → you've matched **`i` chars of string A and `j` chars of string B**.
- Packing a bag → you've considered **the first `i` items** with **capacity `w`** left.

Two moving parts ⇒ your "sticky note" needs two coordinates ⇒ **`dp[i][j]`**. That's the whole leap. The definition sentence just gets a bit longer:

> **`dp[i][j]` = the best/count answer for the sub-problem restricted to the first `i` of one thing and the first `j` (or capacity `w`) of another thing.**

**Three families cover the vast majority of 2D DP:**

| Family | State `dp[i][j]` means… | Signature move |
|---|---|---|
| **Grid paths** | best/count to reach cell `(i, j)` | come from **top or left** |
| **Knapsack** | best value using first `i` items with capacity `j` | **take item or skip item** |
| **Subsequence / two-string** | answer for `A[0..i)` vs `B[0..j)` | **characters match → extend; else skip one side** |

Everything below is a variation on these three. Learn the *shape of the table* for each and you're done.

---

## 2. When to Use It — Pattern Triggers

| Trigger in the problem | Family | Table shape |
|---|---|---|
| "**grid / matrix**, move right or down, count paths / min sum" | Grid paths | `dp[m][n]` over cells |
| "**two strings/arrays**, longest common / edit / align" | Subsequence DP | `dp[n+1][m+1]`, indices into each |
| "pick a **subset** to hit a **target sum / weight**", "can we partition" | Knapsack (0/1) | `dp[i][capacity]` |
| "**unlimited** copies of each item / coin" | Unbounded knapsack | 1D over capacity, coins outer |
| "**longest increasing / non-decreasing** subsequence" | LIS | 1D `dp[i]`, or O(n log n) with binary search |
| "**subsequence**" (not contiguous) vs "**substring**" (contiguous) | Subsequence DP vs [sliding window](05-sliding-window.md) | read carefully! |

> **Subsequence vs substring — the trap that trips everyone.** A **substring/subarray** is *contiguous* ("abcde" → "bcd"). A **subsequence** keeps order but *may skip* ("abcde" → "ace"). Most 2D string DP is about **subsequences**. If the problem is about a *contiguous* window, you probably want [05 — Sliding Window](05-sliding-window.md), not this chapter. Misreading this wastes interviews — reread the problem and confirm which one it is.

---

## 3. Picture It (Diagram)

Let's visualise the star of this chapter: **Longest Common Subsequence (LCS)** of `A = "abcde"` and `B = "ace"`. We build a grid where `dp[i][j]` = length of the LCS of the first `i` chars of `A` and first `j` chars of `B`. Row 0 and column 0 are the "empty string" base cases (LCS with nothing = 0).

```
          ""   a    c    e     ← B (j = 0..3)
      ┌────┬────┬────┬────┐
   "" │ 0  │ 0  │ 0  │ 0  │
      ├────┼────┼────┼────┤
   a  │ 0  │ 1  │ 1  │ 1  │   a==a → 1 + dp[↖]
      ├────┼────┼────┼────┤
   b  │ 0  │ 1  │ 1  │ 1  │   no match → max(dp[↑], dp[←])
      ├────┼────┼────┼────┤
   c  │ 0  │ 1  │ 2  │ 2  │   c==c → 1 + dp[↖]
      ├────┼────┼────┼────┤
   d  │ 0  │ 1  │ 2  │ 2  │
      ├────┼────┼────┼────┤
   e  │ 0  │ 1  │ 2  │ 3  │   e==e → 1 + dp[↖]  ← answer, bottom-right
      └────┴────┴────┴────┘
    (A, i = 0..5)
```

Read the two rules straight off the grid:

- **Characters match** (`A[i-1] === B[j-1]`) → walk **diagonally up-left** and add 1: `dp[i][j] = dp[i-1][j-1] + 1`.
- **No match** → take the better of **dropping the current char of A** (cell above) or **of B** (cell to the left): `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`.

The final answer sits in the **bottom-right corner**: `dp[5][3] = 3` (the LCS is `"ace"`). Notice the **fill order**: every cell needs the cells *above*, *left*, and *diagonal* — all of which are already filled if you go row-by-row, left-to-right. Getting that order right is half the battle in 2D DP.

---

## 4. The Template (TypeScript)

### Setting up a 2D table (and the #1 JS gotcha)

> ⚠️ **JS/TS gotcha — 2D array init (the classic reference-sharing bug).**
> ```typescript
> // ❌ WRONG — every "row" is the SAME array by reference
> const dp = new Array(n).fill(new Array(m).fill(0));
> dp[0][0] = 5;        // now dp[1][0], dp[2][0]… are ALL 5. Silent, brutal bug.
>
> // ✅ RIGHT — Array.from builds a fresh row each time
> const dp = Array.from({ length: n }, () => new Array<number>(m).fill(0));
> ```
> `fill(someArray)` copies the *reference*, so all rows alias one array. `Array.from({length:n}, () => …)` calls the factory `n` times, giving `n` independent rows. **This bug is the single most common way 2D DP quietly returns garbage.** Burn the `Array.from` pattern into muscle memory.

### Family 1 — Grid paths (Unique Paths, LeetCode 62)

> Robot at top-left of an `m × n` grid, moves only **right or down**, count paths to bottom-right.

State `dp[i][j]` = number of paths to reach cell `(i, j)`. You arrive from the top or the left, so `dp[i][j] = dp[i-1][j] + dp[i][j-1]`. First row and first column are all `1` (only one straight-line path).

```typescript
function uniquePaths(m: number, n: number): number {
  // Space-optimised to 1D: we only ever need the row above.
  const dp = new Array<number>(n).fill(1); // first row = all 1s
  for (let i = 1; i < m; i++) {
    for (let j = 1; j < n; j++) {
      // dp[j] (old) = value from row above; dp[j-1] = value from the left
      dp[j] += dp[j - 1];
    }
  }
  return dp[n - 1];
}
// uniquePaths(3, 7) === 28
```

**Min Path Sum (LeetCode 64)** is the same grid, but instead of *counting* you *minimise*: `dp[i][j] = grid[i][j] + Math.min(fromTop, fromLeft)`. Count → sum, `+` → `min`. Same table, different operator — that's the pattern.

### Family 2 — 0/1 Knapsack (the take/skip template in 2D)

> `n` items, item `i` has `weight[i]` and `value[i]`, bag capacity `W`. Each item used **at most once**. Maximise value.

State `dp[i][w]` = best value using the first `i` items with capacity `w`. At each item you **skip** (`dp[i-1][w]`) or **take** it if it fits (`value[i-1] + dp[i-1][w - weight[i-1]]`).

```typescript
function knapsack01(weights: number[], values: number[], W: number): number {
  const n = weights.length;
  const dp = Array.from({ length: n + 1 }, () => new Array<number>(W + 1).fill(0));
  for (let i = 1; i <= n; i++) {
    for (let w = 0; w <= W; w++) {
      dp[i][w] = dp[i - 1][w]; // skip item i
      if (weights[i - 1] <= w) {
        dp[i][w] = Math.max(
          dp[i][w],
          values[i - 1] + dp[i - 1][w - weights[i - 1]] // take item i
        );
      }
    }
  }
  return dp[n][W];
}
```

**1D space-optimised knapsack** — iterate capacity **backward** so each item is used at most once:

```typescript
function knapsack01_1D(weights: number[], values: number[], W: number): number {
  const dp = new Array<number>(W + 1).fill(0);
  for (let i = 0; i < weights.length; i++) {
    for (let w = W; w >= weights[i]; w--) { // ← BACKWARD is load-bearing
      dp[w] = Math.max(dp[w], values[i] + dp[w - weights[i]]);
    }
  }
  return dp[W];
}
```

> **Why backward?** Going forward would let you reuse item `i` within the same pass (that's *unbounded* knapsack — which is exactly what Coin Change II does, looping **forward**). 0/1 knapsack forbids reuse, so we iterate capacity high→low. The *direction of the inner loop* is the entire difference between "use once" and "use unlimited." Interviewers probe this.

### Family 3 — Subsequence / two-string DP (LCS, Edit Distance)

```typescript
function longestCommonSubsequence(a: string, b: string): number {
  const n = a.length, m = b.length;
  const dp = Array.from({ length: n + 1 }, () => new Array<number>(m + 1).fill(0));
  for (let i = 1; i <= n; i++) {
    for (let j = 1; j <= m; j++) {
      if (a[i - 1] === b[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1] + 1;          // match → extend diagonal
      } else {
        dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]); // skip one side
      }
    }
  }
  return dp[n][m];
}
```

**Edit Distance (LeetCode 72)** — same skeleton, three choices when chars differ (insert / delete / replace), plus non-zero base cases (`dp[i][0] = i` means "delete `i` chars"):

```typescript
function minDistance(word1: string, word2: string): number {
  const n = word1.length, m = word2.length;
  const dp = Array.from({ length: n + 1 }, () => new Array<number>(m + 1).fill(0));
  for (let i = 0; i <= n; i++) dp[i][0] = i;     // turn word1[0..i) into "" → i deletes
  for (let j = 0; j <= m; j++) dp[0][j] = j;     // "" → word2[0..j) → j inserts
  for (let i = 1; i <= n; i++) {
    for (let j = 1; j <= m; j++) {
      if (word1[i - 1] === word2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1];              // free, chars already match
      } else {
        dp[i][j] = 1 + Math.min(
          dp[i - 1][j],     // delete from word1
          dp[i][j - 1],     // insert into word1
          dp[i - 1][j - 1]  // replace
        );
      }
    }
  }
  return dp[n][m];
}
```

### Bonus — Longest Increasing Subsequence, two ways (LeetCode 300)

The O(n²) DP is pure [DP I](18-dynamic-programming-1.md); the O(n log n) version swaps the inner scan for a **binary search** — a lovely cross-over with [07 — Binary Search](07-binary-search.md).

```typescript
// O(n^2): dp[i] = length of the longest increasing subsequence ENDING at i.
function lengthOfLIS_n2(nums: number[]): number {
  if (nums.length === 0) return 0;
  const dp = new Array<number>(nums.length).fill(1);
  let best = 1;
  for (let i = 1; i < nums.length; i++) {
    for (let j = 0; j < i; j++) {
      if (nums[j] < nums[i]) dp[i] = Math.max(dp[i], dp[j] + 1);
    }
    best = Math.max(best, dp[i]);
  }
  return best; // NOTE: answer is max over dp[], not dp[n-1]
}

// O(n log n): "patience sorting". tails[k] = smallest possible tail of an
// increasing subsequence of length k+1. Binary-search where each number fits.
function lengthOfLIS_nlogn(nums: number[]): number {
  const tails: number[] = [];
  for (const x of nums) {
    let lo = 0, hi = tails.length;
    while (lo < hi) {                 // find first tail >= x (lower_bound)
      const mid = (lo + hi) >> 1;
      if (tails[mid] < x) lo = mid + 1;
      else hi = mid;
    }
    if (lo === tails.length) tails.push(x); // x extends the longest run
    else tails[lo] = x;                      // x replaces to keep tails small
  }
  return tails.length;
}
```

> `tails` is **not** the actual subsequence — its *length* is the answer. This confuses people; don't try to read the LIS out of `tails`. If you need the sequence itself, track predecessor indices separately.

---

## 5. Complexity

Same rule as [DP I](18-dynamic-programming-1.md): **time = number of states × work per state.** With two indices, states = the grid size.

| Problem | Time | Space (table) | Space-optimised |
|---|---|---|---|
| Unique Paths / Min Path Sum | O(m·n) | O(m·n) | **O(n)** (keep one row) |
| 0/1 Knapsack / Subset Sum | O(n·W) | O(n·W) | **O(W)** (1D, backward) |
| Partition Equal Subset Sum | O(n·sum) | O(sum) | already 1D |
| LCS / Edit Distance | O(n·m) | O(n·m) | O(min(n,m)) (two rows) |
| LIS (DP) | O(n²) | O(n) | — |
| LIS (patience + binary search) | **O(n log n)** | O(n) | — |

The recurring trick: if `dp[i]` only needs **row `i-1`**, you can collapse the 2D table to **one or two rows** and drop a whole factor of space. Great follow-up answer when the interviewer asks "can you reduce the memory?"

---

## 6. Worked Example (a classic LeetCode problem)

### Longest Common Subsequence (LeetCode 1143) — fully traced

> Given `text1` and `text2`, return the length of their longest common subsequence.

We traced the grid for `A = "abcde"`, `B = "ace"` in §3 — let's connect it to the recipe:

1. **State:** `dp[i][j]` = LCS length of `A[0..i)` and `B[0..j)`.
2. **Recurrence:** match → `dp[i-1][j-1] + 1`; else → `max(dp[i-1][j], dp[i][j-1])`.
3. **Base cases:** `dp[0][*] = dp[*][0] = 0` (LCS with an empty string is 0) — the top row and left column of the grid.
4. **Style:** bottom-up table (fill row by row so `↑`, `←`, `↖` are ready).
5. **Answer:** `dp[n][m]` — the bottom-right cell.

**Walking a few cells of the §3 grid:**

- `dp[1][1]`: `A[0]='a'`, `B[0]='a'` → **match** → `dp[0][0] + 1 = 1`.
- `dp[3][2]`: `A[2]='c'`, `B[1]='c'` → **match** → `dp[2][1] + 1 = 1 + 1 = 2`.
- `dp[2][2]`: `A[1]='b'`, `B[1]='c'` → **no match** → `max(dp[1][2]=1, dp[2][1]=1) = 1`.
- `dp[5][3]`: `A[4]='e'`, `B[2]='e'` → **match** → `dp[4][2] + 1 = 2 + 1 = 3`. ✅

Final answer **3** (`"ace"`). The `longestCommonSubsequence` function in §4 produces exactly this grid.

### Two more to cement the families

- **Unique Paths (62):** the *count* twin of grid DP — `dp[i][j] = dp[i-1][j] + dp[i][j-1]`, answer bottom-right. Try tracing a `3×3` grid by hand; you'll get `6`.
- **Coin Change (322):** you met the **min** version in [DP I §7](18-dynamic-programming-1.md); its cousin **Coin Change II (518)** *counts* combinations and is textbook **unbounded knapsack** — loop **coins outer, amount inner, forward** (so each coin can repeat). The loop order is the whole trick, and interviewers love to ask why.

---

## 7. Practice Set (curated LeetCode)

Grouped by family so you *feel* the shared skeleton. Do one from each family, then come back for the rest.

| # | Problem | LeetCode | Difficulty | Family |
|---|---|---|---|---|
| 1 | Unique Paths | 62 | Medium | Grid paths (count) |
| 2 | Minimum Path Sum | 64 | Medium | Grid paths (min) |
| 3 | 0/1 Knapsack | — (classic) | Medium | Knapsack (the template) |
| 4 | Partition Equal Subset Sum | 416 | Medium | Subset-sum knapsack (boolean) |
| 5 | Target Sum | 494 | Medium | Subset-sum in disguise (+/− → partition) |
| 6 | Coin Change | 322 | Medium | Unbounded knapsack (min) |
| 7 | Longest Common Subsequence | 1143 | Medium | Two-string DP |
| 8 | Edit Distance | 72 | Hard | Two-string DP (3 choices) |
| 9 | Longest Increasing Subsequence | 300 | Medium | Sequence DP + [binary search](07-binary-search.md) |

**Partition Equal Subset Sum (416)** — a 1D knapsack you should be able to write fast:

```typescript
function canPartition(nums: number[]): boolean {
  const total = nums.reduce((a, b) => a + b, 0);
  if (total % 2 !== 0) return false;       // odd total can't split evenly
  const target = total / 2;
  const dp = new Array<boolean>(target + 1).fill(false);
  dp[0] = true;                            // sum 0 is always reachable (empty subset)
  for (const num of nums) {
    for (let s = target; s >= num; s--) {  // backward → each number used once (0/1)
      dp[s] = dp[s] || dp[s - num];
    }
  }
  return dp[target];
}
// canPartition([1,5,11,5]) === true   → {1,5,5} and {11}
```

> **Target Sum (494) insight:** assigning `+`/`−` to reach `S` is algebraically the same as choosing a subset `P` (the `+` group) with `sum(P) = (total + S) / 2`. So it *reduces to subset-sum* — recognising this reduction is the interview win. Sorting sometimes helps preprocessing here; see [08 — Sorting](08-sorting.md).

---

## 8. Common Pitfalls & Interview Tips

| Pitfall | What goes wrong | Fix |
|---|---|---|
| **2D array reference sharing** | `new Array(n).fill(new Array(m).fill(0))` aliases every row; writes leak across rows. | `Array.from({length:n}, () => new Array(m).fill(0))`. Always. |
| **Off-by-one table size** | Two-string DP needs `(n+1) × (m+1)` to hold the empty-string row/col; sizing `n × m` corrupts base cases. | Size to `+1` and index the string with `s[i-1]`. |
| **Wrong fill order** | Filling before the cells you depend on (`↑`, `←`, `↖`) are ready. | Row-by-row, left-to-right for standard grids; know *which* neighbours each cell needs. |
| **Subsequence vs substring** | Solving a contiguous-window problem with subsequence DP (or vice-versa). | Reread: "subsequence" (may skip) = this chapter; "substring/subarray" (contiguous) = [sliding window](05-sliding-window.md). |
| **Inner-loop direction in knapsack** | Forward loop turns 0/1 into unbounded (reuses items). | 0/1 = **backward** capacity; unbounded/coins = **forward**. |
| **Answer in the wrong cell** | LIS answer is `max(dp)`, not `dp[n-1]`; LCS answer is `dp[n][m]`. | Decide the answer location explicitly (recipe step 5). |
| **Forgetting base rows aren't all zero** | Edit Distance base row is `0,1,2,3…`, not zeros. | Initialise base row/column to match the problem's meaning. |

**Interview scripts:**
- **Draw the grid on the whiteboard first.** Label axes with the two strings/dimensions, fill row 0 and column 0, then state the recurrence for a generic `dp[i][j]`. Interviewers grade this setup heavily — a correct table structure is most of the credit.
- **Say the recurrence in words:** "If the characters match I extend the diagonal; otherwise I drop one character and take the better side." Clear narration beats silent coding.
- **Offer the space optimisation** as your closing move: "This is O(n·m) space, but each row only needs the previous row, so I can bring it down to O(min(n, m))."
- For **LIS**, mention both: "O(n²) DP is straightforward; if they want faster, patience sorting with binary search is O(n log n)" — signals range and links to [07 — Binary Search](07-binary-search.md).

> **Real-project analogy (santhu.site).** A "diff" view — highlighting what changed between two versions of your resume or a doc on [santhu.site](https://santhu.site) — is powered by exactly this two-string DP. Git's diff, VS Code's inline compare, `react-diff-viewer`: all descendants of LCS / edit-distance. You've been *using* 2D DP in your tools this whole time.

---

## 9. TL;DR

- **2D DP = DP I with a two-coordinate state.** When *two* things vary (row & column, item & capacity, index in A & index in B), your sticky note becomes `dp[i][j]`.
- **Three families cover almost everything:** **grid paths** (come from top/left), **knapsack** (take/skip an item under a capacity), **subsequence/two-string** (match → extend diagonal; else drop one side).
- **The must-know JS gotcha:** build 2D tables with `Array.from({length:n}, () => new Array(m).fill(0))` — *never* `new Array(n).fill(new Array(m))`, which shares one row across all rows.
- **Loop direction is semantic:** 0/1 knapsack iterates capacity **backward** (use once); unbounded/coin-count iterates **forward** (reuse allowed).
- **Get the small stuff right:** size two-string tables `(n+1)×(m+1)`, initialise base rows to their real meaning (Edit Distance ≠ zeros), fill in an order where `↑`/`←`/`↖` are ready, and locate the answer consciously (LCS → `dp[n][m]`; LIS → `max(dp)`).
- **Complexity = states × work:** grids O(m·n), knapsack O(n·W), two-string O(n·m); LIS is O(n²) or **O(n log n)** with [binary search](07-binary-search.md). Collapse to one/two rows to cut space.
- **subsequence ≠ substring** — reread the problem; contiguous windows belong to [sliding window](05-sliding-window.md), not here.

**Where to go next:** you've now got the full DP toolkit — 1D ([18 — DP I](18-dynamic-programming-1.md)) and 2D (here). Next up, [20 — Greedy & Intervals](20-greedy-and-intervals.md) shows you the *other* way to make sequential choices — committing to the locally-best option — and, crucially, **how to tell when greedy is safe versus when you're forced back to DP.**
