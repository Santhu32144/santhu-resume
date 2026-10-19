# 15. Tries (Prefix Trees)

> **One-liner:** A trie is a **tree of characters** where every root-to-node path spells a prefix — so `insert`, `search`, and `startsWith` all cost **O(L)** (length of the word), *independent of how many words you've stored*.
> **Pattern power:** The instant a problem is about **prefixes** — autocomplete, "words that start with…", a dictionary you query with wildcards, or DFS over a grid pruned by a word list — a trie is the structure that makes it clean and fast. A hash set can confirm an *exact* word, but only a trie can answer *"what can this prefix become?"* efficiently.

---

## 1. The Idea in Plain Words

Picture the index tabs of a paper dictionary: everything under **"CA"** shares that opening, and to reach "cat" you flip C → A → T. A **trie** (pronounced "try," from re**trie**val; also called a **prefix tree**) is exactly that idea as a data structure: a tree where **each edge is one character**, and following a path from the root **spells out a prefix**. Reach the end of a word and you flip a flag that says *"a real word ends here."*

Two words that share a prefix **share the same nodes** for that prefix. "cat", "car", and "card" all travel the same `c → a` before branching. That sharing is the whole point — it's what makes prefix questions cheap.

Each node holds just two things:

- **`children`** — a map from *next character* → *child node*.
- **`isEndOfWord`** — a boolean: does a complete word terminate right here?

That second field is subtle and easy to forget. Without it, a trie can only tell you *"some word passes through here"*, not *"a word ends here."* Insert `"apple"` and, without the flag, you'd wrongly think `"app"` is also a stored word — it's just a prefix, not a real entry. The flag is what separates *a stored word* from *a passer-through prefix*.

**Why not just a hash set?** A `Set<string>` answers *"is `"cat"` present?"* in `O(L)` too — same as a trie. The difference is **prefix power**:

- *"How many stored words start with `"ca"`?"* or *"give me all completions of `"ca"`"* — a `Set` must **scan every key** (`O(N·L)`). A trie **walks to the `"ca"` node in `O(L)`**, and *everything beneath that node is the answer*. That's autocomplete, and it's the trie's reason to exist.

So: reach for a trie when the questions are about **shared beginnings**, not just exact membership.

---

## 2. When to Use It — Pattern Triggers

| If the problem says… | Why it's a trie |
|---|---|
| "**prefix**", "starts with", "common prefix" | the defining operation — `O(L)` prefix walk |
| "**autocomplete**", "search **suggestions**", "type-ahead" | walk to the prefix node, DFS-collect what's below |
| "**dictionary** of words" + many lookups | build once, query each word in `O(L)` |
| "search word with **`.`/wildcard**" | at `.`, DFS into *all* children (see §6) |
| "**word search** on a board / grid" | trie of the word list prunes dead DFS paths early |
| "**replace** words by their **root/prefix**" | walk each word until you hit a stored root |
| "**longest word** buildable one letter at a time" | valid only if every prefix is also a word (flag on each step) |

**Anti-triggers:** if you only ever check **exact membership**, need **counting/grouping**, or there's **no prefix/character structure**, a plain hash set/map is simpler and lighter — use [03-hashing-maps-and-sets.md](03-hashing-maps-and-sets.md). Don't reach for a trie just because the input is strings.

> **Frequency note:** tries are **moderate-frequency** in interviews — you won't see them every loop. But when a problem *does* fit (anything with "prefix" or "autocomplete"), reaching for a trie is a **strong signal** that you know your data structures. LC208 "Implement Trie" is a common warm-up specifically to check you can build one.

---

## 3. Picture It (Diagram)

Insert `"cat"`, `"car"`, `"card"`, `"dog"`. A `*` marks `isEndOfWord = true`:

```
                (root)
               /      \
             c          d
             |          |
             a          o
            / \         |
          t*   r*       g*          t* = "cat"   r* = "car"   g* = "dog"
               |
               d*                    d* = "card"

Shared paths (the magic):
  • "cat", "car", "card" all reuse the  c → a  nodes.
  • "car" and "card" reuse  c → a → r ;  "card" just extends one more edge.

Reading the flags:
  • node r is marked *  → "car" is a stored word,
    AND it keeps going to d* → "card" is ALSO stored.
  • The node at c → a is NOT marked → "ca" is only a prefix, not a word.
```

Answering queries by walking the tree:

```
search("car")      : root → c → a → r ,  r.isEndOfWord? yes  → TRUE
search("ca")       : root → c → a ,      a.isEndOfWord? no   → FALSE  (prefix only)
startsWith("ca")   : root → c → a  reached at all?   yes     → TRUE
startsWith("cak")  : root → c → a → (no 'k' child)          → FALSE
```

Same three-step motion every time — *walk the characters* — which is why cost depends on **word length**, never on how many words are in the trie.

---

## 4. The Template (TypeScript)

A clean, correct trie. Note `children` is a **`Map`** (why, in §8) and every node carries the **end-of-word flag**.

```ts
class TrieNode {
  children: Map<string, TrieNode> = new Map();
  isEndOfWord = false;
}

class Trie {
  private root = new TrieNode();

  /** Add a word. O(L). Creates nodes for any new characters along the path. */
  insert(word: string): void {
    let node = this.root;
    for (const ch of word) {
      if (!node.children.has(ch)) node.children.set(ch, new TrieNode());
      node = node.children.get(ch)!;
    }
    node.isEndOfWord = true;              // mark the end — do NOT forget this
  }

  /** Is `word` a stored word (not merely a prefix)? O(L). */
  search(word: string): boolean {
    const node = this.walk(word);
    return node !== null && node.isEndOfWord;   // must land on a marked node
  }

  /** Does ANY stored word start with `prefix`? O(L). */
  startsWith(prefix: string): boolean {
    return this.walk(prefix) !== null;          // reaching the node is enough
  }

  /** Follow characters from the root; return the node reached, or null. */
  private walk(str: string): TrieNode | null {
    let node = this.root;
    for (const ch of str) {
      const next = node.children.get(ch);
      if (!next) return null;
      node = next;
    }
    return node;
  }
}
```

The **only** difference between `search` and `startsWith` is the last line: `search` demands `isEndOfWord`, `startsWith` is happy just to arrive. Everything else — the walk — is shared.

**Bonus: autocomplete.** Add this method to the `Trie` class above. Walk to the prefix node, then DFS everything beneath it — this is the real-world payoff (e.g. the search box on **[Habex](https://habex.santhu.site)** suggesting habit names as you type, without scanning every stored habit):

```ts
  /** Up to `limit` stored words that start with `prefix`. */
  autocomplete(prefix: string, limit = 10): string[] {
    const start = this.walk(prefix);              // reuse the private walk() above
    if (!start) return [];                        // no word has this prefix
    const out: string[] = [];
    const dfs = (node: TrieNode, suffix: string): void => {
      if (out.length >= limit) return;
      if (node.isEndOfWord) out.push(prefix + suffix);
      for (const [ch, child] of node.children) dfs(child, suffix + ch);
    };
    dfs(start, "");
    return out;                                   // up to `limit` completions of `prefix`
  }
```

> The DFS here is exactly the recursion/backtracking idea from [11-recursion-and-backtracking.md](11-recursion-and-backtracking.md), and "explore all children of a node" is the same tree/graph traversal shape as [16-graphs-bfs-dfs.md](16-graphs-bfs-dfs.md) — a trie is, after all, a tree.

---

## 5. Complexity

Let **L** = length of the word/prefix, **N** = number of words, **Σ** = alphabet size (26 for lowercase English).

| Operation | Time | Space | Note |
|---|---|---|---|
| `insert` | **O(L)** | up to O(L) new nodes | independent of N |
| `search` | **O(L)** | O(1) | must end on a marked node |
| `startsWith` | **O(L)** | O(1) | just reach the node |
| `autocomplete(prefix)` | O(L + size of subtree) | O(output) | walk, then collect below |
| Build from N words | O(N·L) | **O(total chars)** worst case | shared prefixes save memory |

**The headline:** every core operation is **O(L)** — it does **not** depend on how many words you've stored. A dictionary with a million words searches a 5-letter word in 5 steps. Compare that to a hash set which is also `O(L)` for *exact* lookup but **can't do prefix enumeration** without scanning all `N` keys.

**The trade-off is memory.** Every distinct character on every non-shared path is a node, and each node carries a map — that's a real constant-factor cost. A trie *buys* prefix speed by *spending* space. (Production systems use compressed/radix tries to shrink this; that's beyond interview scope, but it's a good sentence to drop.)

---

## 6. Worked Example (a classic LeetCode problem)

### Implement Trie (Prefix Tree) — [LeetCode 208](https://leetcode.com/problems/implement-trie-prefix-tree/)

> Implement `insert`, `search`, and `startsWith`. That's literally the class in §4 — this problem exists to check you can build one from scratch.

**Full trace** of the standard LC208 call sequence, watching the tree grow:

| Call | What happens | Trie after | Returns |
|---|---|---|---|
| `insert("apple")` | create nodes `a·p·p·l·e`, mark `e` | `a→p→p→l→e*` | — |
| `search("apple")` | walk `a·p·p·l·e`, land on `e*` | unchanged | **true** |
| `search("app")` | walk `a·p·p`, land on 2nd `p` — **not** marked | unchanged | **false** |
| `startsWith("app")` | walk `a·p·p` — node exists | unchanged | **true** |
| `insert("app")` | path `a·p·p` already exists; just mark 2nd `p` | `a→p→p*→l→e*` | — |
| `search("app")` | walk `a·p·p`, now `p` **is** marked | unchanged | **true** |

The pivotal rows are `search("app")` before and after `insert("app")`: the *nodes never changed* — only the **`isEndOfWord` flag** flipped. That single boolean is the entire difference between "a prefix" and "a word." Forget it and both those searches return the same wrong answer.

### Also know: Design Add and Search Words (wildcards) — [LeetCode 211](https://leetcode.com/problems/design-add-and-search-words-data-structure/)

Now `search` may contain `.`, which matches **any** single character. At a `.` you can't follow one edge — you must **try every child**. That turns the linear walk into a **DFS with backtracking** ([11-recursion-and-backtracking.md](11-recursion-and-backtracking.md)):

```ts
class WordDictionary {
  private root = new TrieNode();

  addWord(word: string): void {                 // identical to Trie.insert
    let node = this.root;
    for (const ch of word) {
      if (!node.children.has(ch)) node.children.set(ch, new TrieNode());
      node = node.children.get(ch)!;
    }
    node.isEndOfWord = true;
  }

  search(word: string): boolean {
    const dfs = (node: TrieNode, i: number): boolean => {
      if (i === word.length) return node.isEndOfWord;   // consumed the whole pattern
      const ch = word[i];
      if (ch === ".") {                                 // wildcard: try ALL children
        for (const child of node.children.values()) {
          if (dfs(child, i + 1)) return true;
        }
        return false;
      }
      const next = node.children.get(ch);               // normal char: one path
      return next ? dfs(next, i + 1) : false;
    };
    return dfs(this.root, 0);
  }
}
```

A concrete character costs one step; a `.` branches into every child and recurses — the trie makes that branching natural because the children are right there.

### Also know: Word Search II — [LeetCode 212](https://leetcode.com/problems/word-search-ii/)

Given a grid of letters and a list of words, find which words appear. Brute-force DFS per word is far too slow. The trick: **build a trie of all the words**, then DFS the grid **once**, walking the trie in lockstep with your grid path — the moment the current path isn't a prefix in the trie, **prune** (stop). This fuses:

- **grid DFS / backtracking** across neighboring cells — [16-graphs-bfs-dfs.md](16-graphs-bfs-dfs.md) and [11-recursion-and-backtracking.md](11-recursion-and-backtracking.md), and
- the **trie** as a live "is this still a valid prefix?" oracle that kills dead branches instantly.

That prefix-pruning is why the trie belongs here: it converts *"try every word everywhere"* into *"explore the grid only along paths some word could still complete."* It's the flagship example of a trie multiplying the power of another pattern.

---

## 7. Practice Set (curated LeetCode)

| # | Problem | LC | Difficulty | What it drills |
|---|---|---|---|---|
| 1 | Implement Trie (Prefix Tree) | 208 | Medium | build the base class (the template) |
| 2 | Design Add and Search Words | 211 | Medium | wildcard `.` → DFS over children |
| 3 | Longest Word in Dictionary | 720 | Medium | every prefix must be a word (flags on the path) |
| 4 | Replace Words | 648 | Medium | walk each word until you hit a stored root |
| 5 | Map Sum Pairs | 677 | Medium | store a value per word; sum a prefix subtree |
| 6 | Search Suggestions System | 1268 | Medium | autocomplete: top-3 completions per prefix |
| 7 | Implement Trie II (prefix counts) | 1804 | Medium | add `countWordsEqualTo` / `countWordsStartingWith` |
| 8 | Word Search II | 212 | Hard | trie + grid backtracking (prune by prefix) |

Start with **208** until you can type the class blind, then **211** for the wildcard twist, and only tackle **212** once grid DFS ([16](16-graphs-bfs-dfs.md)) feels comfortable.

---

## 8. Common Pitfalls & Interview Tips

| Pitfall | Fix / What to say |
|---|---|
| **Forgetting `isEndOfWord`.** Then `search` == `startsWith`, and `"app"` looks stored just because `"apple"` is. | Mark the last node on `insert`; require the flag in `search`. This is *the* trie bug interviewers watch for. |
| **Object vs. `Map` for children.** A plain `{}` inherits keys like `"constructor"` and needs `Object.keys()` to iterate. | Prefer **`Map`** (clean iteration, `.has`, `.size`, no prototype traps). If you must use an object, use `Object.create(null)`. A fixed `TrieNode[]` of size 26 is fastest but only for a known lowercase alphabet. |
| **Confusing `search` and `startsWith`.** | Only the last line differs: `search` needs `isEndOfWord`, `startsWith` just needs the node to exist. |
| **Ignoring memory cost.** Tries can be heavy — a node + map per character. | Acknowledge it: prefix speed is bought with space; mention compressed/radix tries exist for large dictionaries. |
| **Wildcard without backtracking.** Trying to match `.` down a single path. | At `.`, **recurse into every child** and return true if *any* branch matches. |
| **Not pruning in Word Search II.** DFS explores dead grid paths. | Use the trie to stop the instant the current path is no longer a valid prefix. |

**Talking track:** *"This is a prefix problem, so I'll use a trie — a character tree where each node has a children map and an end-of-word flag. Insert, search, and startsWith are all O(word length), independent of the dictionary size, which is why it beats a hash set for autocomplete: I walk to the prefix node and everything beneath it is a valid completion."* That names the structure, the fields, the complexity, and the *reason* — exactly the reasoning that earns the point.

---

## 9. TL;DR

- A **trie / prefix tree** is a tree where **each edge is a character** and each root-to-node path spells a **prefix**; words that share a prefix **share nodes**.
- Every node = a **`children` map** + an **`isEndOfWord` flag**. Forgetting that flag is the classic bug — it's the only thing distinguishing *a stored word* from *a passer-through prefix*.
- **`insert` / `search` / `startsWith` are all `O(L)`** (word length) — **independent of how many words** are stored. The cost is **memory**: a node per character.
- **Beats a hash set for prefixes:** a set does exact lookup in `O(L)` too, but **only a trie enumerates all words under a prefix** without scanning every key → that's **autocomplete**.
- **`search` vs `startsWith`:** identical walk; `search` additionally requires `isEndOfWord`.
- **Wildcards** (`.`): at a wildcard, **DFS into all children** — backtracking ([11](11-recursion-and-backtracking.md)).
- **Word Search II:** a trie of the word list **prunes grid DFS** ([16](16-graphs-bfs-dfs.md)) — the flagship "trie supercharges another pattern" problem.
- **Trigger words:** "prefix", "starts with", "autocomplete", "dictionary of words", "wildcard search", "word search on a board".
- **Frequency:** moderate — but a **strong signal** when it fits. If you see "**prefix**", think **trie**. When you only need exact membership/counting, stay with a hash set ([03](03-hashing-maps-and-sets.md)).

**Next:** [16 — Graphs: BFS & DFS](16-graphs-bfs-dfs.md) — the traversals behind grid problems and the trie-powered Word Search II.
