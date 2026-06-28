---
title: "Advanced Graphs: Swim In Rising Water — Kotlin Solution"
date: 2026-06-28
categories: [Kotlin DSA, Advanced Graphs]
tags: [swim-in-rising-water, graph, heap, dijkstra, binary-search, matrix, hard, leetcode, leetcode-778]
leetcode_number: 778
leetcode_url: https://leetcode.com/problems/swimming-rising-water/
difficulty: Hard
permalink: /posts/kotlin-dsa-advanced-graphs-swim-in-rising-water/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [778 — Swim In Rising Water](https://leetcode.com/problems/swim-in-rising-water/) |
| **Difficulty** | Hard |
| **Topic** | Graph, Heap, Dijkstra-style, Binary Search, Matrix |

> Given an `n x n` grid where `grid[i][j]` is the elevation at that
> cell, water rises over time `t` (at time `t`, all cells with elevation
> `<= t` are submerged/swimmable). You can swim between adjacent cells
> only if both cells are currently submerged. Return the **minimum time**
> `t` such that you can swim from `(0,0)` to `(n-1,n-1)`.

**Example:**
```
Input:  grid = [[0,2],[1,3]]
Output: 3
Explanation: at t=3, every cell (0,1,2,3) is submerged — you can swim
             the whole grid

Input:  grid = [[0,1,2,3,4],[24,23,22,21,5],[12,13,14,15,16],
                [11,17,18,19,20],[10,9,8,7,6]]
Output: 16
```

**Constraints:**
- `n == grid.length == grid[i].length`
- `1 <= n <= 50`
- `0 <= grid[i][j] < n²`
- All values in `grid` are unique

---

## Approach

This is a "minimize the maximum value along a path" problem —
structurally similar to Dijkstra's, but instead of **summing** edge
weights along a path, we care about the **maximum** elevation
encountered along the way (since you must wait for the highest point on
your route to submerge before passing through it).

**Key insight — modified Dijkstra's, tracking the minimum possible
"path maximum" instead of path sum:** use a min-heap keyed by the
highest elevation encountered so far on any path to that cell. Greedily
expand the cell with the **smallest** such "bottleneck" value next —
exactly mirroring Dijkstra's frontier-expansion greedy choice, just with
`max` replacing `+` as the way costs combine along a path.

**Walk through `grid = [[0,2],[1,3]]`:**
```
heap: [(0, (0,0))]   (starting cell, elevation 0)

pop (0, (0,0)): visit. neighbors: (0,1)=2, (1,0)=1
  push (max(0,2)=2, (0,1))
  push (max(0,1)=1, (1,0))

pop (1, (1,0)): visit. neighbor: (1,1)=3
  push (max(1,3)=3, (1,1))

pop (2, (0,1)): visit. neighbor: (1,1)=3
  push (max(2,3)=3, (1,1))  (already a candidate, but heap handles duplicates fine)

pop (3, (1,1)): this is the destination! return 3 ✓
```

---

## Kotlin Solution

### Approach 1 — Modified Dijkstra's, min-heap tracks path maximum (optimal)

```kotlin
fun swimInWater(grid: Array<IntArray>): Int {
    val n = grid.size
    val visited = Array(n) { BooleanArray(n) }
    val heap = PriorityQueue<IntArray>(compareBy { it[0] })  // [maxElevationSoFar, row, col]
    heap.add(intArrayOf(grid[0][0], 0, 0))
    visited[0][0] = true

    val directions = listOf(0 to 1, 0 to -1, 1 to 0, -1 to 0)

    while (heap.isNotEmpty()) {
        val (maxElev, r, c) = heap.poll()

        if (r == n - 1 && c == n - 1) return maxElev   // reached the destination

        for ((dr, dc) in directions) {
            val nr = r + dr
            val nc = c + dc
            if (nr in 0 until n && nc in 0 until n && !visited[nr][nc]) {
                visited[nr][nc] = true
                heap.add(intArrayOf(maxOf(maxElev, grid[nr][nc]), nr, nc))
            }
        }
    }

    return -1   // unreachable — shouldn't happen given problem guarantees
}
```

### Approach 2 — Binary search on the answer + BFS/DFS feasibility check

```kotlin
fun swimInWater(grid: Array<IntArray>): Int {
    val n = grid.size

    fun canReachEnd(t: Int): Boolean {
        if (grid[0][0] > t) return false

        val visited = Array(n) { BooleanArray(n) }
        val queue: ArrayDeque<Pair<Int, Int>> = ArrayDeque()
        queue.addLast(0 to 0)
        visited[0][0] = true

        val directions = listOf(0 to 1, 0 to -1, 1 to 0, -1 to 0)

        while (queue.isNotEmpty()) {
            val (r, c) = queue.removeFirst()
            if (r == n - 1 && c == n - 1) return true

            for ((dr, dc) in directions) {
                val nr = r + dr
                val nc = c + dc
                if (nr in 0 until n && nc in 0 until n && !visited[nr][nc] && grid[nr][nc] <= t) {
                    visited[nr][nc] = true
                    queue.addLast(nr to nc)
                }
            }
        }

        return false
    }

    var low = grid[0][0]
    var high = n * n - 1

    while (low < high) {
        val mid = low + (high - low) / 2
        if (canReachEnd(mid)) high = mid else low = mid + 1
    }

    return low
}
```

Reframes the problem as **binary search on the answer time `t`**: "can
we reach the end if water has risen to exactly `t`?" is a simple
BFS/DFS feasibility check, and that feasibility is monotonic in `t` (if
reachable at some `t`, also reachable at any larger `t`) — exactly the
condition that makes binary search valid, the same "binary search on
answer space" pattern from Koko Eating Bananas in the Binary Search
phase.

---

## Why Using `max` Instead of `+` Still Preserves Dijkstra's Greedy Correctness

This is the conceptual leap that generalizes Dijkstra's beyond pure
summed-weight shortest paths:

```kotlin
heap.add(intArrayOf(maxOf(maxElev, grid[nr][nc]), nr, nc))
// NOT: maxElev + grid[nr][nc]
```

Dijkstra's correctness relies on the **combining operation** being
*monotonic* — extending a path can never make its "cost" smaller. Both
`+` (with non-negative weights) and `max` (with any real-valued
elevations) satisfy this: adding more cells to a path can only keep the
running maximum the same or increase it, never decrease it. Because this
monotonicity holds, the same greedy argument applies — once a cell is
popped from the heap with its true minimum possible "path maximum," that
value can never be improved upon later.

**Why binary search (Approach 2) works as a completely different but
equally valid strategy:** "can we reach the destination by time `t`" is
a **monotonic** predicate in `t` — if reachable at some `t`, it's
trivially still reachable at any `t' > t` (more cells become available,
never fewer). Monotonic predicates are exactly the precondition for
binary search to correctly find the threshold value.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Modified Dijkstra's (Approach 1) | More direct, single pass, the most natural generalization of Dijkstra's |
| Binary search + BFS (Approach 2) | Want to practice recognizing monotonic feasibility predicates; same overall complexity |

---

## Complexity

| | Modified Dijkstra's | Binary Search + BFS |
|---|---|---|
| **Time** | O(n² log n) | O(n² log(n²)) — log(max elevation) search iterations, each O(n²) BFS |
| **Space** | O(n²) | O(n²) |

---

## Key Takeaway

> Dijkstra's Algorithm generalizes beyond simple weighted-sum shortest
> paths to **any** monotonic path-combining operation — here, `max`
> instead of `+`. Recognizing that the same greedy frontier-expansion
> logic still applies whenever combining costs along a path can only
> stay the same or grow is a powerful generalization. The binary-search
> alternative is equally instructive: it reframes the problem entirely,
> exploiting the fact that "reachable by time `t`" is monotonic, directly
> connecting back to the binary-search-on-answer-space technique from
> the Binary Search phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/swimming-rising-water/)

---

{% include kotlin-dsa-nav.html %}
