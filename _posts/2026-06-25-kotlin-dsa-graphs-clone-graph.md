---
title: "Graphs: Clone Graph — Kotlin Solution"
date: 2026-06-25
categories: [Kotlin DSA, Graphs]
tags: [clone-graph, graph, dfs, bfs, hashmap, medium, leetcode, leetcode-133]
leetcode_number: 133
leetcode_url: https://leetcode.com/problems/clone-graph/
difficulty: Medium
permalink: /posts/kotlin-dsa-graphs-clone-graph/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [133 — Clone Graph](https://leetcode.com/problems/clone-graph/) |
| **Difficulty** | Medium |
| **Topic** | Graph, DFS, BFS, HashMap |

> Given a reference to a node in a **connected, undirected** graph
> (where each node holds a value and a list of neighbors), return a
> **deep copy** of the entire graph.

**Example:**
```
Input: adjList = [[2,4],[1,3],[2,4],[1,3]]
(node 1 connects to 2,4 — node 2 connects to 1,3 — etc.)
Output: [[2,4],[1,3],[2,4],[1,3]]
(a fully independent deep copy with identical structure)

Input: adjList = [[]]
Output: [[]]
(single node, no neighbors)
```

**Constraints:**
- Nodes: `[0, 100]`
- `1 <= Node.val <= 100`
- Node values are unique
- No repeated edges, no self-loops
- The graph is connected

---

## Approach

This generalizes Copy List With Random Pointer (from the Linked List
phase) to an arbitrary graph structure — same core challenge: avoiding
infinite loops when nodes reference each other, this time through
**cycles** rather than just forward/random pointers.

**Key insight:** Use a `HashMap<originalNode, cloneNode>` to track which
nodes have already been cloned. Traverse the graph (DFS or BFS) — the
**first** time we encounter a node, create its clone and record it in the
map immediately, **before** recursing into its neighbors. This is what
prevents infinite recursion on cyclic graphs: by the time we'd revisit an
already-seen node through a different path, the map already has its
clone ready, so we return that instead of cloning again.

**Walk through a graph `1 - 2 - 3 - 1` (a triangle, cyclic)** starting
DFS from node 1:
```
dfs(node=1):
  not in map → create clone(1), map={1: clone(1)}
  for each neighbor of original 1: [2, 3]
    dfs(node=2):
      not in map → create clone(2), map={1:clone(1), 2:clone(2)}
      for each neighbor of original 2: [1, 3]
        dfs(node=1): ALREADY IN MAP → return clone(1) directly, no re-cloning
        clone(2).neighbors.add(clone(1))
        dfs(node=3):
          not in map → create clone(3), map={..., 3:clone(3)}
          for each neighbor of original 3: [2, 1]
            dfs(node=2): ALREADY IN MAP → return clone(2)
            clone(3).neighbors.add(clone(2))
            dfs(node=1): ALREADY IN MAP → return clone(1)
            clone(3).neighbors.add(clone(1))
      clone(2).neighbors.add(clone(3))
  clone(1).neighbors.add(clone(2))
  dfs(node=3): ALREADY IN MAP → return clone(3)
  clone(1).neighbors.add(clone(3))

Result: a fully cloned triangle, no infinite loop ✓
```

---

## Kotlin Solution

### Approach 1 — DFS with a HashMap tracking cloned nodes (optimal, most intuitive)

```kotlin
class Node(var `val`: Int) {
    var neighbors: MutableList<Node> = mutableListOf()
}

fun cloneGraph(node: Node?): Node? {
    if (node == null) return null

    val visited = HashMap<Node, Node>()

    fun dfs(original: Node): Node {
        if (original in visited) return visited[original]!!

        val clone = Node(original.`val`)
        visited[original] = clone   // record BEFORE recursing — prevents infinite loop

        for (neighbor in original.neighbors) {
            clone.neighbors.add(dfs(neighbor))
        }

        return clone
    }

    return dfs(node)
}
```

### Approach 2 — Iterative BFS with the same HashMap idea

```kotlin
fun cloneGraph(node: Node?): Node? {
    if (node == null) return null

    val visited = HashMap<Node, Node>()
    visited[node] = Node(node.`val`)

    val queue: ArrayDeque<Node> = ArrayDeque()
    queue.addLast(node)

    while (queue.isNotEmpty()) {
        val curr = queue.removeFirst()

        for (neighbor in curr.neighbors) {
            if (neighbor !in visited) {
                visited[neighbor] = Node(neighbor.`val`)
                queue.addLast(neighbor)
            }
            visited[curr]!!.neighbors.add(visited[neighbor]!!)
        }
    }

    return visited[node]
}
```

Avoids recursion entirely — processes nodes level by level, building
each clone's neighbor list as their originals are dequeued.

---

## Why Recording the Clone in the Map BEFORE Recursing Is Essential

This is the exact same insight as Copy List With Random Pointer's
"forward reference" problem, just generalized to handle **cycles**
instead of one-directional random pointers:

```kotlin
val clone = Node(original.`val`)
visited[original] = clone   // MUST happen before the neighbor loop below
for (neighbor in original.neighbors) {
    clone.neighbors.add(dfs(neighbor))
}
```

If we recorded the clone **after** processing neighbors instead, a cyclic
graph (where some neighbor eventually loops back to `original`) would
cause `dfs` to call itself on `original` again before it's been marked
as visited — infinite recursion, stack overflow.

**Why `HashMap<Node, Node>` works correctly even though `Node` doesn't
override `equals`/`hashCode`:** by default, Kotlin/Java objects use
reference equality for hashing — which is exactly what we want here,
since we need to distinguish "this specific original node object" from
any other node, even if two different nodes happened to share the same
`val`.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| DFS with HashMap (Approach 1) | Most intuitive, mirrors the recursive structure of the graph itself |
| BFS with HashMap (Approach 2) | Want to avoid recursion stack depth on very large graphs |

---

## Complexity

| | |
|---|---|
| **Time** | O(V + E) — V nodes, E edges, each visited/processed once |
| **Space** | O(V) — the HashMap, plus recursion stack (DFS) or queue (BFS) |

---

## Key Takeaway

> Cloning a graph with potential cycles requires exactly the same
> "record before recursing" discipline as Copy List With Random Pointer
> — a HashMap mapping original→clone, populated the **instant** a node is
> first encountered, before any of its neighbors are processed. This
> guarantees that revisiting a node through a different path (which
> happens constantly in cyclic graphs, unlike in a simple linked list)
> safely returns the already-created clone instead of recursing
> indefinitely.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/clone-graph/)

---

{% include kotlin-dsa-nav.html %}
