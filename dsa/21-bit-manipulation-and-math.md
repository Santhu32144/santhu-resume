# 21. Bit Manipulation & Math

> **One-liner:** Stop thinking of a number as a value and start thinking of it as a **row of on/off switches** — then a handful of `&`, `|`, `^`, and shift tricks solve in `O(1)` space what loops and hash maps do with extra memory.
> **Pattern power:** This is the **smallest** pattern in the track by interview frequency — but the ~7 classics here (Single Number, Counting Bits, Missing Number, power-of-two, subsets-via-mask) recur often enough that missing one is a needless -1. Learn the toolkit, bank the easy wins, and don't over-invest.

---

## 1. The Idea in Plain Words

Every whole number is stored in the computer as **binary** — a string of `0`s and `1`s, where each position is a **power of 2**. Reading right to left, the positions are worth `1, 2, 4, 8, 16, …`. A bit is `1` if that power is "included" in the number and `0` if not.

```
 decimal 13
 place   ...  16    8    4    2    1
 bit     ...   0    1    1    0    1     →   8 + 4 + 1  =  13
```

So `13` is `1101` in binary. That's the entire mental model: **a number is a set of switches, one per power of two.** Bit manipulation is just flipping, reading, and combining those switches directly instead of doing arithmetic.

Why bother? Because a few operations that would cost you a loop or a `Set` become a **single CPU instruction** with **zero extra memory**. That `O(1)`-space property is the whole selling point.

### The JS gotcha you MUST know: bitwise ops are 32-bit *signed*

This is the one thing that trips up JS/TS candidates. JavaScript numbers are 64-bit floats — but the moment you use a **bitwise operator** (`&`, `|`, `^`, `~`, `<<`, `>>`), JS **converts the number to a 32-bit signed integer**, does the operation, then converts back. Consequences:

- The **top (32nd) bit is the sign bit**. So `1 << 31` is **negative** (`-2147483648`), not `2147483648`.
- `~0 === -1`, because flipping all 32 bits of `0` gives all-ones, which in two's-complement is `-1`. In general `~x === -(x + 1)`.
- Anything above `2^31 - 1` (`2147483647`) **wraps around** into negatives.
- The shift amount is taken **mod 32**, so `1 << 32 === 1` (not `4294967296`) — a silent bug.
- There is one escape hatch: `>>>`, the **unsigned** right shift, is the *only* operator that returns an unsigned 32-bit result (range `0 … 2^32 - 1`). You'll reach for it constantly.

> Contrast this with plain arithmetic (`+`, `*`), which uses the full 64-bit float and is safe up to `Number.MAX_SAFE_INTEGER` (`2^53 - 1`). The 32-bit trap **only** applies to the bitwise operators. When you need big-number math, use `BigInt` (below), which sidesteps the whole issue.

### When bit tricks actually appear in interviews

Honestly? **Not that often.** At Zepto/Swiggy/CRED/Razorpay/Groww you'll see arrays, hashing, sliding window, trees, and DP far more. But a small, predictable set of bit problems recurs — and they're usually *Easy*, so they're free points if you know the trick and a cold sweat if you don't. Know the toolkit in section 4, do the ~9 practice problems, and stop. Don't grind 50 bit puzzles.

### Math you actually need

The "& Math" half of this chapter is four small tools that show up inside *other* problems (a GCD step in a fraction problem, a mod in a DP counting problem). One line each — learn the idea, not a derivation.

**GCD — Euclid's algorithm.** `gcd(a, b) = gcd(b, a mod b)` until `b` hits `0`. `O(log min(a, b))`.

```ts
const gcd = (a: number, b: number): number => (b === 0 ? a : gcd(b, a % b));
const lcm = (a: number, b: number): number => (a / gcd(a, b)) * b; // least common multiple
```

**Sieve of Eratosthenes — all primes up to `n`.** Walk from 2; whenever you meet a prime, cross out *all its multiples* as composite. `O(n log log n)` — near-linear.

```ts
function primesUpTo(n: number): number[] {
  const isPrime = new Array<boolean>(n + 1).fill(true);
  isPrime[0] = isPrime[1] = false;
  for (let i = 2; i * i <= n; i++) {
    if (isPrime[i]) {
      for (let j = i * i; j <= n; j += i) isPrime[j] = false; // start at i*i, smaller multiples already marked
    }
  }
  return isPrime.flatMap((prime, value) => (prime ? [value] : []));
}
```

**Modulo arithmetic & "answer mod 1e9+7".** When a problem counts something (paths, ways, arrangements) the answer can explode past `2^53`, so problems ask for it **modulo `1_000_000_007`** — a large prime that keeps every intermediate value small. The rules: `(a + b) % m` and `(a * b) % m` distribute, so you apply `% m` **after every step**, never at the end only. The JS trap: two numbers near `1e9` multiply to ~`1e18`, which is **past `Number.MAX_SAFE_INTEGER`** — silent precision loss. Use `BigInt` for the multiply:

```ts
const MOD = 1_000_000_007n;
const mulMod = (a: bigint, b: bigint): bigint => (a * b) % MOD; // BigInt dodges the 2^53 precision cliff
```

**Fast (binary) exponentiation.** `base^exp` by squaring: read `exp`'s bits, square the base each step, multiply it in on `1` bits. `O(log exp)` instead of `O(exp)`.

```ts
function modPow(base: bigint, exp: bigint, mod: bigint): bigint {
  let result = 1n;
  base %= mod;
  while (exp > 0n) {
    if (exp & 1n) result = (result * base) % mod; // odd bit → fold current base into result
    base = (base * base) % mod;                   // square the base
    exp >>= 1n;                                    // shift to the next bit
  }
  return result;
}
```

> Note `BigInt` supports the same bit operators (`&`, `>>`, `<<`) — `exp & 1n`, `exp >>= 1n` — but with **arbitrary precision and no 32-bit wrap**. For math-flavoured problems, BigInt is the safe default.

---

## 2. When to Use It — Pattern Triggers

Scan the problem for these signals:

- **"Find the single / unique number"** — every element appears twice (or three times) except one. → **XOR** everything; pairs cancel. *"Find the element that appears once."*
- **"Count the bits / set bits / 1s"** — Hamming weight, or bit counts for a whole range `0…n`. → **Brian Kernighan** or the `dp[i] = dp[i >> 1] + (i & 1)` recurrence.
- **"Is it a power of two / four?"** — a power of two has **exactly one** set bit. → `x > 0 && (x & (x - 1)) === 0`.
- **"Generate all subsets / combinations"** with `n ≤ ~20` — → **bitmask**: each integer `0 … 2^n - 1` *is* a subset (bit `j` set = include element `j`). An alternative to backtracking → [11-recursion-and-backtracking.md](11-recursion-and-backtracking.md).
- **"Do it with no extra space"** / **"O(1) memory"** on a find-the-odd-one-out or missing-number problem → **XOR** replaces the hash set.
- **"Add / multiply without using `+` or `*`"** — → simulate with XOR (sum) + AND-shift (carry).
- **"Toggle / set / check a flag"**, bit flags, permission masks → the check/set/clear/toggle toolkit.

> If the problem is really "have I seen this before?" or "how many of each?", a **hash map / set** ([03-hashing-maps-and-sets.md](03-hashing-maps-and-sets.md)) is clearer and you should say so — reach for bits only when the *uniqueness structure* (exactly-twice, exactly-once) or the *space constraint* makes XOR the elegant move.

---

## 3. Picture It (Diagram)

**A number is its bits.** `13 = 1101`:

```
 value:    13
 binary:    1    1    0    1
 weight:    8    4    2    1      →  8 + 4 + 1 = 13
```

**XOR (`^`)** — output bit is `1` only where the two input bits **differ** (like a "spot the difference"):

```
   5  =  0 1 0 1
   3  =  0 0 1 1
 ------------------  XOR   (1 where bits DIFFER)
   6  =  0 1 1 0
```

**AND (`&`)** — output bit is `1` only where **both** are `1` (like an intersection / a mask):

```
   5  =  0 1 0 1
   3  =  0 0 1 1
 ------------------  AND   (1 only where BOTH are 1)
   1  =  0 0 0 1
```

The XOR picture is the key to this chapter: notice that `5 ^ 5` would be all-zeros (`0000`) because a number never differs from itself. Hold that thought — it powers the worked example.

---

## 4. The Template (TypeScript)

### The five operators

```ts
5 & 3;    // AND          → 1    bit set only where BOTH are 1
5 | 3;    // OR           → 7    bit set where EITHER is 1
5 ^ 3;    // XOR          → 6    bit set where the bits DIFFER
~5;       // NOT          → -6   flips every bit;  ~x === -(x + 1)
1 << 4;   // left shift   → 16   x * 2^4  (adds zeros on the right)
32 >> 2;  // right shift  → 8    Math.floor(x / 2^2), copies the SIGN bit in
-8 >> 1;  // arithmetic   → -4   sign-preserving
-8 >>> 1; // UNSIGNED     → 2147483644   zero-filled; the only unsigned-result op
```

### The bit toolkit — check / set / clear / toggle the i-th bit

`i` is 0-indexed from the **least-significant (rightmost)** bit.

```ts
const getBit    = (x: number, i: number): number => (x >> i) & 1;  // read i-th bit → 0 or 1
const setBit    = (x: number, i: number): number => x | (1 << i);  // force i-th bit to 1
const clearBit  = (x: number, i: number): number => x & ~(1 << i); // force i-th bit to 0
const toggleBit = (x: number, i: number): number => x ^ (1 << i);  // flip i-th bit
```

### The two "lowest set bit" one-liners

```ts
const lowestSetBit  = (x: number): number => x & -x;      // ISOLATE the lowest 1-bit:  12 (1100) → 4 (100)
const dropLowestBit = (x: number): number => x & (x - 1); // REMOVE  the lowest 1-bit:  12 (1100) → 8 (1000)
```

`x & -x` works because `-x` is two's-complement (flip-and-add-one), which lines up a single shared `1` exactly at the lowest set bit. `x & (x - 1)` works because subtracting 1 flips the lowest `1` and everything below it, so ANDing clears just that bit. Both are `O(1)`.

### Count set bits (Brian Kernighan)

```ts
function countSetBits(x: number): number {
  let count = 0;
  while (x !== 0) {
    x &= x - 1; // drop the lowest set bit each pass → loops ONCE PER SET BIT, not 32 times
    count++;
  }
  return count;
}
// For LeetCode "Number of 1 Bits", if bit 31 might be set, start with:  x = x >>> 0  (read as unsigned).
```

### The XOR identities that power half of all bit puzzles

```ts
// a ^ a === 0     anything XOR itself cancels to 0
// a ^ 0 === a     anything XOR 0 is unchanged
// XOR is commutative AND associative → order doesn't matter.
// ⇒ XOR a whole list where every value appears twice, and only the loner survives.
```

---

## 5. Complexity

| Operation | Time | Space | Note |
|---|---|---|---|
| Any single `&` `|` `^` `~` `<<` `>>` | `O(1)` | `O(1)` | one machine instruction |
| Check/set/clear/toggle a bit | `O(1)` | `O(1)` | mask + one op |
| Count set bits (Brian Kernighan) | `O(k)` | `O(1)` | `k` = number of set bits (≤ 32) → effectively `O(1)` |
| Single Number via XOR over `n` items | `O(n)` | `O(1)` | beats the `O(n)` **space** of a hash set |
| Counting Bits for `0…n` (DP) | `O(n)` | `O(n)` | one pass with the `>> 1` recurrence |
| Subsets via bitmask, `n` elements | `O(n · 2^n)` | `O(1)` extra | `2^n` masks × `O(n)` to read each |
| GCD (Euclid) | `O(log min(a,b))` | `O(1)` | |
| Sieve of Eratosthenes up to `n` | `O(n log log n)` | `O(n)` | |
| Fast exponentiation | `O(log exp)` | `O(1)` | |

The headline: bit ops are constant-time, and XOR's superpower is turning `O(n)`-space "seen it?" problems into `O(1)` space.

---

## 6. Worked Example — Single Number (LeetCode 136)

> **Problem:** Every element in `nums` appears **exactly twice**, except for **one** element that appears once. Find that single one — in **`O(n)` time and `O(1)` extra space**.

### Brute force (the baseline you say out loud)

A hash map counting occurrences, then return the key with count 1 — `O(n)` time but `O(n)` **space**. Correct, but the problem *explicitly* asks for `O(1)` space, which is a giant hint that a hash set is *not* the intended answer.

### Key insight — the `a ^ a = 0` cancellation

Recall the two identities: `a ^ a = 0` and `a ^ 0 = a`, and that XOR doesn't care about order. So if you XOR **every** number together, every value that appears **twice cancels itself to 0**, and the lone value gets XOR-ed with `0`, leaving it untouched. What survives is exactly the answer.

```ts
function singleNumber(nums: number[]): number {
  let result = 0;
  for (const n of nums) {
    result ^= n; // pairs cancel to 0; the loner survives
  }
  return result;
}
```

### Trace — `nums = [4, 1, 2, 1, 2]`

| Step | `n` | `result` before | `result` after (`result ^ n`) |
|---|---|---|---|
| 1 | 4 | `0000` | `0100` (4) |
| 2 | 1 | `0100` | `0101` (5) |
| 3 | 2 | `0101` | `0111` (7) |
| 4 | 1 | `0111` | `0110` (6)  ← the first `1` cancels |
| 5 | 2 | `0110` | `0100` (4)  ← the first `2` cancels |

Result `= 4`, the single number. No map, no sort — one linear pass, `O(1)` space. That's the whole trick.

### Three more classics to recognise on sight

**Number of 1 Bits (LeetCode 191)** — count set bits with Brian Kernighan (section 4). One line per set bit.

**Counting Bits (LeetCode 338)** — return the bit count for *every* number `0…n`. The DP: a number `i` has the same bits as `i >> 1` (itself shifted right, dropping the last bit) plus whatever the last bit is (`i & 1`):

```ts
function countBits(n: number): number[] {
  const dp = new Array<number>(n + 1).fill(0);
  for (let i = 1; i <= n; i++) {
    dp[i] = dp[i >> 1] + (i & 1); // reuse an already-computed smaller answer → O(n) total
  }
  return dp;
}
```

**Missing Number (LeetCode 268)** — an array holds `n` distinct numbers from the range `0…n` with exactly one missing. XOR **all the indices `0…n`** and **all the values**; every present number cancels with its index-twin, leaving the one index that never had a value:

```ts
function missingNumber(nums: number[]): number {
  let x = nums.length;              // seed with n (the largest index)
  for (let i = 0; i < nums.length; i++) {
    x ^= i ^ nums[i];               // XOR in each index and each value
  }
  return x;                         // the index that never cancelled
}
```

---

## 7. Practice Set (curated LeetCode)

Do them in order. The first six are Easy and are the ones that actually recur — bank them.

| # | Name | Difficulty | Trigger it teaches |
|---|---|---|---|
| 136 | Single Number | Easy | XOR cancellation (`a^a=0`) |
| 268 | Missing Number | Easy | XOR indices against values |
| 191 | Number of 1 Bits | Easy | Brian Kernighan `x & (x-1)` |
| 338 | Counting Bits | Easy | `dp[i] = dp[i>>1] + (i&1)` |
| 231 | Power of Two | Easy | single set bit: `x & (x-1) === 0` |
| 190 | Reverse Bits | Easy | shift-in loop + `>>>` for unsigned |
| 137 | Single Number II | Medium | count bits mod 3 (appears 3×) |
| 260 | Single Number III | Medium | XOR-all, split by `xor & -xor` |
| 371 | Sum of Two Integers | Medium | XOR = sum, `(a&b)<<1` = carry |
| 78 | Subsets | Medium | bitmask enumeration of `2^n` sets |

**Single Number III** and **Sum of Two Integers** are the two worth studying deeply — they show the "isolate a differing bit" and "simulate a carry" ideas:

```ts
// 260 — two elements appear once, the rest twice. XOR-all gives a^b; isolate a bit where they differ.
function singleNumberIII(nums: number[]): [number, number] {
  let xorAll = 0;
  for (const n of nums) xorAll ^= n;   // = a ^ b (the two loners)
  const diff = xorAll & -xorAll;       // lowest bit where a and b differ
  let a = 0;
  for (const n of nums) if (n & diff) a ^= n; // partition: only the "diff-bit set" group
  return [a, xorAll ^ a];              // b = (a^b) ^ a
}

// 371 — add without '+'. XOR is the sum WITHOUT carry; (a & b) << 1 is the carry. Repeat until no carry.
function getSum(a: number, b: number): number {
  while (b !== 0) {
    const carry = (a & b) << 1; // where both are 1, a carry propagates left
    a = a ^ b;                  // add the bits that don't carry
    b = carry;                  // fold the carry back in next round
  }
  return a; // works precisely because JS bit ops are 32-bit — the carry chain terminates in ≤32 steps
}
```

---

## 8. Common Pitfalls & Interview Tips

- **JS bitwise ops are 32-bit *signed* — this is THE trap.** `1 << 31` is **negative**. `~0 === -1`. Values above `2^31 - 1` wrap into negatives. And because the shift count is taken mod 32, `1 << 32 === 1` (not `4294967296`). Say out loud: *"JS coerces to Int32 for bit ops, so I'll watch the sign bit and use `>>>` when I need unsigned."*

- **`>>>` is the unsigned right shift — reach for it deliberately.** It's the *only* operator that returns an unsigned 32-bit result. Use it for **Reverse Bits** (you want the bits, not a sign), for counting bits when bit 31 may be set (`x >>> 0` first), and generally to coerce a number back into `0 … 2^32 - 1`.

- **Operator precedence WILL bite you — parenthesise.** The bitwise operators `& ^ |` bind **looser** than `===`, `==`, and `+`. So `x & 1 === 0` parses as `x & (1 === 0)` → `x & false` → wrong answer, no error. Always write `(x & 1) === 0`. Same for `(a + b) << 1` when you mean to shift the sum. When unsure, over-parenthesise — reviewers never penalise clarity here.

- **`sort()` is still lexicographic** if a bit problem ever needs sorting — pass `(a, b) => a - b`. (Carried over from [08-sorting.md](08-sorting.md).)

- **Big-number math → use `BigInt`, not bit ops.** The 32-bit ceiling means bitwise tricks silently corrupt large values. Products near `1e9 * 1e9` also blow past `Number.MAX_SAFE_INTEGER` (`2^53 - 1`). For "answer mod 1e9+7" and fast exponentiation, use `BigInt` (`1_000_000_007n`) and apply `% MOD` after every multiply — this shows up in DP counting problems ([18-dynamic-programming-1.md](18-dynamic-programming-1.md), [19-dynamic-programming-2.md](19-dynamic-programming-2.md)).

- **Don't over-index on bit tricks.** This is the lowest-yield chapter for interview *frequency*. Know the ~7 classics cold, be able to explain XOR cancellation and `x & (x-1)`, and move your remaining hours to arrays, hashing, trees, graphs, and DP. If an interviewer *does* give you a bit problem, it's almost always one of the practice-set items above.

- **What to say out loud (scores communication points):** *"Every element appears twice except one, and I need O(1) space — that's the XOR cancellation pattern. XOR-ing the whole array makes pairs vanish since `a ^ a = 0`, leaving the unique value. O(n) time, O(1) space."*

---

## 9. TL;DR

- **A number is a row of bits (powers of two).** Bit manipulation flips/reads/combines switches directly: `O(1)` time, `O(1)` space.
- **Toolkit:** `getBit (x>>i)&1`, `setBit x|(1<<i)`, `clearBit x&~(1<<i)`, `toggleBit x^(1<<i)`, isolate-lowest `x & -x`, drop-lowest `x & (x-1)`, count-bits via Brian Kernighan.
- **XOR is the star:** `a^a=0`, `a^0=a`, order-independent ⇒ Single Number, Missing Number, Single Number III — all `O(1)` space, no hash set.
- **JS trap:** bit ops are **32-bit signed**; `1<<31` is negative, `1<<32===1`, `~0===-1`. Use `>>>` for unsigned; parenthesise because `& ^ |` bind looser than `===`/`+`; use **`BigInt`** for big-number / mod-1e9+7 math.
- **Math you need:** GCD via Euclid (`gcd(a,b)=gcd(b,a%b)`), Sieve of Eratosthenes for primes, modulo `1e9+7` (apply after every step, BigInt the multiply), fast exponentiation (`O(log exp)`).
- **Don't over-invest** — this is the lowest-frequency pattern. Bank the ~7 Easy classics and spend the rest of your time on the [150-problem plan](22-the-150-problem-plan.md).
