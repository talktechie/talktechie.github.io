---
title: "Trees: Binary Tree Level Order Traversal — Kotlin Solution"
date: 2026-06-21
categories: [Kotlin DSA, Trees]
tags: [binary-tree-level-order-traversal, tree, bfs, queue, medium, leetcode, leetcode-102]
leetcode_number: 102
leetcode_url: https://leetcode.com/problems/binary-tree-level-order-traversal/
difficulty: Medium
permalink: /posts/kotlin-dsa-trees-binary-tree-level-order-traversal/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [102 — Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/) |
| **Difficulty** | Medium |
| **Topic** | Tree, BFS, Queue |

> Given the root of a binary tree, return the **level order traversal**
> of its node values — i.e., visit nodes level by level, left to right,
> grouping each level into its own list.

**Example:**
```
Input:      3
          /   \
         9     20
              /   \
             15    7
Output: [[3],[9,20],[15,7]]

Input:  [1]
Output: [[1]]

Input:  []
Output: []
```

**Constraints:**
- Nodes: `[0, 2000]`
- `-1000 <= Node.val <= 1000`

---

## Approach

This is the canonical **BFS** problem for trees — the first problem in
this phase where DFS recursion isn't the natural fit. We need to process
nodes level by level, which is exactly what a queue is built for.

**Key insight:** Use a queue, but process it in **batches** — snapshot the
queue's current size before adding any new children, then process exactly
that many nodes as "this level." Every child added during that batch
belongs to the *next* level, naturally separating levels without any
extra bookkeeping.

**Walk through `[3,9,20,null,null,15,7]`:**
```
queue: [3]
Level 0: size=1 → process node 3 → add children 9,20 → result=[[3]]
queue: [9,20]
Level 1: size=2 → process 9 (no children), process 20 (add children 15,7)
        → result=[[3],[9,20]]
queue: [15,7]
Level 2: size=2 → process 15, process 7 (no children for either)
        → result=[[3],[9,20],[15,7]]
queue: [] → done

Result: [[3],[9,20],[15,7]] ✓
```

---

## Kotlin Solution

### Approach 1 — Iterative BFS with level-size snapshot (optimal)

```kotlin
fun levelOrder(root: TreeNode?): List<List<Int>> {
    if (root == null) return emptyList()

    val result = mutableListOf<List<Int>>()
    val queue: ArrayDeque<TreeNode> = ArrayDeque()
    queue.addLast(root)

    while (queue.isNotEmpty()) {
        val levelSize = queue.size
        val level = mutableListOf<Int>()

        repeat(levelSize) {
            val node = queue.removeFirst()
            level.add(node.`val`)

            node.left?.let { queue.addLast(it) }
            node.right?.let { queue.addLast(it) }
        }

        result.add(level)
    }

    return result
}
```

### Approach 2 — Recursive DFS, tracking depth explicitly

```kotlin
fun levelOrder(root: TreeNode?): List<List<Int>> {
    val result = mutableListOf<MutableList<Int>>()

    fun dfs(node: TreeNode?, depth: Int) {
        if (node == null) return

        if (depth == result.size) result.add(mutableListOf())
        result[depth].add(node.`val`)

        dfs(node.left, depth + 1)
        dfs(node.right, depth + 1)
    }

    dfs(root, 0)
    return result
}
```

A less obvious but valid alternative — recursion can also produce
level-order output by tracking depth and lazily growing the result list
as deeper levels are first encountered.

---

## Why Snapshotting `queue.size` Is Essential

This is the one detail that makes or breaks the BFS solution:

```kotlin
val levelSize = queue.size   // MUST capture this before the inner loop starts
repeat(levelSize) {
    val node = queue.removeFirst()
    // ...
    node.left?.let { queue.addLast(it) }   // these additions grow queue.size
    node.right?.let { queue.addLast(it) }  // but we already fixed levelSize above
}
```

If we used `queue.size` directly inside a `while` condition instead of a
fixed `repeat(levelSize)`, the loop would never terminate correctly — the
size keeps growing as children are added, so the loop would accidentally
swallow nodes from the next level into the current one.

**Why DFS can also solve a "BFS-shaped" problem (Approach 2):** the key is
that `depth` tells us exactly which level a node belongs to, regardless of
traversal order — we're not relying on visiting order to group levels,
just using `depth == result.size` as a signal "this is a level we haven't
started yet":
```kotlin
if (depth == result.size) result.add(mutableListOf())
// First time we reach a given depth, create a new sublist for it
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Iterative BFS (Approach 1) | Always — most natural fit, the standard expected answer |
| Recursive DFS with depth tracking (Approach 2) | Interesting alternative, useful if recursion is preferred/required |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — every node visited once |
| **Space** | O(w) — w = maximum width of the tree (queue size at the widest level) |

---

## Key Takeaway

> Level-order traversal is BFS's signature use case on trees. The
> "snapshot the queue size before the inner loop" trick is the reusable
> pattern here — it cleanly separates "this level" from "the next level"
> without any extra markers or sentinel values. This same BFS skeleton
> powers Binary Tree Right Side View and many other level-based problems
> later in this phase, just with a small tweak to what gets recorded
> from each level.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/binary-tree-level-order-traversal/)

---

{% include kotlin-dsa-nav.html %}
