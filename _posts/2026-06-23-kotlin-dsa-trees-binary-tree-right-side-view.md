---
title: "Trees: Binary Tree Right Side View — Kotlin Solution"
date: 2026-06-21
categories: [Kotlin DSA, Trees]
tags: [binary-tree-right-side-view, tree, bfs, dfs, medium, leetcode, leetcode-199]
leetcode_number: 199
leetcode_url: https://leetcode.com/problems/binary-tree-right-side-view/
difficulty: Medium
permalink: /posts/kotlin-dsa-trees-binary-tree-right-side-view/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [199 — Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/) |
| **Difficulty** | Medium |
| **Topic** | Tree, BFS, DFS |

> Given the root of a binary tree, imagine standing on the **right side**
> of it. Return the values of the nodes you can see, ordered from top to
> bottom.

**Example:**
```
Input:      1
          /   \
         2     3
          \     \
           5     4
Output: [1,3,4]

Input:  [1,null,3]
Output: [1,3]

Input:  []
Output: []
```

**Constraints:**
- Nodes: `[0, 100]`
- `-100 <= Node.val <= 100`

---

## Approach

This builds directly on Level Order Traversal — once you can group nodes
by level, "the rightmost node of each level" is just **the last element**
of every level's list.

**Key insight:** Same BFS-by-level template as LC 102. The only change:
instead of collecting every value in a level, just keep the **last** node
processed at each level — that's the one visible from the right side.

**Walk through `[1,2,3,null,5,null,4]`:**
```
Level 0: [1]      → rightmost = 1
Level 1: [2,3]    → rightmost = 3
Level 2: [5,4]    → rightmost = 4   (5 is 2's right child, 4 is 3's right child)

Result: [1,3,4] ✓
```

---

## Kotlin Solution

### Approach 1 — BFS, take the last node of each level (optimal)

```kotlin
fun rightSideView(root: TreeNode?): List<Int> {
    if (root == null) return emptyList()

    val result = mutableListOf<Int>()
    val queue: ArrayDeque<TreeNode> = ArrayDeque()
    queue.addLast(root)

    while (queue.isNotEmpty()) {
        val levelSize = queue.size

        repeat(levelSize) { i ->
            val node = queue.removeFirst()

            if (i == levelSize - 1) result.add(node.`val`)  // last node in this level

            node.left?.let { queue.addLast(it) }
            node.right?.let { queue.addLast(it) }
        }
    }

    return result
}
```

### Approach 2 — DFS, right child first, record first node seen at each depth

```kotlin
fun rightSideView(root: TreeNode?): List<Int> {
    val result = mutableListOf<Int>()

    fun dfs(node: TreeNode?, depth: Int) {
        if (node == null) return

        if (depth == result.size) result.add(node.`val`)  // first time reaching this depth

        dfs(node.right, depth + 1)  // visit RIGHT first
        dfs(node.left, depth + 1)
    }

    dfs(root, 0)
    return result
}
```

A neat trick: by visiting the **right** child before the left at every
node, the *first* node encountered at each depth is guaranteed to be the
rightmost one — no need to compare or track "last" explicitly.

---

## Why Visiting Right Before Left Works in the DFS Version

This is the cleverest part of Approach 2. Normally DFS visits left
before right. Flipping that order means: for any given depth, the very
first node our traversal reaches is always the rightmost one at that
depth, because we always exhaust the rightward path before ever
considering a leftward branch.

```kotlin
if (depth == result.size) result.add(node.`val`)
// This condition is only true the FIRST time we reach this depth —
// and because of right-first traversal, that first arrival is
// guaranteed to be the rightmost node
dfs(node.right, depth + 1)  // explore right subtree fully first
dfs(node.left, depth + 1)   // left subtree visited after, won't override result[depth]
```

**Why the BFS version checks `i == levelSize - 1`:**
```kotlin
repeat(levelSize) { i ->
    val node = queue.removeFirst()
    if (i == levelSize - 1) result.add(node.`val`)
    // ...
}
```
Since the queue processes nodes left-to-right within a level (children
were added in left-then-right order during the previous level), the last
node dequeued in any level batch is always the rightmost one.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| BFS, last node per level (Approach 1) | Most intuitive — directly extends Level Order Traversal |
| DFS, right-first (Approach 2) | Elegant, avoids needing the level-size snapshot pattern |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — every node visited once |
| **Space** | O(w) BFS (max width) / O(h) DFS (max height) |

---

## Key Takeaway

> "What's visible from one side" is "the boundary node at each level" —
> a small variation on level-order traversal. The BFS approach is the
> direct extension of LC 102; the DFS approach with right-first traversal
> is the more elegant trick worth knowing, since it eliminates the need
> for level-size bookkeeping entirely by exploiting traversal order
> itself to guarantee correctness.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/binary-tree-right-side-view/)

---

{% include kotlin-dsa-nav.html %}
