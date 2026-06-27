---
title: "Graphs: Max Area of Island — Kotlin Solution"
date: 2026-06-25
categories: [Kotlin DSA, Graphs]
tags: [max-area-of-island, graph, dfs, bfs, matrix, medium, leetcode, leetcode-695]
leetcode_number: 695
leetcode_url: https://leetcode.com/problems/max-area-of-island/
difficulty: Medium
permalink: /posts/kotlin-dsa-graphs-max-area-of-island/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [695 — Max Area of Island](https://leetcode.com/problems/max-area-of-island/) |
| **Difficulty** | Medium |
| **Topic** | Graph, DFS, BFS, Matrix |

> Given an `m x n` binary grid, an **island** is a group of `1`s
> connected horizontally/vertically. Return the **area** (number of
> cells) of the **largest** island. If there's no island, return `0`.

**Example:**
```
Input:
[[0,0,1,0,0,0,0,1,0,0,0,0,0],
 [0,0,0,0,0,0,0,1,1,1,0,0,0],
 [0,1,1,0,1,0,0,0,0,0,0,0,0],
 [0,1,0,0,1,1,0,0,1,0,1,0,0],
 [0,1,0,0,1,1,0,0,1,1,1,0,0],
 [0,0,0,0,0,0,0,0,0,0,1,0,0],
 [0,0,0,0,0,0,0,1,1,1,0,0,0],
 [0,0,0,0,0,0,0,1,1,0,0,0,0]]
Output: 6
Explanation: the largest island (around rows 3-4) has area 6

Input:  [[0,0,0,0,0,0,0,0]]
Output: 0
```

**Constraints:**
- `m == grid.length`, `n == grid[i].length`
- `1 <= m, n <= 50`
- `grid[i][j]` is `0` or `1`

---

## Approach

This is a direct extension of Number of Islands — same flood-fill
mechanics, but instead of just counting islands, each flood-fill call
needs to **return its area** so we can track the maximum across all of them.

**Key insight:** Modify the flood-fill function to **return a count**
(1 for the current cell, plus the sum of areas from all 4 recursive
directions) instead of just marking visited cells with no return value.
Track the running maximum across every flood-fill call triggered during
the main scan.

**Walk through a small island shaped like an L:**
```
grid:
1 0
1 1

dfs(0,0): mark visited, area = 1 + dfs(1,0) + dfs(0,1) [via the other 2 directions=0]
  dfs(1,0): mark visited, area = 1 + dfs(2,0)[out of bounds=0] + dfs(1,1)
    dfs(1,1): mark visited, area = 1 + (no more unvisited neighbors) = 1
  dfs(1,0) total = 1 + 0 + 1 = 2
dfs(0,0) total = 1 + 2 + 0 = 3

Max area found: 3 ✓ (matches the 3 connected '1' cells)
```

---

## Kotlin Solution

### Approach 1 — DFS flood-fill returning area, track running max (optimal)

```kotlin
fun maxAreaOfIsland(grid: Array<IntArray>): Int {
    val rows = grid.size
    val cols = grid[0].size

    fun dfs(r: Int, c: Int): Int {
        if (r < 0 || r >= rows || c < 0 || c >= cols || grid[r][c] != 1) return 0

        grid[r][c] = 0   // mark visited

        return 1 + dfs(r + 1, c) + dfs(r - 1, c) + dfs(r, c + 1) + dfs(r, c - 1)
    }

    var maxArea = 0

    for (r in 0 until rows) {
        for (c in 0 until cols) {
            if (grid[r][c] == 1) {
                maxArea = maxOf(maxArea, dfs(r, c))
            }
        }
    }

    return maxArea
}
```

### Approach 2 — BFS flood-fill, counting cells visited per island

```kotlin
fun maxAreaOfIsland(grid: Array<IntArray>): Int {
    val rows = grid.size
    val cols = grid[0].size

    fun bfs(startR: Int, startC: Int): Int {
        val queue: ArrayDeque<Pair<Int, Int>> = ArrayDeque()
        queue.addLast(startR to startC)
        grid[startR][startC] = 0
        var area = 0

        val directions = listOf(0 to 1, 0 to -1, 1 to 0, -1 to 0)

        while (queue.isNotEmpty()) {
            val (r, c) = queue.removeFirst()
            area++

            for ((dr, dc) in directions) {
                val nr = r + dr
                val nc = c + dc
                if (nr in 0 until rows && nc in 0 until cols && grid[nr][nc] == 1) {
                    grid[nr][nc] = 0
                    queue.addLast(nr to nc)
                }
            }
        }

        return area
    }

    var maxArea = 0

    for (r in 0 until rows) {
        for (c in 0 until cols) {
            if (grid[r][c] == 1) {
                maxArea = maxOf(maxArea, bfs(r, c))
            }
        }
    }

    return maxArea
}
```

---

## Why Returning a Value Is the Only Real Change From Number of Islands

The traversal mechanics — bounds check, mark visited, recurse in 4
directions — are **identical** to Number of Islands. The only difference
is what the flood-fill function communicates back to its caller:

```kotlin
// Number of Islands' dfs: no return value, just marks cells visited
fun dfs(r: Int, c: Int) { ... }

// Max Area of Island's dfs: returns the count of cells in this island
fun dfs(r: Int, c: Int): Int {
    // ...
    return 1 + dfs(r + 1, c) + dfs(r - 1, c) + dfs(r, c + 1) + dfs(r, c - 1)
    // "1" for the current cell, plus whatever each direction's
    // recursive call contributes
}
```

This is a great illustration of how a flood-fill template can be
**augmented** to compute different things — counting components (Number
of Islands), measuring component size (this problem), or even collecting
the actual cells of a component, all using the same traversal skeleton
with a different payload threaded through the recursion.

**Why invalid cells return `0`, not skip silently:** the recursive sum
`1 + dfs(...) + dfs(...) + dfs(...) + dfs(...)` requires every call to
contribute *some* numeric value — `0` for "this direction added nothing"
keeps the arithmetic correct without needing special-case handling.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| DFS returning area (Approach 1) | Simpler to write, naturally expresses "this island's size is 1 plus its neighbors'" |
| BFS with counter (Approach 2) | Prefer iterative traversal, avoiding recursion depth concerns |

---

## Complexity

| | |
|---|---|
| **Time** | O(rows × cols) — every cell visited once |
| **Space** | O(rows × cols) worst case — recursion/queue depth for one giant island |

---

## Key Takeaway

> Once you have a working flood-fill template (from Number of Islands),
> extending it to compute something more specific — like component size
> — usually just means changing what the function **returns**, not how
> it traverses. This composability is one of the most valuable transfer
> skills in the Graphs phase: the same DFS/BFS skeleton answers very
> different questions depending on what payload you thread through the
> recursion and accumulate at the call site.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/max-area-of-island/)

---

{% include kotlin-dsa-nav.html %}
