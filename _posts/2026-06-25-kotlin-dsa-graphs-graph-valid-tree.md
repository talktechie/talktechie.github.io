---
title: "Graphs: Graph Valid Tree — Kotlin Solution"
date: 2026-06-25
categories: [Kotlin DSA, Graphs]
tags: [graph-valid-tree, graph, dfs, union-find, medium, leetcode]
leetcode_number: 261
leetcode_url: https://leetcode.com/problems/graph-valid-tree/
difficulty: Medium
permalink: /posts/kotlin-dsa-graphs-graph-valid-tree/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [261 — Graph Valid Tree](https://leetcode.com/problems/graph-valid-tree/) (Premium — also on NeetCode) |
| **Difficulty** | Medium |
| **Topic** | Graph, DFS, Union-Find |

> Given `n` nodes labeled `0` to `n-1` and a list of undirected `edges`,
> determine if these edges form a **valid tree** — connected, with no
> cycles.

**Example:**
```
Input:  n = 5, edges = [[0,1],[0,2],[0,3],[1,4]]
Output: true
Explanation: a connected, acyclic structure — a valid tree

Input:  n = 5, edges = [[0,1],[1,2],[2,3],[1,3],[1,4]]
Output: false
Explanation: 1-2-3-1 forms a cycle — not a tree
```

**Constraints:**
- `1 <= n <= 2000`
- `0 <= edges.length <= 5000`
- No self-loops or duplicate edges

---

## Approach

A graph with `n` nodes is a valid tree if and only if **two** conditions
both hold:
1. It has **exactly `n - 1` edges** (a tree always has exactly one fewer
   edge than nodes).
2. It is **fully connected** (every node reachable from any other).

Condition 1 alone isn't sufficient — `n-1` edges could still form a
disconnected graph with a cycle in one part and an isolated node
elsewhere. But checking connectivity via a single full traversal,
**combined** with the edge-count check, is sufficient and efficient.

**Key insight:** If `edges.size != n - 1`, immediately return `false` —
no further work needed. Otherwise, run a single DFS/BFS from node 0,
counting how many distinct nodes get visited. If all `n` nodes are
reached, and the edge count was already confirmed correct, the graph
must be both connected and acyclic (a graph with `n` nodes, `n-1` edges,
and full connectivity cannot contain a cycle — adding a cycle would
require an extra edge beyond what full connectivity alone needs).

**Walk through `n = 5, edges = [[0,1],[0,2],[0,3],[1,4]]`:**
```
edges.size = 4 = n-1 = 5-1 = 4 ✓ (passes the necessary count check)

DFS from node 0:
  visit 0 → visit 1 (via edge 0-1) → visit 4 (via edge 1-4)
  visit 2 (via edge 0-2)
  visit 3 (via edge 0-3)

Total visited: {0,1,2,3,4} = 5 nodes = n ✓

Both conditions satisfied → true ✓
```

---

## Kotlin Solution

### Approach 1 — Edge count check + single DFS connectivity check (optimal)

```kotlin
fun validTree(n: Int, edges: Array<IntArray>): Boolean {
    if (edges.size != n - 1) return false   // necessary condition, fails fast

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

    dfs(0)

    return visited.all { it }   // every node reached → fully connected
}
```

### Approach 2 — Union-Find (Disjoint Set Union)

```kotlin
fun validTree(n: Int, edges: Array<IntArray>): Boolean {
    if (edges.size != n - 1) return false

    val parent = IntArray(n) { it }

    fun find(x: Int): Int {
        var root = x
        while (parent[root] != root) root = parent[root]
        parent[x] = root   // path compression
        return root
    }

    for ((a, b) in edges) {
        val rootA = find(a)
        val rootB = find(b)
        if (rootA == rootB) return false   // already connected — this edge creates a cycle
        parent[rootA] = rootB
    }

    return true   // n-1 edges added with no cycle detected → must be a valid tree
}
```

Union-Find directly detects a cycle the instant it would form (when
union-ing two nodes that are already in the same component), rather than
checking connectivity as a separate pass at the end.

---

## Why Edge Count + Connectivity Together Are Sufficient (But Neither Alone Is)

This is worth understanding precisely, since it's a common source of
subtly wrong solutions:

```kotlin
if (edges.size != n - 1) return false
```

- **Too few edges** (`< n-1`): impossible to connect all `n` nodes — some
  must be disconnected. Immediately invalid.
- **Too many edges** (`> n-1`): by the Handshake/graph-theory property,
  any connected graph with more than `n-1` edges **must** contain a
  cycle. Immediately invalid.
- **Exactly `n-1` edges, but disconnected:** possible! E.g., a 4-cycle
  among nodes {0,1,2,3} (4 edges) plus an isolated node 4 — that's 4
  edges total for `n=5` nodes, satisfying `n-1=4`, but it's clearly not a
  valid tree (disconnected, and the 4-cycle itself isn't acyclic either).

This is exactly why **both** checks are necessary: edge count alone
permits this disconnected/cyclic counterexample, and connectivity alone
(without the edge count restricting structure) wouldn't catch certain
malformed inputs either. Together, they're a tight, efficient
characterization of "valid tree."

**Why Union-Find's cycle check happens during edge processing, not
after:** the instant `find(a) == find(b)` for some edge, that edge
connects two nodes already in the same component — adding it would
create a cycle, immediately disqualifying the structure as a tree.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| DFS + edge count (Approach 1) | Simple, intuitive, directly checks both tree properties |
| Union-Find (Approach 2) | Detects the cycle the moment it forms; generalizes well to other "is this still a forest" questions |

---

## Complexity

| | DFS | Union-Find |
|---|---|---|
| **Time** | O(V + E) | O(E × α(V)) ≈ O(E) with path compression |
| **Space** | O(V + E) | O(V) |

---

## Key Takeaway

> "Is this a valid tree" decomposes into two independently necessary, but
> only jointly sufficient, conditions: exactly `n-1` edges, and full
> connectivity. Checking the edge count first is a cheap O(1) filter that
> avoids unnecessary traversal work on clearly invalid inputs. Union-Find
> offers an elegant alternative that catches cycles the instant they'd
> form, rather than verifying connectivity as a separate, final pass —
> a preview of the Union-Find technique used again in Redundant
> Connection later in this phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/graph-valid-tree/)

---

{% include kotlin-dsa-nav.html %}
