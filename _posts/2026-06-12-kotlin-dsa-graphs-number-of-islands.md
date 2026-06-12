---
title: "Graphs: Number of Islands — Kotlin Solution"
date: 2026-06-12
categories: [Kotlin DSA, Graphs]
tags: [number-of-islands, graph, dfs, bfs, medium, leetcode, leetcode-200]
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
| **Topic** | Graph, DFS, BFS |

> Given an `m x n` 2D binary grid where `'1'` represents land and `'0'` represents water,
> return the number of islands.
>
> An island is surrounded by water and is formed by connecting adjacent lands
> horizontally or vertically.

**Example:**
```
Input:
11110
11010
11000
00000

Output: 1  (all connected land = one island)

Input:
11000
11000
00100
00011

Output: 3
```

---

## Approach

**DFS flood fill** — scan the grid. When you find a `'1'`, increment the island
count and flood-fill the entire connected land to `'0'` (mark as visited).
Next time you scan that area, it's all water — won't be counted again.

**Walk through grid 2:**
```
Find '1' at (0,0) → flood fill → marks (0,0)(0,1)(1,0)(1,1) as '0', count=1
Find '1' at (2,2) → flood fill → marks (2,2) as '0', count=2
Find '1' at (3,3) → flood fill → marks (3,3)(3,4) as '0', count=3
Result: 3 ✓
```

---

## Kotlin Solution

### Approach 1 — DFS (recursive)

```kotlin
fun numIslands(grid: Array<CharArray>): Int {
    var count = 0

    for (r in grid.indices) {
        for (c in grid[r].indices) {
            if (grid[r][c] == '1') {
                count++
                dfs(grid, r, c)
            }
        }
    }

    return count
}

private fun dfs(grid: Array<CharArray>, r: Int, c: Int) {
    if (r < 0 || r >= grid.size || c < 0 || c >= grid[0].size) return
    if (grid[r][c] != '1') return

    grid[r][c] = '0'  // mark visited

    dfs(grid, r + 1, c)
    dfs(grid, r - 1, c)
    dfs(grid, r, c + 1)
    dfs(grid, r, c - 1)
}
```

### Approach 2 — BFS (iterative)

```kotlin
fun numIslands(grid: Array<CharArray>): Int {
    var count = 0
    val directions = arrayOf(intArrayOf(1,0), intArrayOf(-1,0),
                             intArrayOf(0,1), intArrayOf(0,-1))

    for (r in grid.indices) {
        for (c in grid[r].indices) {
            if (grid[r][c] == '1') {
                count++
                val queue = ArrayDeque<Pair<Int, Int>>()
                queue.add(r to c)
                grid[r][c] = '0'

                while (queue.isNotEmpty()) {
                    val (row, col) = queue.removeFirst()
                    for ((dr, dc) in directions) {
                        val nr = row + dr
                        val nc = col + dc
                        if (nr in grid.indices && nc in grid[0].indices
                            && grid[nr][nc] == '1') {
                            grid[nr][nc] = '0'
                            queue.add(nr to nc)
                        }
                    }
                }
            }
        }
    }

    return count
}
```

---

## Why Kotlin Shines Here

**`grid.indices` and `grid[r].indices`** — clean range iteration:
```kotlin
for (r in grid.indices)
for (c in grid[r].indices)
// vs Java: for (int r = 0; r < grid.length; r++)
```

**`r to c`** — Pair creation with infix notation:
```kotlin
queue.add(r to c)
val (row, col) = queue.removeFirst()  // destructuring on dequeue
// vs Java: queue.add(new int[]{r, c}); int[] cell = queue.poll();
```

**`nr in grid.indices`** — bounds check reads like English:
```kotlin
if (nr in grid.indices && nc in grid[0].indices)
// vs Java: if (nr >= 0 && nr < grid.length && nc >= 0 && nc < grid[0].length)
```

**`for ((dr, dc) in directions)`** — destructuring in for loop:
```kotlin
for ((dr, dc) in directions) { ... }
// vs Java: for (int[] dir : directions) { int dr = dir[0]; int dc = dir[1]; }
```

---

## Complexity

| | |
|---|---|
| **Time** | O(m × n) — every cell visited at most once |
| **Space** | O(m × n) — recursion stack / BFS queue in worst case |

---

## Key Takeaway

> When you find land, flood-fill the entire island to water.
> Each flood-fill = one island. Count the flood-fills.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/number-of-islands/)

---

{% include kotlin-dsa-nav.html %}
