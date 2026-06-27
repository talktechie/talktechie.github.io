---
title: "Graphs: Redundant Connection — Kotlin Solution"
date: 2026-06-25
categories: [Kotlin DSA, Graphs]
tags: [redundant-connection, graph, union-find, medium, leetcode, leetcode-684]
leetcode_number: 684
leetcode_url: https://leetcode.com/problems/redundant-connection/
difficulty: Medium
permalink: /posts/kotlin-dsa-graphs-redundant-connection/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [684 — Redundant Connection](https://leetcode.com/problems/redundant-connection/) |
| **Difficulty** | Medium |
| **Topic** | Graph, Union-Find |

> A tree with `n` nodes had one **extra** edge added, creating exactly
> one cycle. Given `edges` (originally a valid tree, plus one redundant
> edge), return the **edge that can be removed** to restore a valid
> tree. If multiple answers exist, return the one that appears **last**
> in the input.

**Example:**
```
Input:  edges = [[1,2],[1,3],[2,3]]
Output: [2,3]
Explanation: edges 1-2 and 1-3 already connect all 3 nodes as a tree;
             2-3 is the extra edge that creates the cycle, and it
             appears last among the redundant options

Input:  edges = [[1,2],[2,3],[3,4],[1,4],[1,5]]
Output: [1,4]
```

**Constraints:**
- `n == edges.length`
- `3 <= n <= 1000`
- Each edge is `[ai, bi]` with `1 <= ai < bi <= n`
- No duplicate edges

---

## Approach

This is a direct, focused application of Union-Find's cycle-detection
property, introduced in Graph Valid Tree — except now we need to
identify and return the **specific edge** that creates the cycle, not
just confirm a cycle exists.

**Key insight:** Process edges **in the given order**, union-ing each
pair. The moment we attempt to union two nodes that are **already** in
the same component (i.e., `find(a) == find(b)` before unioning), that
edge is exactly the redundant one — it connects two nodes that were
already connected through some earlier-processed edge, meaning this
edge's only effect is to create a cycle. Since we process in order and
return immediately upon detection, we automatically satisfy "return the
one that appears last" — any earlier redundant-seeming edge would have
already been processed without issue, since at that point the cycle
hadn't fully formed yet.

**Walk through `edges = [[1,2],[1,3],[2,3]]`:**
```
parent: {1:1, 2:2, 3:3} (each node starts as its own root)

process [1,2]: find(1)=1, find(2)=2 → different roots → union: parent[1]=2
  parent: {1:2, 2:2, 3:3}

process [1,3]: find(1)=2 (follows 1→2), find(3)=3 → different roots → union: parent[2]=3
  parent: {1:2, 2:3, 3:3}

process [2,3]: find(2)=3 (follows 2→3), find(3)=3 → SAME ROOT!
  → this edge creates a cycle → return [2,3] ✓
```

---

## Kotlin Solution

### Approach 1 — Union-Find, return the first edge that connects an already-unified pair (optimal)

```kotlin
fun findRedundantConnection(edges: Array<IntArray>): IntArray {
    val n = edges.size
    val parent = IntArray(n + 1) { it }   // nodes are 1-indexed

    fun find(x: Int): Int {
        var root = x
        while (parent[root] != root) root = parent[root]
        parent[x] = root   // path compression
        return root
    }

    for ((a, b) in edges) {
        val rootA = find(a)
        val rootB = find(b)

        if (rootA == rootB) {
            return intArrayOf(a, b)   // this edge connects an already-connected pair — redundant
        }

        parent[rootA] = rootB
    }

    return intArrayOf()   // unreachable given problem guarantees
}
```

### Approach 2 — Union-Find with union by rank (slightly more efficient at scale)

```kotlin
fun findRedundantConnection(edges: Array<IntArray>): IntArray {
    val n = edges.size
    val parent = IntArray(n + 1) { it }
    val rank = IntArray(n + 1)

    fun find(x: Int): Int {
        var root = x
        while (parent[root] != root) root = parent[root]
        parent[x] = root
        return root
    }

    fun union(a: Int, b: Int): Boolean {
        val rootA = find(a)
        val rootB = find(b)
        if (rootA == rootB) return false   // already connected — redundant edge

        if (rank[rootA] < rank[rootB]) {
            parent[rootA] = rootB
        } else if (rank[rootA] > rank[rootB]) {
            parent[rootB] = rootA
        } else {
            parent[rootB] = rootA
            rank[rootA]++
        }
        return true
    }

    for ((a, b) in edges) {
        if (!union(a, b)) return intArrayOf(a, b)
    }

    return intArrayOf()
}
```

Union by rank attaches the smaller tree under the larger tree's root,
keeping the overall structure flatter — a further optimization on top of
path compression, more impactful as graphs grow large.

---

## Why Processing Edges IN ORDER Automatically Satisfies "Return the Last Valid Answer"

This is a subtle but elegant consequence of how Union-Find naturally
operates, worth understanding precisely:

```kotlin
for ((a, b) in edges) {
    val rootA = find(a)
    val rootB = find(b)
    if (rootA == rootB) return intArrayOf(a, b)   // first such edge encountered
    parent[rootA] = rootB
}
```

Before the actual redundant edge is reached, **every** earlier edge in
the list successfully unions two **previously separate** components
(this is guaranteed, since the problem states the input is a valid tree
plus exactly one extra edge — meaning every edge except the truly
redundant one is "necessary" for connectivity at the point it's
processed). The **first** edge we encounter that connects an
already-unified pair must therefore be the (unique) redundant edge — and
because we process in input order and return immediately, this is
naturally the one that appears latest among any edges that *could* be
candidates, satisfying the problem's tiebreak rule without any extra
logic.

**Why `n + 1` sized arrays, not `n`:** nodes are explicitly 1-indexed
(`1 <= ai < bi <= n`) per the constraints, so allocating `parent` and
`rank` with size `n+1` avoids needing to offset every node index by 1
throughout the code.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Basic Union-Find (Approach 1) | Simple, sufficient for this problem's constraints (n ≤ 1000) |
| Union by rank (Approach 2) | Want the more rigorously optimal Union-Find variant, useful habit for larger-scale graph problems |

---

## Complexity

| | Basic | Union by Rank |
|---|---|---|
| **Time** | O(E × α(V)) ≈ O(E) with path compression alone | O(E × α(V)), slightly tighter constant in practice |
| **Space** | O(V) | O(V) |

---

## Key Takeaway

> Redundant Connection is Union-Find's cycle-detection property applied
> with surgical precision: process edges in order, and the very first
> edge that tries to connect two nodes already in the same component is
> the answer. This problem is a clean, minimal illustration of why
> Union-Find is often the most natural tool whenever a problem talks
> about edges being added one at a time and asks something about when/
> where connectivity-related structure breaks down or becomes redundant.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/redundant-connection/)

---

{% include kotlin-dsa-nav.html %}
