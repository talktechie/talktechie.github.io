---
title: "Backtracking: Word Search — Kotlin Solution"
date: 2026-06-24
categories: [Kotlin DSA, Backtracking]
tags: [word-search, backtracking, matrix, dfs, medium, leetcode, leetcode-79]
leetcode_number: 79
leetcode_url: https://leetcode.com/problems/word-search/
difficulty: Medium
permalink: /posts/kotlin-dsa-backtracking-word-search/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [79 — Word Search](https://leetcode.com/problems/word-search/) |
| **Difficulty** | Medium |
| **Topic** | Backtracking, Matrix, DFS |

> Given an `m x n` grid of characters `board` and a string `word`, return
> `true` if `word` can be constructed from letters of sequentially
> adjacent cells (horizontally or vertically neighboring), using the
> **same cell at most once**.

**Example:**
```
Input:
board = [["A","B","C","E"],
         ["S","F","C","S"],
         ["A","D","E","E"]]
word = "ABCCED"
Output: true

word = "SEE"
Output: true

word = "ABCB"
Output: false
Explanation: the second 'B' would require reusing the same cell as the
             first 'B' — not allowed
```

**Constraints:**
- `m == board.length`, `n == board[i].length`
- `1 <= m, n <= 6`
- `1 <= word.length <= 15`

---

## Approach

This is the single-word predecessor to Word Search II (covered in the
Trie phase) — here we're searching for just **one** target word, so a
Trie isn't necessary; direct backtracking from every starting cell is
simpler and sufficient.

**Key insight:** Try starting the search from **every** cell in the
grid. From a given cell, if its character matches the **current**
character we need from `word`, mark it temporarily visited and
recursively check all 4 neighbors for the **next** character. If a path
fails, backtrack (un-mark the cell) and try a different direction.

**Walk through `word = "SEE"` starting near the matching 'S' at
board[1][3]:**
```
dfs(row=1, col=3, charIndex=0, board[1][3]='S'):
  matches word[0]='S' → mark visited, check word[1]='E' next
  neighbors of (1,3): (0,3)='E', (2,3)='E', (1,2)='C'
    try (0,3)='E': matches word[1] → mark visited, check word[2]='E'
      neighbors of (0,3): (1,3) already visited, no others in bounds
        with 'E' available → dead end here? Let's check (2,3) path instead
    try (2,3)='E': matches word[1] → mark visited, check word[2]='E'
      neighbors of (2,3): (1,3) visited, (2,2)='E' → matches word[2]!
        charIndex reaches word.length → FOUND → return true ✓
```

---

## Kotlin Solution

### Approach 1 — Backtracking DFS from every cell, mark/unmark visited in-place (optimal)

```kotlin
fun exist(board: Array<CharArray>, word: String): Boolean {
    val rows = board.size
    val cols = board[0].size

    fun dfs(r: Int, c: Int, index: Int): Boolean {
        if (index == word.length) return true   // matched every character
        if (r < 0 || r >= rows || c < 0 || c >= cols) return false
        if (board[r][c] != word[index]) return false

        val temp = board[r][c]
        board[r][c] = '#'   // mark visited (using a character no valid input contains)

        val found = dfs(r + 1, c, index + 1) ||
                    dfs(r - 1, c, index + 1) ||
                    dfs(r, c + 1, index + 1) ||
                    dfs(r, c - 1, index + 1)

        board[r][c] = temp   // backtrack — restore for other paths/starting points

        return found
    }

    for (r in 0 until rows) {
        for (c in 0 until cols) {
            if (dfs(r, c, 0)) return true
        }
    }

    return false
}
```

### Approach 2 — Using a separate `visited` boolean grid (doesn't mutate input)

```kotlin
fun exist(board: Array<CharArray>, word: String): Boolean {
    val rows = board.size
    val cols = board[0].size
    val visited = Array(rows) { BooleanArray(cols) }

    fun dfs(r: Int, c: Int, index: Int): Boolean {
        if (index == word.length) return true
        if (r < 0 || r >= rows || c < 0 || c >= cols) return false
        if (visited[r][c] || board[r][c] != word[index]) return false

        visited[r][c] = true

        val found = dfs(r + 1, c, index + 1) ||
                    dfs(r - 1, c, index + 1) ||
                    dfs(r, c + 1, index + 1) ||
                    dfs(r, c - 1, index + 1)

        visited[r][c] = false   // backtrack

        return found
    }

    for (r in 0 until rows) {
        for (c in 0 until cols) {
            if (dfs(r, c, 0)) return true
        }
    }

    return false
}
```

Avoids touching `board` directly, at the cost of an extra O(rows × cols)
array. Useful if the original board must remain completely untouched for
some external reason (not required by this problem, but a common
real-world constraint).

---

## Why `||` Short-Circuits the Direction Checks Efficiently

```kotlin
val found = dfs(r + 1, c, index + 1) ||
            dfs(r - 1, c, index + 1) ||
            dfs(r, c + 1, index + 1) ||
            dfs(r, c - 1, index + 1)
```

Kotlin's `||` stops evaluating as soon as one operand is `true` — the
moment **any** direction successfully completes the word, the remaining
directions are never even checked. This is backtracking's "succeed fast"
counterpart to "fail fast" pruning: once we know an answer exists, there's
no value in continuing to search for alternative paths.

**Why marking the cell as visited (and restoring it) is essential:**
without this, the same cell could be reused multiple times within a
single word search — e.g., searching for `"ABCB"` over a grid containing
just one `"AB"` pair could incorrectly succeed by reusing the same `'B'`
cell twice, violating the problem's "use a cell at most once per search"
rule. The temporary mutation, undone immediately after exploring all 4
neighbors, is exactly the backtracking discipline seen throughout this
phase: **try → recurse → undo**, every single time.

**Why `'#'` is a safe sentinel value:** the problem doesn't specify
character constraints precisely, but `'#'` is a common safe choice since
it's distinguishable from typical alphabetic grid contents — any
character guaranteed not to appear in valid input works.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| In-place mutation (Approach 1) | Always preferred — O(1) extra space, simplest to write |
| Separate `visited` grid (Approach 2) | Original board must remain untouched throughout the search |

---

## Complexity

| | |
|---|---|
| **Time** | O(rows × cols × 4^L) worst case — L = word length; in practice much faster due to early mismatches |
| **Space** | O(L) — recursion depth |

---

## Key Takeaway

> Word Search is grid-based backtracking: try every starting cell, and
> from each, explore all 4 directions recursively while matching against
> the target word character by character — marking cells visited to
> prevent reuse, and always restoring them on backtrack. The pattern of
> mark → recurse in all directions → unmark is identical in spirit to the
> "choose → recurse → undo" backbone from Subsets, just applied to grid
> coordinates instead of array indices. This problem is the direct
> single-word stepping stone to Word Search II's Trie-based multi-word
> generalization.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/word-search/)

---

{% include kotlin-dsa-nav.html %}
