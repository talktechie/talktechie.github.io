---
title: "Graphs: Pacific Atlantic Water Flow — Kotlin Solution"
date: 2026-06-25
categories: [Kotlin DSA, Graphs]
tags: [pacific-atlantic-water-flow, graph, dfs, bfs, matrix, medium, leetcode, leetcode-417]
leetcode_number: 417
leetcode_url: https://leetcode.com/problems/pacific-atlantic-water-flow/
difficulty: Medium
permalink: /posts/kotlin-dsa-graphs-pacific-atlantic-water-flow/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [417 — Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/) |
| **Difficulty** | Medium |
| **Topic** | Graph, DFS, BFS, Matrix |

> Given an `m x n` grid of heights, the **Pacific** Ocean touches the
> top and left edges, the **Atlantic** touches the bottom and right
> edges. Water flows from a cell to an adjacent cell only if that
> neighbor's height is **less than or equal to** the current cell's
> height. Return all coordinates from which water can flow to **both**
> oceans.

**Example:**
```
Input:
[[1,2,2,3,5],
 [3,2,3,4,4],
 [2,4,5,3,1],
 [6,7,1,4,5],
 [5,1,1,2,4]]
Output: [[0,4],[1,3],[1,4],[2,2],[3,0],[3,1],[4,0]]

Input:  [[1]]
Output: [[0,0]]
```

**Constraints:**
- `m == heights.length`, `n == heights[r].length`
- `1 <= m, n <= 200`
- `0 <= heights[r][c] <= 10⁵`

---

## Approach

A naive approach — for every cell, simulate where water flows and check
if it reaches both oceans — is O((rows×cols)²), far too slow for a 200×200
grid. The key trick: **reverse the problem**, flowing **outward from the
oceans inward** instead of from each cell outward.

**Key insight:** Water flows from high to low. So reversed, we ask "can
the ocean reach this cell by flowing **uphill or equal** from the
border?" Run a flood-fill (DFS or BFS) starting from **every** Pacific
border cell, moving to neighbors with height **≥** current height
(reversed direction). Do the same separately for the Atlantic border.
A cell that's reachable from **both** flood-fills is part of the answer.

**Walk through identifying Pacific-reachable cells (top/left border
cells start the search):**
```
Start flood-fill from every cell along the top row and left column.
From each, move to a neighbor only if neighbor's height >= current
height (since we're simulating water flowing the OPPOSITE direction,
from the ocean backward uphill into the grid).

This single combined flood-fill (multi-source, just like Walls And
Gates) marks every cell that could ACTUALLY drain into the Pacific in
the real (forward) direction.

Repeat separately for the Atlantic, starting from bottom row + right column.

Intersect both reachable sets → cells in both = the answer.
```

---

## Kotlin Solution

### Approach 1 — Reverse multi-source DFS from each ocean's border (optimal)

```kotlin
fun pacificAtlantic(heights: Array<IntArray>): List<List<Int>> {
    val rows = heights.size
    val cols = heights[0].size

    val pacific = Array(rows) { BooleanArray(cols) }
    val atlantic = Array(rows) { BooleanArray(cols) }

    fun dfs(r: Int, c: Int, visited: Array<BooleanArray>) {
        visited[r][c] = true

        val directions = listOf(0 to 1, 0 to -1, 1 to 0, -1 to 0)
        for ((dr, dc) in directions) {
            val nr = r + dr
            val nc = c + dc
            if (nr in 0 until rows && nc in 0 until cols &&
                !visited[nr][nc] && heights[nr][nc] >= heights[r][c]) {
                dfs(nr, nc, visited)
            }
        }
    }

    for (c in 0 until cols) {
        dfs(0, c, pacific)            // top row touches Pacific
        dfs(rows - 1, c, atlantic)    // bottom row touches Atlantic
    }
    for (r in 0 until rows) {
        dfs(r, 0, pacific)            // left column touches Pacific
        dfs(r, cols - 1, atlantic)    // right column touches Atlantic
    }

    val result = mutableListOf<List<Int>>()
    for (r in 0 until rows) {
        for (c in 0 until cols) {
            if (pacific[r][c] && atlantic[r][c]) result.add(listOf(r, c))
        }
    }

    return result
}
```

### Approach 2 — Same idea, using BFS instead of DFS

```kotlin
fun pacificAtlantic(heights: Array<IntArray>): List<List<Int>> {
    val rows = heights.size
    val cols = heights[0].size
    val directions = listOf(0 to 1, 0 to -1, 1 to 0, -1 to 0)

    fun bfs(starts: List<Pair<Int, Int>>): Array<BooleanArray> {
        val visited = Array(rows) { BooleanArray(cols) }
        val queue: ArrayDeque<Pair<Int, Int>> = ArrayDeque()

        for ((r, c) in starts) {
            visited[r][c] = true
            queue.addLast(r to c)
        }

        while (queue.isNotEmpty()) {
            val (r, c) = queue.removeFirst()
            for ((dr, dc) in directions) {
                val nr = r + dr; val nc = c + dc
                if (nr in 0 until rows && nc in 0 until cols &&
                    !visited[nr][nc] && heights[nr][nc] >= heights[r][c]) {
                    visited[nr][nc] = true
                    queue.addLast(nr to nc)
                }
            }
        }

        return visited
    }

    val pacificStarts = (0 until cols).map { 0 to it } + (0 until rows).map { it to 0 }
    val atlanticStarts = (0 until cols).map { rows - 1 to it } + (0 until rows).map { it to cols - 1 }

    val pacific = bfs(pacificStarts)
    val atlantic = bfs(atlanticStarts)

    val result = mutableListOf<List<Int>>()
    for (r in 0 until rows) {
        for (c in 0 until cols) {
            if (pacific[r][c] && atlantic[r][c]) result.add(listOf(r, c))
        }
    }

    return result
}
```

Uses the multi-source BFS pattern directly (all border cells seeded at
once), matching the style established in Walls And Gates and Rotting
Oranges.

---

## Why Reversing the Direction of Flow Is the Critical Insight

This is the trickiest conceptual leap in the problem. Forward simulation
("starting at this cell, where does water actually go?") would require
checking, for every single cell, whether some downhill-or-equal path
exists all the way to a border — extremely expensive to do per-cell.

Instead, we ask the **reverse** question from each ocean's perspective:
"if I'm standing at the ocean's edge, which cells could have drained
into me?" That's equivalent to flowing **uphill** (or staying level)
from the border inward:

```kotlin
if (... && heights[nr][nc] >= heights[r][c]) {
    dfs(nr, nc, visited)
}
// We're moving from a LOWER (or equal) cell to a HIGHER (or equal) one —
// this is the reverse of water's actual downhill flow direction
```

A cell `X` is reachable in this reversed search from the Pacific if and
only if water could *actually* flow from `X` to the Pacific in the
forward direction — the reversal doesn't change reachability, it just
lets us run **one** flood-fill per ocean (starting from all border cells
at once) instead of needing a separate, expensive check from every
single cell in the grid.

**Why running this from every border cell of an ocean simultaneously is
multi-source flood-fill, exactly like Walls And Gates:** instead of
"all gates," we have "all Pacific border cells" (or "all Atlantic border
cells") as the multiple simultaneous starting points for a single
combined search.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| DFS per border cell (Approach 1) | Simpler to write, recursion handles the traversal naturally |
| BFS multi-source (Approach 2) | Prefer iterative style consistent with Walls And Gates / Rotting Oranges |

---

## Complexity

| | |
|---|---|
| **Time** | O(rows × cols) — each ocean's flood-fill visits every cell at most once |
| **Space** | O(rows × cols) — the two visited grids, plus recursion/queue space |

---

## Key Takeaway

> When a problem asks "can X reach a target" for every possible X, and
> checking forward from each X individually would be too slow, consider
> **reversing the direction** and flood-filling backward from the
> target(s) instead — this collapses what looks like an
> O(n) × (expensive per-cell check) problem into a constant number of
> O(V+E) flood-fills. Combined with multi-source seeding (one combined
> search per ocean, not one per border cell), this is the same "search
> from the boundary inward" idea seen in Walls And Gates, applied with an
> added twist: reversing the traversal's direction relative to the
> problem's natural forward logic.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/pacific-atlantic-water-flow/)

---

{% include kotlin-dsa-nav.html %}
