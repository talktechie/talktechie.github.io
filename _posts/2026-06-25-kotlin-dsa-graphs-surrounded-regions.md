---
title: "Graphs: Surrounded Regions — Kotlin Solution"
date: 2026-06-25
categories: [Kotlin DSA, Graphs]
tags: [surrounded-regions, graph, dfs, bfs, matrix, medium, leetcode, leetcode-130]
leetcode_number: 130
leetcode_url: https://leetcode.com/problems/surrounded-regions/
difficulty: Medium
permalink: /posts/kotlin-dsa-graphs-surrounded-regions/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [130 — Surrounded Regions](https://leetcode.com/problems/surrounded-regions/) |
| **Difficulty** | Medium |
| **Topic** | Graph, DFS, BFS, Matrix |

> Given an `m x n` matrix `board` of `'X'` and `'O'`, **capture** every
> region of `'O'`s that is **completely surrounded** by `'X'`s (no path
> to the border) by flipping them to `'X'`. Regions connected to the
> border are safe and remain `'O'`.

**Example:**
```
Input:
[["X","X","X","X"],
 ["X","O","O","X"],
 ["X","X","O","X"],
 ["X","O","X","X"]]
Output:
[["X","X","X","X"],
 ["X","X","X","X"],
 ["X","X","X","X"],
 ["X","O","X","X"]]
Explanation: the middle region of O's is fully surrounded and gets
             captured; the bottom-left O touches the border and survives
```

**Constraints:**
- `m == board.length`, `n == board[i].length`
- `1 <= m, n <= 200`
- `board[i][j]` is `'X'` or `'O'`

---

## Approach

The naive idea — for every `'O'`, check if it can reach the border —
would mean redoing a search from scratch for every cell, which is slow
and redundant. The trick here is the same **reversal** idea as Pacific
Atlantic Water Flow: instead of checking each region individually, work
backward from the border.

**Key insight:** Any `'O'` connected (directly or transitively) to the
**border** can never be captured — water (or in this case, freedom from
capture) "escapes" through the border. So: flood-fill from every
border `'O'`, marking all border-connected `'O'`s as **safe**. Anything
left over — every `'O'` that was *not* marked safe — must be fully
surrounded, and gets flipped to `'X'`.

**Walk through the example grid:**
```
Border cells with 'O': only (3,1) touches the border (bottom row)

Flood-fill from (3,1): mark it safe. Check its neighbors —
  (2,1)='X' → skip, (3,0)='X' → skip, (3,2)='X' → skip
  No connected 'O's beyond (3,1) itself.

Safe set: {(3,1)}

Now scan the entire board: any 'O' NOT in the safe set gets flipped to 'X'
  (1,1), (1,2), (2,2) → all flipped to 'X' (they were never connected
  to the border)
  (3,1) → stays 'O' (it's in the safe set)

Result matches expected output ✓
```

---

## Kotlin Solution

### Approach 1 — Flood-fill from border O's, mark safe, flip everything else (optimal)

```kotlin
fun solve(board: Array<CharArray>) {
    val rows = board.size
    val cols = board[0].size

    fun dfs(r: Int, c: Int) {
        if (r < 0 || r >= rows || c < 0 || c >= cols || board[r][c] != 'O') return

        board[r][c] = 'S'   // mark as "safe" temporarily

        dfs(r + 1, c)
        dfs(r - 1, c)
        dfs(r, c + 1)
        dfs(r, c - 1)
    }

    // Flood-fill from every border cell
    for (r in 0 until rows) {
        dfs(r, 0)
        dfs(r, cols - 1)
    }
    for (c in 0 until cols) {
        dfs(0, c)
        dfs(rows - 1, c)
    }

    // Anything still 'O' was never connected to the border — capture it
    // Anything marked 'S' was safe — restore it back to 'O'
    for (r in 0 until rows) {
        for (c in 0 until cols) {
            when (board[r][c]) {
                'O' -> board[r][c] = 'X'
                'S' -> board[r][c] = 'O'
            }
        }
    }
}
```

### Approach 2 — BFS from border O's instead of DFS

```kotlin
fun solve(board: Array<CharArray>) {
    val rows = board.size
    val cols = board[0].size
    val queue: ArrayDeque<Pair<Int, Int>> = ArrayDeque()

    fun trySafe(r: Int, c: Int) {
        if (r in 0 until rows && c in 0 until cols && board[r][c] == 'O') {
            board[r][c] = 'S'
            queue.addLast(r to c)
        }
    }

    for (r in 0 until rows) { trySafe(r, 0); trySafe(r, cols - 1) }
    for (c in 0 until cols) { trySafe(0, c); trySafe(rows - 1, c) }

    val directions = listOf(0 to 1, 0 to -1, 1 to 0, -1 to 0)

    while (queue.isNotEmpty()) {
        val (r, c) = queue.removeFirst()
        for ((dr, dc) in directions) {
            trySafe(r + dr, c + dc)
        }
    }

    for (r in 0 until rows) {
        for (c in 0 until cols) {
            when (board[r][c]) {
                'O' -> board[r][c] = 'X'
                'S' -> board[r][c] = 'O'
            }
        }
    }
}
```

---

## Why the Temporary `'S'` Marker Is Necessary

This is a subtle but important detail: we need a **third** state beyond
just `'O'` and `'X'`, because the final transformation rule depends on
**which** cells were border-connected, and a simple boolean `visited`
array would work too — but using a sentinel character directly in the
board avoids needing extra space:

```kotlin
board[r][c] = 'S'   // temporarily mark border-connected O's distinctly
// ...
when (board[r][c]) {
    'O' -> board[r][c] = 'X'   // never reached the border — captured
    'S' -> board[r][c] = 'O'   // reached the border — restore as-is
}
```

Without a distinct marker, once we start flipping cells we'd lose the
information about which `'O'`s were originally border-connected versus
originally surrounded — both would just look like `'O'` during the
flood-fill itself.

**Why flood-filling FROM the border outward-inward, rather than checking
each region's reachability to the border, avoids redundant work:** this
is the exact same reversal insight as Pacific Atlantic Water Flow — a
single combined traversal starting from all border cells replaces what
would otherwise require a separate reachability check per region.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| DFS with sentinel marker (Approach 1) | Simpler, reuses the board itself as the visited-tracking structure |
| BFS with sentinel marker (Approach 2) | Prefer iterative traversal, avoiding deep recursion on large boards |

---

## Complexity

| | |
|---|---|
| **Time** | O(rows × cols) — every cell visited at most once during the flood-fill, plus one final pass |
| **Space** | O(rows × cols) worst case — recursion/queue depth |

---

## Key Takeaway

> "Region NOT connected to a boundary gets captured/removed" is solved
> by reversing the search direction: flood-fill **from** the boundary
> inward, marking everything reachable as safe, then treating everything
> unreached as the answer. This is conceptually identical to Pacific
> Atlantic Water Flow's border-outward reversal trick, just applied to a
> capture/elimination rule instead of a reachability-to-multiple-targets
> rule — reinforcing how often "search from the edges inward" outperforms
> "search from the inside outward" whenever boundary conditions define
> the actual rule.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/surrounded-regions/)

---

{% include kotlin-dsa-nav.html %}
