---
title: "Advanced Graphs: Min Cost to Connect All Points — Kotlin Solution"
date: 2026-06-28
categories: [Kotlin DSA, Advanced Graphs]
tags: [min-cost-to-connect-all-points, graph, mst, prim, union-find, heap, medium, leetcode, leetcode-1584]
leetcode_number: 1584
leetcode_url: https://leetcode.com/problems/min-cost-to-connect-all-points/
difficulty: Medium
permalink: /posts/kotlin-dsa-advanced-graphs-min-cost-to-connect-all-points/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [1584 — Min Cost to Connect All Points](https://leetcode.com/problems/min-cost-to-connect-all-points/) |
| **Difficulty** | Medium |
| **Topic** | Graph, Minimum Spanning Tree, Prim's Algorithm, Union-Find, Heap |

> Given `points` on a 2D plane, the cost to connect two points is their
> **Manhattan distance**. Return the **minimum cost** to connect all
> points such that there's a path between every pair (directly or
> transitively).

**Example:**
```
Input:  points = [[0,0],[2,2],[3,10],[5,2],[7,0]]
Output: 20

Input:  points = [[3,12],[-2,5],[-4,1]]
Output: 18
```

**Constraints:**
- `1 <= points.length <= 1000`
- `-10⁶ <= xi, yi <= 10⁶`
- All pairs of points are distinct

---

## Approach

This introduces **Minimum Spanning Tree (MST)** — the problem of
connecting every node in a graph with the minimum total edge weight,
forming a tree (no cycles) that touches every node exactly once via
`n-1` edges.

**Key insight (Prim's Algorithm):** Start from any single point. Greedily
grow a connected tree one point at a time, always adding the
**cheapest** edge that connects a point **already in the tree** to a
point **not yet in the tree**. This mirrors Dijkstra's "always expand
the closest frontier" greedy choice, but optimizes for the cheapest
**individual edge** added, not the cheapest **cumulative path** from a
fixed source.

**Walk through a simplified trace conceptually:** starting from point 0,
maintain a min-heap of `(cost, point)` representing the cheapest known
edge connecting each unvisited point to the current tree. Repeatedly pop
the cheapest, add its cost to the total, mark it visited, and update the
heap with any newly-discovered cheaper connections from this point to
its still-unvisited neighbors.

---

## Kotlin Solution

### Approach 1 — Prim's Algorithm with a min-heap (optimal for dense graphs)

```kotlin
fun minCostConnectPoints(points: Array<IntArray>): Int {
    val n = points.size
    if (n <= 1) return 0

    fun manhattan(a: IntArray, b: IntArray) = Math.abs(a[0] - b[0]) + Math.abs(a[1] - b[1])

    val visited = BooleanArray(n)
    val heap = PriorityQueue<IntArray>(compareBy { it[0] })  // [cost, pointIndex]
    heap.add(intArrayOf(0, 0))   // start from point 0, cost 0 to "connect" itself

    var totalCost = 0
    var edgesUsed = 0

    while (edgesUsed < n) {
        val (cost, point) = heap.poll()

        if (visited[point]) continue   // already connected via a cheaper edge earlier

        visited[point] = true
        totalCost += cost
        edgesUsed++

        for (next in 0 until n) {
            if (!visited[next]) {
                heap.add(intArrayOf(manhattan(points[point], points[next]), next))
            }
        }
    }

    return totalCost
}
```

### Approach 2 — Kruskal's Algorithm with Union-Find (alternative MST approach)

```kotlin
fun minCostConnectPoints(points: Array<IntArray>): Int {
    val n = points.size
    if (n <= 1) return 0

    fun manhattan(a: IntArray, b: IntArray) = Math.abs(a[0] - b[0]) + Math.abs(a[1] - b[1])

    // Generate every possible edge, sorted by cost
    val edges = mutableListOf<IntArray>()   // [cost, u, v]
    for (i in 0 until n) {
        for (j in i + 1 until n) {
            edges.add(intArrayOf(manhattan(points[i], points[j]), i, j))
        }
    }
    edges.sortBy { it[0] }

    val parent = IntArray(n) { it }
    fun find(x: Int): Int {
        var root = x
        while (parent[root] != root) root = parent[root]
        parent[x] = root
        return root
    }

    var totalCost = 0
    var edgesUsed = 0

    for ((cost, u, v) in edges) {
        val rootU = find(u)
        val rootV = find(v)
        if (rootU != rootV) {
            parent[rootU] = rootV
            totalCost += cost
            edgesUsed++
            if (edgesUsed == n - 1) break   // MST complete — exactly n-1 edges needed
        }
    }

    return totalCost
}
```

Generates **all** O(n²) possible edges upfront, sorts them by cost, and
greedily adds the cheapest edge that doesn't create a cycle (checked via
Union-Find) — the classic Kruskal's MST approach, directly reusing the
Union-Find technique from the earlier Graphs phase.

---

## Why Prim's Is Generally Preferred Here (Dense Graphs)

This problem is a **complete graph** — every pair of points has a
direct edge (their Manhattan distance) — meaning there are O(n²) total
possible edges. This density is the deciding factor between the two MST
algorithms:

```kotlin
// Prim's: grows the heap by checking edges FROM the tree TO unvisited
// points only — never needs to enumerate or sort the full O(n²) edge list
for (next in 0 until n) {
    if (!visited[next]) {
        heap.add(intArrayOf(manhattan(points[point], points[next]), next))
    }
}
```

Kruskal's (Approach 2) requires generating and sorting **all** O(n²)
edges upfront — for `n = 1000` points, that's roughly 500,000 edges to
sort, an O(n² log n²) cost. Prim's, by contrast, never materializes the
full edge list; it only ever considers edges from already-visited points
outward, giving roughly O(n²) total heap operations without the sorting
overhead — meaningfully better for dense graphs like this one.

**Why the stale-entry check (`if (visited[point]) continue`) appears here
too:** exactly the same reasoning as Dijkstra's — a point might be
pushed onto the heap multiple times with different costs as the tree
grows, and the first time it's popped (with the smallest such cost) is
the one that matters; later, larger-cost entries for the same
already-visited point are simply skipped.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Prim's (Approach 1) | Dense graphs (like this problem, a complete graph) — avoids enumerating all O(n²) edges upfront |
| Kruskal's (Approach 2) | Sparse graphs with relatively few edges compared to nodes — sorting a smaller edge list is cheaper |

---

## Complexity

| | Prim's | Kruskal's |
|---|---|---|
| **Time** | O(n² log n) | O(n² log n²) — dominated by sorting all edges |
| **Space** | O(n²) — heap can hold many entries | O(n²) — storing all edges |

---

## Key Takeaway

> Minimum Spanning Tree problems generalize "connect everything as
> cheaply as possible" beyond simple pairwise distances. Prim's
> Algorithm grows a tree greedily from a single starting point, always
> adding the cheapest edge connecting the tree to something new —
> conceptually similar to Dijkstra's frontier expansion, but optimizing
> per-edge cost rather than cumulative path cost. Kruskal's takes the
> opposite approach, sorting all edges globally and using Union-Find to
> avoid cycles. Choosing between them comes down to graph density: Prim's
> wins on dense graphs (like this complete-graph problem), Kruskal's
> wins on sparse ones.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/min-cost-to-connect-all-points/)

---

{% include kotlin-dsa-nav.html %}
