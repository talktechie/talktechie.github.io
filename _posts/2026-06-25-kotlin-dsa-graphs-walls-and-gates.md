---
title: "Graphs: Walls And Gates — Kotlin Solution"
date: 2026-06-25
categories: [Kotlin DSA, Graphs]
tags: [walls-and-gates, graph, bfs, matrix, multi-source-bfs, medium, leetcode, leetcode-286]
leetcode_number: 286
leetcode_url: https://leetcode.com/problems/walls-and-gates/
difficulty: Medium
permalink: /posts/kotlin-dsa-graphs-walls-and-gates/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [286 — Walls And Gates](https://leetcode.com/problems/walls-and-gates/) |
| **Difficulty** | Medium |
| **Topic** | Graph, BFS, Matrix, Multi-Source BFS |

> You are given an `m x n` grid with three kinds of cells: `-1` (a wall),
> `0` (a gate), and `INF` (`2147483647`, an empty room). Fill every empty
> room with the distance to its **nearest** gate. If a room cannot reach
> any gate, leave it as `INF`.

**Example:**
```
Input:
[[INF,-1,0,INF],
 [INF,INF,INF,-1],
 [INF,-1,INF,-1],
 [0,-1,INF,INF]]
Output:
[[3,-1,0,1],
 [2,2,1,-1],
 [1,-1,2,-1],
 [0,-1,3,4]]
```

**Constraints:**
- `m == rooms.length`, `n == rooms[i].length`
- `1 <= m, n <= 250`
- `rooms[i][j]` is `-1`, `0`, or `2³¹ - 1`

---

## Approach

A naive approach — BFS from **every** empty room outward looking for the
nearest gate — would be O((rows×cols)²) in the worst case, far too slow.

**Key insight — Multi-Source BFS, run from ALL gates simultaneously:**
instead of searching outward from each room, search outward from **every
gate at once**, in a single combined BFS. Push all gates into the queue
as starting points (distance 0), then expand level by level — by BFS's
nature, the **first** time any room is reached, it's guaranteed to be via
the **shortest** path from *some* gate, since all gates start
"simultaneously" at distance 0.

**Walk through a small grid:**
```
Initial queue: all gates, e.g. positions (0,2) and (3,0), both distance 0

Level 1 (distance 1 from nearest gate): expand neighbors of all gates
  (0,2)'s empty neighbors: (0,3) → set to 1, enqueue
  (3,0)'s empty neighbors: (2,0) → set to 1, enqueue
  ... (walls at -1 are skipped, already-visited cells skipped)

Level 2 (distance 2): expand neighbors of everything just set to 1
  (0,3)'s neighbors: none new (already visited or out of bounds)
  (2,0)'s neighbors: (1,0) → set to 2, enqueue

... continues until queue is empty ...

Every reachable room now holds its shortest distance to ANY gate ✓
```

---

## Kotlin Solution

### Approach 1 — Multi-source BFS, all gates as initial sources (optimal)

```kotlin
fun wallsAndGates(rooms: Array<IntArray>) {
    val rows = rooms.size
    val cols = rooms[0].size
    val INF = Int.MAX_VALUE

    val queue: ArrayDeque<Pair<Int, Int>> = ArrayDeque()

    // Seed the queue with EVERY gate simultaneously
    for (r in 0 until rows) {
        for (c in 0 until cols) {
            if (rooms[r][c] == 0) queue.addLast(r to c)
        }
    }

    val directions = listOf(0 to 1, 0 to -1, 1 to 0, -1 to 0)

    while (queue.isNotEmpty()) {
        val (r, c) = queue.removeFirst()

        for ((dr, dc) in directions) {
            val nr = r + dr
            val nc = c + dc

            if (nr in 0 until rows && nc in 0 until cols && rooms[nr][nc] == INF) {
                rooms[nr][nc] = rooms[r][c] + 1
                queue.addLast(nr to nc)
            }
        }
    }
}
```

### Approach 2 — Single-source BFS from each gate independently (naive, slower)

```kotlin
fun wallsAndGates(rooms: Array<IntArray>) {
    val rows = rooms.size
    val cols = rooms[0].size
    val INF = Int.MAX_VALUE
    val directions = listOf(0 to 1, 0 to -1, 1 to 0, -1 to 0)

    fun bfsFromGate(startR: Int, startC: Int) {
        val queue: ArrayDeque<Pair<Int, Int>> = ArrayDeque()
        queue.addLast(startR to startC)
        val dist = Array(rows) { IntArray(cols) { -1 } }
        dist[startR][startC] = 0

        while (queue.isNotEmpty()) {
            val (r, c) = queue.removeFirst()
            for ((dr, dc) in directions) {
                val nr = r + dr; val nc = c + dc
                if (nr in 0 until rows && nc in 0 until cols &&
                    rooms[nr][nc] != -1 && dist[nr][nc] == -1) {
                    dist[nr][nc] = dist[r][c] + 1
                    if (rooms[nr][nc] > dist[nr][nc]) rooms[nr][nc] = dist[nr][nc]
                    queue.addLast(nr to nc)
                }
            }
        }
    }

    for (r in 0 until rows) {
        for (c in 0 until cols) {
            if (rooms[r][c] == 0) bfsFromGate(r, c)
        }
    }
}
```

Runs a full, separate BFS from every gate, comparing and keeping the
minimum distance found across all of them — correct, but redundantly
re-explores large portions of the grid once per gate, giving
O(gates × rows × cols) in the worst case instead of Approach 1's single
O(rows × cols) pass.

---

## Why Starting BFS From All Gates Simultaneously Guarantees Shortest Distances

This is the single most important insight in the problem, and it
generalizes far beyond this specific grid scenario.

```kotlin
for (r in 0 until rows) {
    for (c in 0 until cols) {
        if (rooms[r][c] == 0) queue.addLast(r to c)   // ALL gates start at distance 0
    }
}
```

In ordinary single-source BFS, level `k` of the traversal contains every
node exactly `k` steps from the **one** source. With **multiple** sources
seeded at the same starting level (distance 0), level `k` now contains
every node exactly `k` steps from its **nearest** source — because BFS
explores breadth-first, a room can only be reached at a smaller distance
through whichever gate path gets there first, and since all gates start
simultaneously, the BFS naturally finds the globally shortest distance
for every room in a single combined pass.

**Why we only enqueue/update cells that are still `INF`:**
```kotlin
if (... && rooms[nr][nc] == INF) {
    rooms[nr][nc] = rooms[r][c] + 1
    queue.addLast(nr to nc)
}
```
Once a room has been assigned a distance, BFS guarantees that's already
its shortest possible distance (since we process strictly increasing
distances level by level) — there's no need to ever revisit or update it
again.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Multi-source BFS (Approach 1) | Always — O(rows × cols) total, the correct optimal solution |
| Single-source BFS per gate (Approach 2) | Never for submission; included only to highlight the inefficiency it avoids |

---

## Complexity

| | Multi-Source BFS | Single-Source per Gate |
|---|---|---|
| **Time** | O(rows × cols) | O(gates × rows × cols) |
| **Space** | O(rows × cols) — queue | O(rows × cols) per gate's BFS |

---

## Key Takeaway

> Whenever a problem asks for "distance to the nearest of several
> targets" across a grid (or any graph), **multi-source BFS** — seeding
> the queue with all targets simultaneously rather than running separate
> searches — collapses what looks like a multi-pass problem into a
> single O(V+E) traversal. This technique appears again almost
> immediately in Rotting Oranges, which uses the exact same
> "seed with multiple starting points, expand level by level" idea.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/walls-and-gates/)

---

{% include kotlin-dsa-nav.html %}
