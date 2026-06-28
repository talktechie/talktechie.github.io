---
title: "Advanced Graphs: Network Delay Time — Kotlin Solution"
date: 2026-06-29
categories: [Kotlin DSA, Advanced Graphs]
tags: [network-delay-time, graph, dijkstra, heap, shortest-path, medium, leetcode, leetcode-743]
leetcode_number: 743
leetcode_url: https://leetcode.com/problems/network-delay-time/
difficulty: Medium
permalink: /posts/kotlin-dsa-advanced-graphs-network-delay-time/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [743 — Network Delay Time](https://leetcode.com/problems/network-delay-time/) |
| **Difficulty** | Medium |
| **Topic** | Graph, Dijkstra's Algorithm, Heap, Shortest Path |

> There are `n` network nodes labeled `1` to `n`. Given `times[i] =
> (u, v, w)` meaning a signal from `u` to `v` takes `w` time, and a
> starting node `k`, return the **time** for the signal to reach
> **all** nodes, or `-1` if impossible.

**Example:**
```
Input:  times = [[2,1,1],[2,3,1],[3,4,1]], n = 4, k = 2
Output: 2
Explanation: from node 2, signal reaches 1 (time 1) and 3 (time 1)
             simultaneously, then 4 from 3 (time 1+1=2) — the LAST
             node to receive the signal determines the total time

Input:  times = [[1,2,1]], n = 2, k = 2
Output: -1
Explanation: node 2 has no outgoing edge to node 1 — node 1 is
             unreachable from node 2
```

**Constraints:**
- `1 <= k <= n <= 100`
- `1 <= times.length <= 6000`
- `0 <= u_i, v_i <= n`, `u_i != v_i`
- `0 <= w_i <= 100`

---

## Approach

This is the entry point to the Advanced Graphs phase, introducing
**Dijkstra's Algorithm** — the standard technique for **shortest path
with weighted edges**, generalizing the unweighted-graph BFS shortest
path technique from the earlier Graphs phase.

**Key insight:** Plain BFS finds shortest paths correctly only when all
edges have **equal weight** (each "step" costs the same). When edges
have different weights, the node reached via the **fewest hops** isn't
necessarily the node reached **fastest**. Dijkstra's fixes this by always
expanding the **currently closest known node next** — using a min-heap
keyed by accumulated distance, not by hop count.

**Walk through `times = [[2,1,1],[2,3,1],[3,4,1]], n = 4, k = 2`:**
```
dist = {1:∞, 2:0, 3:∞, 4:∞}   (node 2 is the source, distance 0)
heap: [(0, node=2)]

pop (0,2): for each edge from 2: (2->1, w=1), (2->3, w=1)
  dist[1] = min(∞, 0+1) = 1 → push (1, node=1)
  dist[3] = min(∞, 0+1) = 1 → push (1, node=3)

pop (1,1): no outgoing edges from 1 → nothing to update

pop (1,3): for edge (3->4, w=1)
  dist[4] = min(∞, 1+1) = 2 → push (2, node=4)

pop (2,4): no outgoing edges → done

Final dist = {1:1, 2:0, 3:1, 4:2}
Max distance across all reachable nodes = 2 ✓ (the signal takes this
long to reach the FARTHEST/last node)
```

---

## Kotlin Solution

### Approach 1 — Dijkstra's Algorithm with a min-heap (optimal)

```kotlin
fun networkDelayTime(times: Array<IntArray>, n: Int, k: Int): Int {
    val graph = Array(n + 1) { mutableListOf<Pair<Int, Int>>() }   // node -> (neighbor, weight)
    for ((u, v, w) in times) {
        graph[u].add(v to w)
    }

    val dist = IntArray(n + 1) { Int.MAX_VALUE }
    dist[k] = 0

    val heap = PriorityQueue<Pair<Int, Int>>(compareBy { it.first })  // (distance, node)
    heap.add(0 to k)

    while (heap.isNotEmpty()) {
        val (d, node) = heap.poll()

        if (d > dist[node]) continue   // stale entry — a shorter path was already found

        for ((neighbor, weight) in graph[node]) {
            val newDist = d + weight
            if (newDist < dist[neighbor]) {
                dist[neighbor] = newDist
                heap.add(newDist to neighbor)
            }
        }
    }

    val maxDist = (1..n).maxOf { dist[it] }
    return if (maxDist == Int.MAX_VALUE) -1 else maxDist
}
```

### Approach 2 — Bellman-Ford (works even with negative weights, slower)

```kotlin
fun networkDelayTime(times: Array<IntArray>, n: Int, k: Int): Int {
    val dist = IntArray(n + 1) { Int.MAX_VALUE }
    dist[k] = 0

    // Relax all edges up to (n-1) times — guaranteed sufficient for
    // shortest paths in a graph with no negative cycles
    repeat(n - 1) {
        for ((u, v, w) in times) {
            if (dist[u] != Int.MAX_VALUE && dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w
            }
        }
    }

    val maxDist = (1..n).maxOf { dist[it] }
    return if (maxDist == Int.MAX_VALUE) -1 else maxDist
}
```

Doesn't require a heap, and correctly handles negative edge weights
(which Dijkstra's cannot) — but runs in O(V × E) instead of Dijkstra's
O(E log V), substantially slower for large graphs.

---

## Why the `if (d > dist[node]) continue` Check Is Necessary

This is a subtle but important detail in heap-based Dijkstra's
implementations:

```kotlin
val (d, node) = heap.poll()
if (d > dist[node]) continue   // stale entry — skip it
```

Because we don't (or can't easily) remove outdated entries from a
standard `PriorityQueue`, a node might be pushed onto the heap **multiple
times** with different distance values, as shorter paths to it are
discovered later. By the time an older, larger-distance entry is popped,
`dist[node]` may have already been updated to something smaller via a
more recent push. Checking this and skipping stale entries avoids
incorrectly re-processing a node using outdated, no-longer-optimal
distance information.

**Why Dijkstra's greedy "always expand the closest known node" choice
is correct:** once a node is popped from the min-heap with its true
shortest distance, that distance can **never** be improved later — any
alternative path to it would have to go through some other node with an
even larger known distance, which (since all edge weights are
non-negative) could only make the total path length larger, not
smaller. This non-negative-weights assumption is exactly why Dijkstra's
fails on graphs with negative edges, and why Bellman-Ford (which doesn't
rely on this greedy assumption) is needed in that case.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Dijkstra's (Approach 1) | All edge weights are non-negative (as guaranteed here) — faster, the standard solution |
| Bellman-Ford (Approach 2) | Negative edge weights might exist, or detecting negative cycles is also needed |

---

## Complexity

| | Dijkstra's | Bellman-Ford |
|---|---|---|
| **Time** | O(E log V) | O(V × E) |
| **Space** | O(V + E) | O(V) |

---

## Key Takeaway

> Network Delay Time introduces Dijkstra's Algorithm — the weighted-edge
> generalization of BFS shortest path. The core mechanism swaps BFS's
> plain queue for a min-heap keyed by accumulated distance, ensuring the
> **closest** known node is always expanded next, which is what
> guarantees correctness when edges have varying costs. This greedy
> "always expand the cheapest frontier" idea, combined with a stale-entry
> guard for heap-based implementations, is the foundation for nearly
> every other weighted shortest-path problem in this Advanced Graphs
> phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/network-delay-time/)

---

{% include kotlin-dsa-nav.html %}
