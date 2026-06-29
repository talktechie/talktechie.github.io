---
title: "2-D DP: Longest Increasing Path In a Matrix — Kotlin Solution"
date: 2026-06-29
categories: [Kotlin DSA, 2-D Dynamic Programming]
tags: [longest-increasing-path-matrix, dynamic-programming, dfs, memoization, matrix, hard, leetcode, leetcode-329]
leetcode_number: 329
leetcode_url: https://leetcode.com/problems/longest-increasing-path-in-a-matrix/
difficulty: Hard
permalink: /posts/kotlin-dsa-2d-dp-longest-increasing-path-in-a-matrix/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [329 — Longest Increasing Path In a Matrix](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/) |
| **Difficulty** | Hard |
| **Topic** | Dynamic Programming, DFS, Memoization, Matrix |

> Given an `m x n` integer matrix, return the length of the **longest
> strictly increasing path**, moving in any of the 4 directions
> (no diagonals, no wraparound).

**Example:**
```
Input:  matrix = [[9,9,4],[6,6,8],[2,1,1]]
Output: 4
Explanation: the path 1→2→6→9 (bottom-left going up-left-ish) has length 4

Input:  matrix = [[3,4,5],[3,2,6],[2,2,1]]
Output: 4
```

**Constraints:**
- `m == matrix.length`, `n == matrix[i].length`
- `1 <= m, n <= 200`
- `0 <= matrix[i][j] <= 2³¹ - 1`

---

## Approach

This combines **DFS** (from the Graphs phase) with **memoization** — a
"DFS + cache" technique sometimes called "memoized DFS" or treated as
DP on an implicit DAG (since strictly increasing values mean no cycles
are ever possible).

**Key insight:** Define `dp[r][c]` as the length of the longest
increasing path **starting** at cell `(r,c)`. For each cell, recursively
explore all 4 neighbors with a **strictly greater** value, taking
`1 + max(neighbor results)`. Memoize each cell's result the first time
it's computed — since the matrix's values are fixed, a cell's "longest
path starting here" never changes no matter how many other paths visit
it.

**Walk through a portion of `matrix = [[9,9,4],[6,6,8],[2,1,1]]`,
computing `dp[2][1]` (value=1):**
```
dp(2,1)=1: neighbors with greater value: (1,1)=6, (2,0)=2, (2,2)=1(not greater)
  dp(1,1)=6: neighbors greater than 6: (0,1)=9, (1,2)=8
    dp(0,1)=9: neighbors greater than 9: none → dp(0,1)=1
    dp(1,2)=8: neighbors greater than 8: (0,2)=4(not greater)... 
      wait, need greater than 8: none qualify → dp(1,2)=1
    dp(1,1) = 1 + max(dp(0,1)=1, dp(1,2)=1) = 1+1 = 2
  dp(2,0)=2: neighbors greater than 2: (1,0)=6
    dp(1,0)=6: neighbors greater than 6: (0,0)=9
      dp(0,0)=9: no greater neighbors → dp(0,0)=1
    dp(1,0) = 1 + dp(0,0) = 1+1 = 2
  dp(2,0) = 1 + dp(1,0) = 1+2 = 3
  dp(2,1) = 1 + max(dp(1,1)=2, dp(2,0)=3) = 1+3 = 4

Longest path starting at (2,1): 4 ✓ (matches expected output:
the path 1→2→6→9, exactly this chain)
```

---

## Kotlin Solution

### Approach 1 — Memoized DFS (optimal)

```kotlin
fun longestIncreasingPath(matrix: Array<IntArray>): Int {
    val rows = matrix.size
    val cols = matrix[0].size
    val memo = Array(rows) { IntArray(cols) }   // 0 = not yet computed
    val directions = listOf(0 to 1, 0 to -1, 1 to 0, -1 to 0)

    fun dfs(r: Int, c: Int): Int {
        if (memo[r][c] != 0) return memo[r][c]   // already computed

        var best = 1   // the cell itself counts as a path of length 1

        for ((dr, dc) in directions) {
            val nr = r + dr
            val nc = c + dc
            if (nr in 0 until rows && nc in 0 until cols && matrix[nr][nc] > matrix[r][c]) {
                best = maxOf(best, 1 + dfs(nr, nc))
            }
        }

        memo[r][c] = best
        return best
    }

    var answer = 0
    for (r in 0 until rows) {
        for (c in 0 until cols) {
            answer = maxOf(answer, dfs(r, c))
        }
    }

    return answer
}
```

### Approach 2 — Topological-sort-style BFS (peel off "peaks" layer by layer)

```kotlin
fun longestIncreasingPath(matrix: Array<IntArray>): Int {
    val rows = matrix.size
    val cols = matrix[0].size
    val directions = listOf(0 to 1, 0 to -1, 1 to 0, -1 to 0)

    // outDegree[r][c] = number of neighbors with a STRICTLY GREATER value
    val outDegree = Array(rows) { IntArray(cols) }
    for (r in 0 until rows) {
        for (c in 0 until cols) {
            for ((dr, dc) in directions) {
                val nr = r + dr; val nc = c + dc
                if (nr in 0 until rows && nc in 0 until cols && matrix[nr][nc] > matrix[r][c]) {
                    outDegree[r][c]++
                }
            }
        }
    }

    val queue: ArrayDeque<Pair<Int, Int>> = ArrayDeque()
    for (r in 0 until rows) {
        for (c in 0 until cols) {
            if (outDegree[r][c] == 0) queue.addLast(r to c)   // "peak" cells — nowhere greater to go
        }
    }

    var length = 0
    while (queue.isNotEmpty()) {
        length++
        repeat(queue.size) {
            val (r, c) = queue.removeFirst()
            for ((dr, dc) in directions) {
                val nr = r + dr; val nc = c + dc
                if (nr in 0 until rows && nc in 0 until cols && matrix[nr][nc] < matrix[r][c]) {
                    outDegree[nr][nc]--
                    if (outDegree[nr][nc] == 0) queue.addLast(nr to nc)
                }
            }
        }
    }

    return length
}
```

Processes cells in reverse: starts from "peak" cells (no greater
neighbor) and peels inward toward smaller values, layer by layer — this
is exactly Course Schedule II's Kahn's algorithm topological sort,
applied to the implicit DAG where edges point from smaller to larger
values.

---

## Why Memoization Is Valid Here Despite Looking Like Graph DFS

Ordinary DFS on a graph with cycles needs a `visited` set to avoid
infinite loops — but a **strictly increasing** path can never revisit a
cell (values only ever increase along the path, so returning to an
earlier cell would require a value to both increase and later decrease
back to itself, impossible). This guarantees the implicit "greater than"
relationship forms a **DAG** (Directed Acyclic Graph), making
memoization both safe and highly effective:

```kotlin
if (memo[r][c] != 0) return memo[r][c]   // safe — this cell's answer
                                            // can NEVER change, since the
                                            // matrix values are fixed
```

Once `dp[r][c]` (the longest increasing path **starting** at `(r,c)`) is
computed, it's permanently correct — no future computation could ever
discover a different value, because it depends only on the fixed matrix
values themselves, not on which cell happened to call into it first.

**Why Approach 2's "peel from the peaks" ordering works:** this is
exactly Kahn's algorithm, but applied to a DAG built implicitly from
"value comparisons" rather than explicit input edges. Peaks (cells with
no strictly-greater neighbor) are the natural "leaves" of this DAG, and
processing them first (then their predecessors, layer by layer) mirrors
topological sort's "process nodes with in-degree/out-degree zero first"
strategy.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Memoized DFS (Approach 1) | Most intuitive — directly answers "longest path starting here," reused via caching |
| Topological BFS (Approach 2) | Want to see the explicit DAG-layering structure, avoids recursion entirely |

---

## Complexity

| | |
|---|---|
| **Time** | O(rows × cols) — each cell's DP value computed exactly once |
| **Space** | O(rows × cols) — the memo table / out-degree tracking |

---

## Key Takeaway

> This Hard-difficulty problem shows that DP doesn't always need an
> explicit grid recurrence written by hand — sometimes the cleanest
> approach is DFS **augmented with memoization**, exploiting a problem
> property (here, strict monotonicity) that guarantees no cycles, and
> therefore guarantees each subproblem's answer is computed exactly
> once and never needs revisiting. Recognizing when a graph-traversal
> problem secretly has a DAG structure — making memoized DFS just as
> valid and efficient as an explicit DP table — is a valuable bridge
> between the Graphs phase and 2-D DP.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/)

---

{% include kotlin-dsa-nav.html %}
