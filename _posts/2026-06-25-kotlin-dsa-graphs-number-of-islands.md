---
title: "Graphs: Number of Islands — Kotlin Solution"
date: 2026-06-25
categories: [Kotlin DSA, Graphs]
tags: [number-of-islands, graph, dfs, bfs, matrix, medium, leetcode, leetcode-200]
leetcode_number: 200
leetcode_url: https://leetcode.com/problems/number-of-islands/
difficulty: Medium
permalink: /posts/kotlin-dsa-graphs-number-of-islands/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [200 — Number of Islands](https://leetcode.com/problems/number-of-islands/) |
| **Difficulty** | Medium |
| **Topic** | Graph, DFS, BFS, Matrix |

> Given an `m x n` 2D grid of `'1'`s (land) and `'0'`s (water), return
> the number of **islands** — a group of `'1'`s connected
> horizontally or vertically (not diagonally), surrounded by water.

**Example:**
```
Input:
[["1","1","1","1","0"],
 ["1","1","0","1","0"],
 ["1","1","0","0","0"],
 ["0","0","0","0","0"]]
Output: 1

Input:
[["1","1","0","0","0"],
 ["1","1","0","0","0"],
 ["0","0","1","0","0"],
 ["0","0","0","1","1"]]
Output: 3
```

**Constraints:**
- `m == grid.length`, `n == grid[i].length`
- `1 <= m, n <= 300`
- `grid[i][j]` is `'0'` or `'1'`

---

## Approach

This is the entry point to the Graphs phase, establishing that **a grid
is just a graph** — each cell is a node, and adjacent cells (up, down,
left, right) are edges. Treating it this way unlocks every standard
graph traversal technique.

**Key insight:** Scan every cell. Whenever an unvisited `'1'` is found,
that's the start of a **new** island — increment the count, then flood-fill
outward (DFS or BFS) to mark every connected `'1'` as visited, so this
same island is never counted again.

**Walk through the second example grid:**
```
Scan (0,0)='1', unvisited → NEW ISLAND #1
  flood-fill marks (0,0),(0,1),(1,0),(1,1) as visited (all connected)

Scan (0,2)='0' → skip
...continue scanning...
Scan (2,2)='1', unvisited → NEW ISLAND #2
  flood-fill marks just (2,2) — no connected neighbors

Scan (3,3)='1', unvisited → NEW ISLAND #3
  flood-fill marks (3,3),(3,4) (connected horizontally)

Total islands found: 3 ✓
```

---

## Kotlin Solution

### Approach 1 — DFS flood-fill, mark visited in-place (optimal, most common)

```kotlin
fun numIslands(grid: Array<CharArray>): Int {
    val rows = grid.size
    val cols = grid[0].size
    var count = 0

    fun dfs(r: Int, c: Int) {
        if (r < 0 || r >= rows || c < 0 || c >= cols || grid[r][c] != '1') return

        grid[r][c] = '0'   // mark visited by "sinking" the land

        dfs(r + 1, c)
        dfs(r - 1, c)
        dfs(r, c + 1)
        dfs(r, c - 1)
    }

    for (r in 0 until rows) {
        for (c in 0 until cols) {
            if (grid[r][c] == '1') {
                count++
                dfs(r, c)
            }
        }
    }

    return count
}
```

### Approach 2 — BFS flood-fill with an explicit queue

```kotlin
fun numIslands(grid: Array<CharArray>): Int {
    val rows = grid.size
    val cols = grid[0].size
    var count = 0

    fun bfs(startR: Int, startC: Int) {
        val queue: ArrayDeque<Pair<Int, Int>> = ArrayDeque()
        queue.addLast(startR to startC)
        grid[startR][startC] = '0'

        val directions = listOf(0 to 1, 0 to -1, 1 to 0, -1 to 0)

        while (queue.isNotEmpty()) {
            val (r, c) = queue.removeFirst()

            for ((dr, dc) in directions) {
                val nr = r + dr
                val nc = c + dc
                if (nr in 0 until rows && nc in 0 until cols && grid[nr][nc] == '1') {
                    grid[nr][nc] = '0'
                    queue.addLast(nr to nc)
                }
            }
        }
    }

    for (r in 0 until rows) {
        for (c in 0 until cols) {
            if (grid[r][c] == '1') {
                count++
                bfs(r, c)
            }
        }
    }

    return count
}
```

Avoids recursion entirely — useful if the grid is large enough that DFS
recursion depth could become a concern (up to 300×300 = 90,000 cells).

---

## Why "Sinking" the Land (Mutating the Grid) Is the Simplest Visited-Tracking

```kotlin
grid[r][c] = '0'   // once visited, this cell can never trigger a new
                    // island count again, and DFS won't recurse into it
```

This reuses the grid itself as the visited-tracking structure — no
separate `visited` boolean array needed. It's safe because the problem
doesn't require the original grid to remain unmodified afterward. If it
did, a separate `Array(rows) { BooleanArray(cols) }` would serve the
same purpose at the cost of O(rows × cols) extra space.

**Why the boundary check comes first in the DFS base case:**
```kotlin
if (r < 0 || r >= rows || c < 0 || c >= cols || grid[r][c] != '1') return
```
Short-circuit evaluation means `grid[r][c]` is never accessed if `r` or
`c` is already out of bounds — checking bounds before the array access
prevents an index-out-of-bounds exception.

**Why every new, unvisited `'1'` found during the scan must be a new
island:** because flood-fill from any previously found island already
marked every cell reachable from it as `'0'` — so by the time the main
scan reaches a still-`'1'` cell, it's guaranteed to belong to an island
we haven't counted yet.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| DFS (Approach 1) | Simpler to write, fine for typical grid sizes in this problem's constraints |
| BFS (Approach 2) | Want to avoid deep recursion on very large or long, snake-like islands |

---

## Complexity

| | |
|---|---|
| **Time** | O(rows × cols) — every cell visited at most a constant number of times |
| **Space** | O(rows × cols) worst case — recursion stack (DFS) or queue (BFS) in the worst case of one giant island |

---

## Key Takeaway

> A grid is a graph in disguise — every cell is a node, and adjacency
> means a shared edge. "Count connected components" (here, islands) is
> solved by scanning every node, and whenever an unvisited one is found,
> flood-filling (DFS or BFS) to mark its entire connected component, while
> incrementing a counter once per component discovered. This exact
> "scan + flood-fill + count" template is the foundation for nearly every
> other problem in this phase, including Max Area of Island and Number of
> Connected Components.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/number-of-islands/)

---

{% include kotlin-dsa-nav.html %}
