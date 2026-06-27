---
title: "Graphs: Number of Connected Components In An Undirected Graph — Kotlin Solution"
date: 2026-06-25
categories: [Kotlin DSA, Graphs]
tags: [number-of-connected-components, graph, dfs, union-find, medium, leetcode]
leetcode_number: 323
leetcode_url: https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/
difficulty: Medium
permalink: /posts/kotlin-dsa-graphs-number-of-connected-components-in-an-undirected-graph/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [323 — Number of Connected Components In An Undirected Graph](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/) (Premium — also on NeetCode) |
| **Difficulty** | Medium |
| **Topic** | Graph, DFS, Union-Find |

> Given `n` nodes labeled `0` to `n-1` and a list of undirected `edges`,
> return the number of **connected components** in the graph.

**Example:**
```
Input:  n = 5, edges = [[0,1],[1,2],[3,4]]
Output: 2
Explanation: {0,1,2} form one component, {3,4} form another

Input:  n = 5, edges = [[0,1],[1,2],[2,3],[3,4]]
Output: 1
```

**Constraints:**
- `1 <= n <= 2000`
- `0 <= edges.length <= n × (n-1) / 2`
- No self-loops or duplicate edges

---

## Approach

This is the array/list version of Number of Islands' "scan and flood-fill"
template — instead of a 2D grid, the structure is an explicit edge list,
but the underlying idea is identical.

**Key insight (DFS/BFS):** Build an adjacency list from the edges. Scan
every node from `0` to `n-1`; whenever an unvisited node is found,
that's the start of a **new** component — flood-fill (DFS or BFS) to mark
every node reachable from it as visited, incrementing the component count
once per new flood-fill triggered.

**Key insight (Union-Find):** Alternatively, process every edge,
union-ing the two endpoints' components together. The final number of
**distinct root parents** remaining is the answer — every union that
successfully merges two previously-separate components reduces the
total component count by exactly one.

**Walk through `n = 5, edges = [[0,1],[1,2],[3,4]]`** using DFS:
```
Scan node 0: unvisited → NEW COMPONENT #1
  flood-fill: visit 0, 1 (via edge 0-1), 2 (via edge 1-2)

Scan node 1: already visited → skip
Scan node 2: already visited → skip
Scan node 3: unvisited → NEW COMPONENT #2
  flood-fill: visit 3, 4 (via edge 3-4)

Scan node 4: already visited → skip

Total components: 2 ✓
```

---

## Kotlin Solution

### Approach 1 — DFS flood-fill, count new components found (intuitive)

```kotlin
fun countComponents(n: Int, edges: Array<IntArray>): Int {
    val graph = Array(n) { mutableListOf<Int>() }
    for ((a, b) in edges) {
        graph[a].add(b)
        graph[b].add(a)
    }

    val visited = BooleanArray(n)

    fun dfs(node: Int) {
        visited[node] = true
        for (neighbor in graph[node]) {
            if (!visited[neighbor]) dfs(neighbor)
        }
    }

    var count = 0
    for (node in 0 until n) {
        if (!visited[node]) {
            count++
            dfs(node)
        }
    }

    return count
}
```

### Approach 2 — Union-Find, count distinct roots (often faster in practice)

```kotlin
fun countComponents(n: Int, edges: Array<IntArray>): Int {
    val parent = IntArray(n) { it }
    var components = n   // start assuming every node is its own component

    fun find(x: Int): Int {
        var root = x
        while (parent[root] != root) root = parent[root]
        parent[x] = root   // path compression
        return root
    }

    for ((a, b) in edges) {
        val rootA = find(a)
        val rootB = find(b)
        if (rootA != rootB) {
            parent[rootA] = rootB
            components--   // successfully merged two components into one
        }
    }

    return components
}
```

Starts by assuming every node is isolated (`n` components), then
decrements once per successful merge — no final scan needed at all, the
running count IS the answer by the time every edge is processed.

---

## Why Union-Find's Running Counter Avoids a Separate Counting Pass

```kotlin
var components = n
// ...
if (rootA != rootB) {
    parent[rootA] = rootB
    components--
}
```

Each node starts as its own isolated component (hence `n` initial
components). Every time we successfully union two **different** roots,
we've just merged two previously-separate components into one — reducing
the total count by exactly one. If `rootA == rootB` already, the edge
connects two nodes already in the same component (this edge is
"redundant" from a connectivity standpoint), and the count doesn't change.

By the time every edge has been processed, `components` already holds
the final answer — no need for a separate "scan all nodes and find
distinct roots" pass at the end, which the DFS approach's "count new
flood-fills triggered" achieves through different but parallel logic.

**Why path compression (`parent[x] = root`) matters for performance:**
without it, `find` could degrade to O(n) per call in a worst-case skewed
tree structure; with it, repeated calls flatten the tree, keeping
amortized `find` calls close to O(1) over many operations.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| DFS flood-fill (Approach 1) | Intuitive, directly mirrors Number of Islands' scan-and-flood-fill pattern |
| Union-Find (Approach 2) | Often faster in practice, no need to build an explicit adjacency list at all |

---

## Complexity

| | DFS | Union-Find |
|---|---|---|
| **Time** | O(V + E) | O(E × α(V)) ≈ O(E) with path compression |
| **Space** | O(V + E) — adjacency list | O(V) |

---

## Key Takeaway

> Counting connected components is the array/edge-list counterpart to
> Number of Islands' grid-based "scan + flood-fill + count" pattern —
> same core idea, different underlying representation. Union-Find offers
> an alternative that's often more efficient in practice for
> edge-list-based graphs: rather than scanning every node and triggering
> separate flood-fills, it processes edges directly and maintains a
> running component count that decreases by exactly one with every
> successful merge — arriving at the final answer with no separate
> counting pass needed.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/)

---

{% include kotlin-dsa-nav.html %}
