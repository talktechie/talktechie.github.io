---
title: "Trees: Diameter of Binary Tree — Kotlin Solution"
date: 2026-06-21
categories: [Kotlin DSA, Trees]
tags: [diameter-of-binary-tree, tree, dfs, recursion, easy, leetcode, leetcode-543]
leetcode_number: 543
leetcode_url: https://leetcode.com/problems/diameter-of-binary-tree/
difficulty: Easy
permalink: /posts/kotlin-dsa-trees-diameter-of-binary-tree/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [543 — Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/) |
| **Difficulty** | Easy |
| **Topic** | Tree, DFS, Recursion |

> Given the root of a binary tree, return the length of the **diameter**
> — the length of the longest path between any two nodes. This path may
> or may not pass through the root. Length is measured in number of
> **edges** between the two nodes.

**Example:**
```
Input:      1
          /   \
         2     3
        / \
       4   5
Output: 3
Explanation: longest path is [4,2,1,3] or [5,2,1,3] — 3 edges

Input:  [1,2]
Output: 1
```

**Constraints:**
- The number of nodes is in the range `[1, 10⁴]`
- `-100 <= Node.val <= 100`

---

## Approach

The key realization: the longest path **through any given node** equals
`leftDepth + rightDepth` — the depth of its left subtree plus the depth
of its right subtree (each depth measured as a count of edges, not nodes).

**Key insight:** The diameter of the *whole tree* is the **maximum** of
`leftDepth + rightDepth` computed at **every single node**, not just the
root. The longest path might dip down through some node deep in the tree,
never touching the root at all.

So we compute depth recursively (same as Maximum Depth of Binary Tree),
but at every node, we also check if `leftDepth + rightDepth` beats the
best diameter found so far.

**Walk through `[1,2,3,4,5]`:**
```
depth(4) = 0 (leaf)   depth(5) = 0 (leaf)
At node 2: leftDepth=depth(4)=0, rightDepth=depth(5)=0
           diameter candidate = 0+0 = 0
           depth(2) = 1 + max(0,0) = 1

depth(3) = 0 (leaf)
At node 1: leftDepth=depth(2)=1, rightDepth=depth(3)=0
           diameter candidate = 1+0 = 1   ← but wait, this misses the real answer

Actually the longest path [4,2,1,3] has edges: 4-2, 2-1, 1-3 = 3 edges.
This is leftDepth(at node 2, going down to 4)=1 PLUS the path up through 1
to 3. Let's redo this checking diameter AT NODE 2 itself:
At node 2: leftDepth=depth(4)=1 (edge to leaf), rightDepth=depth(5)=1
           diameter candidate at node 2 = 1+1 = 2

At node 1: leftDepth=depth(2)=2 (height of subtree rooted at 2),
           rightDepth=depth(3)=1
           diameter candidate at node 1 = 2+1 = 3  ← this is the answer!

Maximum diameter across all nodes: 3 ✓
```

---

## Kotlin Solution

### Approach 1 — Single DFS, track diameter as a side effect (optimal)

```kotlin
fun diameterOfBinaryTree(root: TreeNode?): Int {
    var diameter = 0

    fun depth(node: TreeNode?): Int {
        if (node == null) return 0

        val leftDepth = depth(node.left)
        val rightDepth = depth(node.right)

        diameter = maxOf(diameter, leftDepth + rightDepth)

        return 1 + maxOf(leftDepth, rightDepth)
    }

    depth(root)
    return diameter
}
```

### Approach 2 — Return both depth and diameter as a Pair (no captured variable)

```kotlin
fun diameterOfBinaryTree(root: TreeNode?): Int {
    fun helper(node: TreeNode?): Pair<Int, Int> {  // (depth, bestDiameterInSubtree)
        if (node == null) return 0 to 0

        val (leftDepth, leftDia) = helper(node.left)
        val (rightDepth, rightDia) = helper(node.right)

        val depth = 1 + maxOf(leftDepth, rightDepth)
        val diameter = maxOf(leftDepth + rightDepth, leftDia, rightDia)

        return depth to diameter
    }

    return helper(root).second
}
```

Avoids a mutable captured variable entirely — each call returns both
pieces of information it computed, and the caller combines them. Slightly
more allocation (creating `Pair`s) but a purer functional style.

---

## Why We Check the Diameter at Every Node, Not Just the Root

This is the single most important insight in the problem. The answer
might come from a path entirely within a subtree, never reaching the root:

```kotlin
val leftDepth = depth(node.left)
val rightDepth = depth(node.right)
diameter = maxOf(diameter, leftDepth + rightDepth)
// This check happens for EVERY node during the recursion, not just once
// at the top level — the global maximum could be hiding anywhere
```

**The function still returns depth, not diameter** — this is what lets
the recursion work correctly. The "diameter so far" is tracked as a side
effect (Approach 1) or threaded through as extra return data (Approach 2),
while the actual return value powers the parent's own depth calculation:
```kotlin
return 1 + maxOf(leftDepth, rightDepth)
// the PARENT needs depth, not diameter, to compute ITS OWN diameter check
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Captured mutable variable (Approach 1) | Simpler, most commonly seen, slightly less pure |
| Return Pair of (depth, diameter) (Approach 2) | Want no mutable state, more functional style |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — single DFS pass, every node visited once |
| **Space** | O(h) — recursion stack, h = tree height |

---

## Key Takeaway

> The longest path through a node equals the sum of its two subtrees'
> depths. The tree's diameter is the maximum of this sum across **every**
> node — not just the root — because the longest overall path might be
> entirely contained within some subtree. Compute depth recursively as
> usual, but piggyback a diameter check at each step. This "compute one
> thing while tracking a global best as a side effect during the same
> traversal" pattern reappears in Binary Tree Maximum Path Sum later in
> this phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/diameter-of-binary-tree/)

---

{% include kotlin-dsa-nav.html %}
