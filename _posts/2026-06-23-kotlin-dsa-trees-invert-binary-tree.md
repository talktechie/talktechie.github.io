---
title: "Trees: Invert Binary Tree — Kotlin Solution"
date: 2026-06-21
categories: [Kotlin DSA, Trees]
tags: [invert-binary-tree, tree, dfs, bfs, recursion, easy, leetcode, leetcode-226]
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
| **Topic** | Tree, DFS, BFS, Recursion |

> Given the root of a binary tree, invert the tree (swap every node's left
> and right children), and return its root.

**Example:**
```
Input:      4                  Output:     4
          /   \                          /   \
         2     7                        7     2
        / \   / \                      / \   / \
       1   3 6   9                    9   6 3   1

Input:  [2,1,3]
Output: [2,3,1]
```

**Constraints:**
- The number of nodes is in the range `[0, 100]`
- `-100 <= Node.val <= 100`

---

## Approach

This is the entry point to the Trees phase, establishing the **recursive
DFS template** that nearly every tree problem builds on.

**Key insight:** Inverting a tree means: swap the left and right children
at every node. The recursion is naturally bottom-up — invert the left
subtree, invert the right subtree, then swap them at the current node.
Because every node is processed exactly once, this is O(n).

**Walk through `[4,2,7,1,3,6,9]`:**
```
invert(4):
  invert(2):
    invert(1) → leaf, returns 1 as-is
    invert(3) → leaf, returns 3 as-is
    swap 2's children → 2.left=3, 2.right=1
  invert(7):
    invert(6) → leaf, returns 6 as-is
    invert(9) → leaf, returns 9 as-is
    swap 7's children → 7.left=9, 7.right=6
  swap 4's children → 4.left=7(now 9,6), 4.right=2(now 3,1)

Result:        4
             /   \
            7     2
           / \   / \
          9   6 3   1   ✓
```

---

## Kotlin Solution

### Approach 1 — Recursive DFS (optimal, the standard answer)

```kotlin
class TreeNode(var `val`: Int) {
    var left: TreeNode? = null
    var right: TreeNode? = null
}

fun invertTree(root: TreeNode?): TreeNode? {
    if (root == null) return null

    val left = invertTree(root.left)
    val right = invertTree(root.right)

    root.left = right
    root.right = left

    return root
}
```

### Approach 2 — Iterative BFS with a queue

```kotlin
fun invertTree(root: TreeNode?): TreeNode? {
    if (root == null) return null

    val queue: ArrayDeque<TreeNode> = ArrayDeque()
    queue.addLast(root)

    while (queue.isNotEmpty()) {
        val node = queue.removeFirst()

        val temp = node.left
        node.left = node.right
        node.right = temp

        node.left?.let { queue.addLast(it) }
        node.right?.let { queue.addLast(it) }
    }

    return root
}
```

Level-by-level swapping using a queue — avoids recursion stack space at
the cost of an explicit queue.

---

## Why the Base Case Comes First

```kotlin
if (root == null) return null
```

Every recursive tree function needs a base case for an empty subtree —
without it, calling `invertTree(root.left)` on a leaf node (whose
`left` is `null`) would throw a null pointer exception instead of
gracefully returning.

**The order of operations matters subtly** — recursing first, then
swapping, means the swap happens with the *already-inverted* subtrees:
```kotlin
val left = invertTree(root.left)    // fully invert left subtree first
val right = invertTree(root.right)  // fully invert right subtree first
root.left = right                    // THEN swap the (already inverted) results
root.right = left
```

You could also swap first and recurse on the (now relocated) children —
both orderings produce the same correct result, since inversion is a
symmetric operation applied uniformly.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Recursive DFS (Approach 1) | Always — cleanest, most idiomatic, this is the expected interview answer |
| Iterative BFS (Approach 2) | Want to avoid recursion stack depth on very unbalanced/deep trees |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — every node visited exactly once |
| **Space** | O(h) recursive (h = tree height) / O(w) iterative (w = max width) |

---

## Key Takeaway

> Tree problems almost always follow the same shape: handle the base case
> (usually `null`), recurse on children, then combine the results at the
> current node. Invert Binary Tree is the simplest possible version of
> this template — recurse, then swap. Internalizing this DFS skeleton
> here pays off across nearly every other problem in this phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/invert-binary-tree/)

---

{% include kotlin-dsa-nav.html %}
