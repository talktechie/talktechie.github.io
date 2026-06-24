---
title: "Trie: Word Search II — Kotlin Solution"
date: 2026-06-22
categories: [Kotlin DSA, Trie]
tags: [word-search-ii, trie, dfs, backtracking, matrix, hard, leetcode, leetcode-212]
leetcode_number: 212
leetcode_url: https://leetcode.com/problems/word-search-ii/
difficulty: Hard
permalink: /posts/kotlin-dsa-trie-word-search-ii/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [212 — Word Search II](https://leetcode.com/problems/word-search-ii/) |
| **Difficulty** | Hard |
| **Topic** | Trie, DFS, Backtracking, Matrix |

> Given an `m x n` grid of characters `board` and a list of strings
> `words`, return **all words** from `words` that can be formed by a
> path of adjacent cells in the grid (horizontally or vertically
> connected, no cell reused within a single word).

**Example:**
```
Input:
board = [["o","a","a","n"],
         ["e","t","a","e"],
         ["i","h","k","r"],
         ["i","f","l","v"]]
words = ["oath","pea","eat","rain"]

Output: ["eat","oath"]
Explanation: "oath" can be traced starting top-left going down/right;
             "eat" can be traced through the grid; "pea" and "rain"
             cannot be formed
```

**Constraints:**
- `m == board.length`, `n == board[i].length`
- `1 <= m, n <= 12`
- `1 <= words.length <= 3 * 10⁴`
- `1 <= words[i].length <= 10`
- All `words[i]` are unique

---

## Approach

The naive approach — run a separate DFS/backtracking search on the grid
for **each** word in the list — works for Word Search I (a single word)
but is wasteful here: with up to `3 * 10⁴` words, many of which share
common prefixes, we'd repeat enormous amounts of redundant work exploring
the same grid paths over and over for different words.

**Key insight — combine all words into a single Trie, then do ONE DFS
over the grid:** Build a Trie from every word in `words`. Then run a
single DFS/backtracking search across the grid, where at every step we
check **whether the current path matches a node in the Trie** rather than
matching against one specific target word. The moment we reach a Trie
node marked `isWord = true`, we've found a complete word — record it.
This way, words sharing a prefix (like "eat" and "eats") share the same
grid-exploration work for that shared prefix.

**Additional optimization — prune found words from the Trie:** once a
word is found, remove its `isWord` marker (or the whole branch if it has
no children) so we never re-report it and so dead-end branches stop
being explored on future calls.

**Walk through finding `"oath"` starting at board[0][0]='o':**
```
Trie contains paths for: oath, pea, eat, rain

dfs(row=0, col=0, trieNode=root):
  board[0][0]='o' → root has child 'o'? yes → move to that trie node
  mark board[0][0] as visited, explore neighbors:
    dfs(row=1, col=0, trieNode=trie['o']):
      board[1][0]='e' → does trie['o'] have child 'e'? NO → dead end, backtrack

    dfs(row=0, col=1, trieNode=trie['o']):
      board[0][1]='a' → trie['o'] has child 'a'? yes → move deeper
      ... continues tracing 'a'→'t'→'h' ...
      reaches a trie node with isWord=true for "oath" → add "oath" to results ✓
```

---

## Kotlin Solution

### Approach 1 — Build one Trie, single DFS over the grid (optimal)

```kotlin
class TrieNode {
    val children = HashMap<Char, TrieNode>()
    var word: String? = null   // non-null exactly when a word ends here
}

fun findWords(board: Array<CharArray>, words: Array<String>): List<String> {
    val root = TrieNode()

    // Build the Trie from all target words
    for (w in words) {
        var node = root
        for (c in w) {
            node = node.children.getOrPut(c) { TrieNode() }
        }
        node.word = w
    }

    val rows = board.size
    val cols = board[0].size
    val result = mutableListOf<String>()

    fun dfs(r: Int, c: Int, node: TrieNode) {
        if (r < 0 || r >= rows || c < 0 || c >= cols) return

        val char = board[r][c]
        if (char == '#') return  // already visited in this path

        val next = node.children[char] ?: return  // no matching Trie path — prune

        if (next.word != null) {
            result.add(next.word!!)
            next.word = null   // avoid duplicate results
        }

        board[r][c] = '#'  // mark visited

        dfs(r + 1, c, next)
        dfs(r - 1, c, next)
        dfs(r, c + 1, next)
        dfs(r, c - 1, next)

        board[r][c] = char  // backtrack — restore for other starting points

        // Prune leaf nodes with no remaining words below them (optimization)
        if (next.children.isEmpty()) {
            node.children.remove(char)
        }
    }

    for (r in 0 until rows) {
        for (c in 0 until cols) {
            dfs(r, c, root)
        }
    }

    return result
}
```

### Approach 2 — Without Trie pruning (simpler, slightly slower on large inputs)

```kotlin
fun findWords(board: Array<CharArray>, words: Array<String>): List<String> {
    val root = TrieNode()
    for (w in words) {
        var node = root
        for (c in w) node = node.children.getOrPut(c) { TrieNode() }
        node.word = w
    }

    val rows = board.size
    val cols = board[0].size
    val found = HashSet<String>()

    fun dfs(r: Int, c: Int, node: TrieNode) {
        if (r < 0 || r >= rows || c < 0 || c >= cols || board[r][c] == '#') return

        val next = node.children[board[r][c]] ?: return
        next.word?.let { found.add(it) }

        val temp = board[r][c]
        board[r][c] = '#'

        dfs(r + 1, c, next)
        dfs(r - 1, c, next)
        dfs(r, c + 1, next)
        dfs(r, c - 1, next)

        board[r][c] = temp
    }

    for (r in 0 until rows) for (c in 0 until cols) dfs(r, c, root)

    return found.toList()
}
```

Uses a `HashSet` to naturally avoid duplicate results instead of mutating
the Trie, and skips the leaf-pruning optimization — simpler to write, but
doesn't benefit from shrinking the Trie as words are found, so later
searches don't get faster over time.

---

## Why Combining Words Into One Trie Is the Critical Optimization

Searching for each word independently means re-deriving "which grid paths
are even plausible" from scratch every time — including for words sharing
long common prefixes like `"eat"` and `"eats"`. By building one Trie:

```kotlin
val next = node.children[char] ?: return  // PRUNE immediately
// If the grid's current character has no corresponding Trie edge,
// there's no point continuing down this path for ANY word — not just
// the one we might have been "targeting"
```

Every grid cell visited during the single combined DFS is doing useful
work for **every** word that shares the prefix traced so far
simultaneously, rather than redundantly re-exploring the same cells once
per word in the list.

**Why we remove `next.word` after finding it:**
```kotlin
if (next.word != null) {
    result.add(next.word!!)
    next.word = null   // this exact node won't be reported again
}
```
Without this, the same word could be discovered multiple times via
different starting cells or paths, producing duplicate entries.

**Why pruning empty Trie branches matters at scale:**
```kotlin
if (next.children.isEmpty()) {
    node.children.remove(char)
}
```
Once a branch of the Trie has no more words to find (its only word was
already found, and it has no children leading to other words), removing
it means future DFS calls starting from other grid cells won't waste time
re-exploring that now-useless path.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Trie with pruning (Approach 1) | Always — meaningfully faster on large word lists, the intended optimal solution |
| Trie without pruning (Approach 2) | Smaller inputs, or want simpler code without the pruning complexity |

---

## Complexity

| | |
|---|---|
| **Time** | O(rows × cols × 4^L) worst case — L = max word length; Trie sharing and pruning make this far better in practice |
| **Space** | O(N) for the Trie — N = total characters across all words |

---

## Key Takeaway

> When you need to search a grid (or any large search space) against
> **many** target patterns simultaneously, combine them into a single
> Trie and run **one** traversal — checking Trie-path validity instead
> of matching against one specific word at a time. This closes out the
> Trie phase by combining everything: the Trie structure from LC 208,
> the DFS-with-backtracking from LC 211 (here driven by grid adjacency
> instead of wildcards), and a new pruning technique — removing
> already-found words from the Trie — to keep repeated searches fast as
> the search space shrinks.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/word-search-ii/)

---

{% include kotlin-dsa-nav.html %}
