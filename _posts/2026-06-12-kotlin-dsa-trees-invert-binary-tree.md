---
title: "Trees: Invert Binary Tree — Kotlin Solution"
date: 2026-06-12
categories: [Kotlin DSA, Trees]
tags: [invert-binary-tree, binary-tree, recursion, bfs, easy, leetcode, leetcode-226]
leetcode_number: 226
leetcode_url: https://leetcode.com/problems/invert-binary-tree/
difficulty: Easy
permalink: /posts/kotlin-dsa-trees-invert-binary-tree/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [226 — Invert Binary Tree](https://leetcode.com/problems/invert-binary-tree/) |
| **Difficulty** | Easy |
| **Topic** | Binary Tree, Recursion, BFS |

> Given the root of a binary tree, invert the tree and return its root.

**Example:**
```
Input:       Output:
    4            4
   / \          / \
  2   7        7   2
 / \ / \      / \ / \
1  3 6  9    9  6 3  1
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

At every node, swap left and right children. Then recursively do the same
for every subtree. That's it.

**Walk through:**
```
Step 1: At node 4 → swap(2, 7)         →  4(left=7, right=2)
Step 2: At node 7 → swap(6, 9)         →  7(left=9, right=6)
Step 3: At node 2 → swap(1, 3)         →  2(left=3, right=1)
Step 4: Leaf nodes → nothing to swap
```

---

## Kotlin Solution

### Approach 1 — Recursive (DFS)

```kotlin
fun invertTree(root: TreeNode?): TreeNode? {
    if (root == null) return null

    val temp = root.left
    root.left = root.right
    root.right = temp

    invertTree(root.left)
    invertTree(root.right)

    return root
}
```

### Approach 2 — Idiomatic Kotlin (also allows)

```kotlin
fun invertTree(root: TreeNode?): TreeNode? {
    root?.apply {
        left = invertTree(right).also { right = invertTree(left) }
    }
    return root
}
```

### Approach 3 — Iterative BFS

```kotlin
fun invertTree(root: TreeNode?): TreeNode? {
    if (root == null) return null

    val queue = ArrayDeque<TreeNode>()
    queue.add(root)

    while (queue.isNotEmpty()) {
        val node = queue.removeFirst()

        val temp = node.left
        node.left = node.right
        node.right = temp

        node.left?.let { queue.add(it) }
        node.right?.let { queue.add(it) }
    }

    return root
}
```

---

## Why Kotlin Shines Here

**`apply` scope function** — operate on the node in-place:
```kotlin
root?.apply {
    // 'this' is root inside here — no need to repeat root.left, root.right
}
```

**`also`** — chains a side effect while returning the value:
```kotlin
left = invertTree(right).also { right = invertTree(left) }
// Evaluates right first, assigns result to left, then assigns original left to right
```

**`ArrayDeque` as Queue** — Kotlin's built-in, no Java Queue import needed:
```kotlin
val queue = ArrayDeque<TreeNode>()
queue.add(root)       // enqueue
queue.removeFirst()   // dequeue
```

**`let` for null-safe queue addition:**
```kotlin
node.left?.let { queue.add(it) }
// Only adds to queue if not null
```

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — visit every node once |
| **Space** | O(h) recursive call stack / O(n) BFS queue |

---

## Key Takeaway

> Swap left and right at every node. Recursion handles the rest.
> The whole algorithm fits in 5 lines.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/invert-binary-tree/)

---

{% include kotlin-dsa-nav.html %}
