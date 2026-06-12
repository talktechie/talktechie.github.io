---
title: "Graphs: Clone Graph — Kotlin Solution"
date: 2026-06-12
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

> Given a reference of a node in a connected undirected graph,
> return a deep copy (clone) of the graph.

**Node definition:**
```kotlin
class Node(var `val`: Int) {
    var neighbors: ArrayList<Node?> = ArrayList()
}
```

**Example:**
```
Input:  1 -- 2
        |    |
        4 -- 3

Output: Deep copy of the same structure
```

---

## Approach

The key challenge — cycles. A graph can have cycles, so naive recursion
would loop forever.

**The fix — a HashMap as a visited map:**
`original node → cloned node`

Before cloning a node, check if it's already been cloned.
If yes — return the existing clone. If no — create a new clone,
register it in the map, then clone its neighbors.

**Walk through:**
```
Clone(1): not in map → create Clone(1), map={1→C1}
  Clone(2): not in map → create Clone(2), map={1→C1, 2→C2}
    Clone(1): already in map → return C1  ← cycle handled!
    Clone(3): not in map → create Clone(3)...
  Clone(4): not in map → create Clone(4)...
```

---

## Kotlin Solution

### Approach 1 — DFS with HashMap

```kotlin
fun cloneGraph(node: Node?): Node? {
    if (node == null) return null

    val cloned = HashMap<Node, Node>()

    fun dfs(original: Node): Node {
        cloned[original]?.let { return it }  // already cloned

        val copy = Node(original.`val`)
        cloned[original] = copy              // register before recursing (avoid cycles)

        for (neighbor in original.neighbors) {
            neighbor?.let { copy.neighbors.add(dfs(it)) }
        }

        return copy
    }

    return dfs(node)
}
```

### Approach 2 — BFS with HashMap

```kotlin
fun cloneGraph(node: Node?): Node? {
    if (node == null) return null

    val cloned = HashMap<Node, Node>()
    val queue = ArrayDeque<Node>()

    cloned[node] = Node(node.`val`)
    queue.add(node)

    while (queue.isNotEmpty()) {
        val current = queue.removeFirst()

        for (neighbor in current.neighbors) {
            neighbor ?: continue

            if (neighbor !in cloned) {
                cloned[neighbor] = Node(neighbor.`val`)
                queue.add(neighbor)
            }

            cloned[current]!!.neighbors.add(cloned[neighbor])
        }
    }

    return cloned[node]
}
```

---

## Why Kotlin Shines Here

**Local function `dfs`** — captures `cloned` map without a class field:
```kotlin
fun cloneGraph(node: Node?): Node? {
    val cloned = HashMap<Node, Node>()
    fun dfs(original: Node): Node { ... }  // cloned is in scope
    return dfs(node)
}
// vs Java: needs a helper method with the map as a parameter
```

**`cloned[original]?.let { return it }`** — safe early return:
```kotlin
cloned[original]?.let { return it }
// If already cloned, return immediately
// vs Java: if (cloned.containsKey(original)) return cloned.get(original);
```

**`neighbor !in cloned`** — readable map membership check:
```kotlin
if (neighbor !in cloned) { ... }
// vs Java: if (!cloned.containsKey(neighbor))
```

**`neighbor ?: continue`** — skip null neighbors cleanly:
```kotlin
neighbor ?: continue
// vs Java: if (neighbor == null) continue;
```

---

## Critical — Register Before Recursing

```kotlin
val copy = Node(original.`val`)
cloned[original] = copy   // ← MUST be before the neighbor loop!

for (neighbor in original.neighbors) { ... }
```

If you register after the loop, you hit infinite recursion on cycles.
Register first, then recurse into neighbors.

---

## Complexity

| | |
|---|---|
| **Time** | O(V + E) — visit every node and edge once |
| **Space** | O(V) — HashMap stores one entry per node |

---

## Key Takeaway

> A HashMap from original → clone is your cycle guard.
> Always register the clone before recursing into neighbors.
> Check the map first — if already cloned, return it immediately.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/clone-graph/)

---

{% include kotlin-dsa-nav.html %}
