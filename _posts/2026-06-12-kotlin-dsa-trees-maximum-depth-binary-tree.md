---
title: "Trees: Maximum Depth of Binary Tree — Kotlin Solution"
date: 2026-06-12
categories: [Kotlin DSA, Trees]
tags: [maximum-depth-binary-tree, binary-tree, recursion, dfs, easy, leetcode, leetcode-104]
leetcode_number: 104
leetcode_url: https://leetcode.com/problems/maximum-depth-of-binary-tree/
difficulty: Easy
permalink: /posts/kotlin-dsa-trees-maximum-depth-binary-tree/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [104 — Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/) |
| **Difficulty** | Easy |
| **Topic** | Binary Tree, DFS, BFS |

> Given the root of a binary tree, return its maximum depth.
> Maximum depth is the number of nodes along the longest path
> from the root down to the farthest leaf node.

**Example:**
```
    3
   / \
  9  20
    /  \
   15   7

Output: 3
```

---

## The Node Definition

```kotlin
class TreeNode(var `val`: Int) {
    var left: TreeNode? = null
    var right: TreeNode? = null
}
```

---

## Approach

The depth of a tree is:
> 1 + max(depth of left subtree, depth of right subtree)

At a null node, depth = 0. At a leaf, depth = 1.
Recursion handles the rest naturally.

**Walk through:**
```
depth(3)  = 1 + max(depth(9), depth(20))
depth(9)  = 1 + max(0, 0) = 1
depth(20) = 1 + max(depth(15), depth(7))
depth(15) = 1, depth(7) = 1
depth(20) = 1 + max(1, 1) = 2
depth(3)  = 1 + max(1, 2) = 3 ✓
```

---

## Kotlin Solution

### Approach 1 — Recursive DFS (cleanest)

```kotlin
fun maxDepth(root: TreeNode?): Int {
    if (root == null) return 0
    return 1 + maxOf(maxDepth(root.left), maxDepth(root.right))
}
```

### Approach 2 — Iterative BFS (level-by-level)

```kotlin
fun maxDepth(root: TreeNode?): Int {
    if (root == null) return 0

    val queue = ArrayDeque<TreeNode>()
    queue.add(root)
    var depth = 0

    while (queue.isNotEmpty()) {
        repeat(queue.size) {
            val node = queue.removeFirst()
            node.left?.let { queue.add(it) }
            node.right?.let { queue.add(it) }
        }
        depth++
    }

    return depth
}
```

---

## Why Kotlin Shines Here

**One-liner recursive solution** — Kotlin makes the cleanest version possible:
```kotlin
return 1 + maxOf(maxDepth(root.left), maxDepth(root.right))
// The entire algorithm in one line
// vs Java: return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
// Kotlin's maxOf reads more naturally
```

**`repeat(queue.size)`** — process exactly one level at a time:
```kotlin
repeat(queue.size) {
    // processes all nodes at current depth before moving to next
}
// vs Java: int size = queue.size(); for (int i = 0; i < size; i++)
```

**`?.let`** — null-safe child addition:
```kotlin
node.left?.let { queue.add(it) }
// Only adds if non-null — no if (node.left != null) needed
```

---

## Recursive vs Iterative BFS

| Approach | Time | Space | Notes |
|---|---|---|---|
| Recursive | O(n) | O(h) | Cleanest — one line |
| BFS | O(n) | O(w) | `w` = max width of tree |

`h` = height, `w` = max width. For a balanced tree, h = log n.
For a skewed tree, h = n (recursive risks stack overflow).
BFS is safer for very deep trees.

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — visit every node |
| **Space** | O(h) recursive / O(w) BFS |

---

## Key Takeaway

> Depth = 1 + max(left depth, right depth).
> The recursive solution is a single return statement in Kotlin.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/maximum-depth-of-binary-tree/)

---

{% include kotlin-dsa-nav.html %}
