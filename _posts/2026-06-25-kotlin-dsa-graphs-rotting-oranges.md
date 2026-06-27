---
title: "Graphs: Rotting Oranges — Kotlin Solution"
date: 2026-06-25
categories: [Kotlin DSA, Graphs]
tags: [rotting-oranges, graph, bfs, matrix, multi-source-bfs, medium, leetcode, leetcode-994]
leetcode_number: 994
leetcode_url: https://leetcode.com/problems/rotting-oranges/
difficulty: Medium
permalink: /posts/kotlin-dsa-graphs-rotting-oranges/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [994 — Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) |
| **Difficulty** | Medium |
| **Topic** | Graph, BFS, Matrix, Multi-Source BFS |

> Given a grid where `0` = empty, `1` = fresh orange, `2` = rotten
> orange. Every minute, any fresh orange **adjacent** to a rotten one
> becomes rotten too. Return the minimum number of minutes until no
> fresh orange remains, or `-1` if that's impossible.

**Example:**
```
Input:  [[2,1,1],[1,1,0],[0,1,1]]
Output: 4

Input:  [[2,1,1],[0,1,1],[1,0,1]]
Output: -1
Explanation: the orange in the bottom-left corner is never reached
             because it's isolated by zeros

Input:  [[0,2]]
Output: 0
```

**Constraints:**
- `m == grid.length`, `n == grid[i].length`
- `1 <= m, n <= 10`
- `grid[i][j]` is `0`, `1`, or `2`

---

## Approach

This is multi-source BFS applied to a problem that's *explicitly about
time*, rather than distance — but it's the exact same underlying
technique as Walls And Gates.

**Key insight:** Seed the BFS queue with **every** initially rotten
orange simultaneously. Each level of BFS expansion corresponds to exactly
**one minute** passing — every fresh orange reached during level `k` of
the BFS becomes rotten at minute `k`. The answer is simply the number of
levels processed before the queue empties (assuming all fresh oranges
were eventually reached).

**Walk through `[[2,1,1],[1,1,0],[0,1,1]]`:**
```
Initial queue: [(0,0)] (the one rotten orange), minute=0
fresh count = 6

Minute 1: process (0,0) → rot neighbors (0,1) and (1,0)
  queue: [(0,1),(1,0)]   fresh count: 4

Minute 2: process (0,1) → rot (0,2); process (1,0) → rot (1,1)
  queue: [(0,2),(1,1)]   fresh count: 2

Minute 3: process (0,2) → no new fresh neighbors; process (1,1) → rot (2,1)
  queue: [(2,1)]   fresh count: 1

Minute 4: process (2,1) → rot (2,2)
  queue: [(2,2)]   fresh count: 0

Minute 5: process (2,2) → no new neighbors, queue empties

Total minutes elapsed: 4 (the last minute that caused a rot) ✓
```

---

## Kotlin Solution

### Approach 1 — Multi-source BFS, track minutes via level count (optimal)

```kotlin
fun orangesRotting(grid: Array<IntArray>): Int {
    val rows = grid.size
    val cols = grid[0].size
    val queue: ArrayDeque<Pair<Int, Int>> = ArrayDeque()
    var freshCount = 0

    for (r in 0 until rows) {
        for (c in 0 until cols) {
            when (grid[r][c]) {
                2 -> queue.addLast(r to c)
                1 -> freshCount++
            }
        }
    }

    if (freshCount == 0) return 0   // nothing to rot — already done

    val directions = listOf(0 to 1, 0 to -1, 1 to 0, -1 to 0)
    var minutes = 0

    while (queue.isNotEmpty() && freshCount > 0) {
        minutes++
        val levelSize = queue.size

        repeat(levelSize) {
            val (r, c) = queue.removeFirst()

            for ((dr, dc) in directions) {
                val nr = r + dr
                val nc = c + dc
                if (nr in 0 until rows && nc in 0 until cols && grid[nr][nc] == 1) {
                    grid[nr][nc] = 2
                    freshCount--
                    queue.addLast(nr to nc)
                }
            }
        }
    }

    return if (freshCount == 0) minutes else -1
}
```

### Approach 2 — Track minute alongside each queue entry (no level-size batching)

```kotlin
fun orangesRotting(grid: Array<IntArray>): Int {
    val rows = grid.size
    val cols = grid[0].size
    val queue: ArrayDeque<Triple<Int, Int, Int>> = ArrayDeque()  // (row, col, minute)
    var freshCount = 0
    var maxMinute = 0

    for (r in 0 until rows) {
        for (c in 0 until cols) {
            when (grid[r][c]) {
                2 -> queue.addLast(Triple(r, c, 0))
                1 -> freshCount++
            }
        }
    }

    val directions = listOf(0 to 1, 0 to -1, 1 to 0, -1 to 0)

    while (queue.isNotEmpty()) {
        val (r, c, minute) = queue.removeFirst()
        maxMinute = maxOf(maxMinute, minute)

        for ((dr, dc) in directions) {
            val nr = r + dr
            val nc = c + dc
            if (nr in 0 until rows && nc in 0 until cols && grid[nr][nc] == 1) {
                grid[nr][nc] = 2
                freshCount--
                queue.addLast(Triple(nr, nc, minute + 1))
            }
        }
    }

    return if (freshCount == 0) maxMinute else -1
}
```

Stores the minute explicitly with each queue entry instead of relying on
level-by-level batching — slightly more memory per entry, but avoids
needing the `levelSize` snapshot trick.

---

## Why Checking `freshCount == 0` at the End Detects Unreachable Oranges

```kotlin
return if (freshCount == 0) minutes else -1
```

If any fresh orange is isolated (surrounded by empty cells with no path
to any rotten orange), the BFS queue will empty out **before**
`freshCount` reaches zero — those oranges are simply never enqueued,
because they have no rotten neighbor to trigger their rotting. Checking
`freshCount` after the loop tells us definitively whether every orange
was actually reached.

**Why the level-size snapshot pattern (Approach 1) matches "minutes" so
naturally:** this is the exact same technique as Binary Tree Level Order
Traversal from the Trees phase — `levelSize` captures exactly which
oranges existed in the queue **before** this minute's expansion, ensuring
all of them (and only them) get processed as "this minute's batch,"
while their resulting newly-rotten neighbors correctly wait for the next
minute's iteration.

```kotlin
val levelSize = queue.size   // exactly the oranges rotten as of the PREVIOUS minute
repeat(levelSize) { ... }     // process exactly those, queue any newly-rotten ones
                                // for the NEXT iteration of the outer while loop
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Level-size batching (Approach 1) | Cleaner mental model, directly reuses the BFS-level-tracking pattern from Trees |
| Per-entry minute tracking (Approach 2) | Avoids the level-size snapshot, marginally more memory per queue entry |

---

## Complexity

| | |
|---|---|
| **Time** | O(rows × cols) — every cell processed at most once |
| **Space** | O(rows × cols) — queue holds at most every cell at some point |

---

## Key Takeaway

> Rotting Oranges is multi-source BFS where each **level** of the
> traversal represents one unit of **time** elapsing, rather than
> spatial distance — the same underlying mechanism as Walls And Gates,
> applied to a "simultaneous spread" scenario instead of a "shortest
> distance" one. The level-size snapshot trick from BFS tree traversals
> reappears here in a graph context, reinforcing that BFS's level-by-level
> structure is useful far beyond trees whenever "steps" or "time units"
> need to be counted precisely.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/rotting-oranges/)

---

{% include kotlin-dsa-nav.html %}
