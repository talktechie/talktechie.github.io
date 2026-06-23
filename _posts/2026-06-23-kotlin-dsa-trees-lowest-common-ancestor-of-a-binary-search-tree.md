---
title: "Trees: Lowest Common Ancestor of a Binary Search Tree — Kotlin Solution"
date: 2026-06-21
categories: [Kotlin DSA, Trees]
tags: [lowest-common-ancestor-bst, tree, bst, dfs, recursion, medium, leetcode, leetcode-235]
leetcode_number: 235
leetcode_url: https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/
difficulty: Medium
permalink: /posts/kotlin-dsa-trees-lowest-common-ancestor-of-a-binary-search-tree/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [235 — Lowest Common Ancestor of a Binary Search Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/) |
| **Difficulty** | Medium |
| **Topic** | Tree, BST, DFS, Recursion |

> Given a binary search tree (BST) and two nodes `p` and `q`, find their
> **lowest common ancestor (LCA)** — the deepest node that has both `p`
> and `q` as descendants (a node can be its own descendant).

**Example:**
```
Input:        6
            /    \
           2      8
          / \    / \
         0   4  7   9
            / \
           3   5
p=2, q=8
Output: 6
Explanation: 6 is the LCA since p and q are on different sides of it

p=2, q=4
Output: 2
Explanation: 2 is an ancestor of 4, and a node can be its own descendant
```

**Constraints:**
- Nodes: `[2, 10⁵]`
- `-10⁹ <= Node.val <= 10⁹`
- All `Node.val` are unique
- `p != q`, and both exist in the tree

---

## Approach

A **regular** binary tree's LCA (LC 236, a Hard-tier problem) requires
checking both subtrees with no shortcuts. A **BST**'s ordering property
makes this dramatically simpler.

**Key insight:** In a BST, every node's left subtree contains only
smaller values, and the right subtree only larger ones. This means we can
**compare values directly** to decide which way to go, without searching
both sides:
- If both `p` and `q` are smaller than the current node → LCA is in the
  left subtree
- If both are larger → LCA is in the right subtree
- Otherwise (they're on different sides, or one equals the current node)
  → the **current node** is the LCA — this is the split point

**Walk through `p=2, q=8` starting at root `6`:**
```
At node 6: p=2 < 6, q=8 > 6 → p and q are on DIFFERENT sides
           → 6 is the LCA (the split point) ✓
```

**Walk through `p=2, q=4` starting at root `6`:**
```
At node 6: p=2 < 6, q=4 < 6 → both smaller → go left
At node 2: p=2 == 2, q=4 > 2 → p equals current node → 2 is the LCA ✓
```

---

## Kotlin Solution

### Approach 1 — Iterative, using BST ordering (optimal, O(1) space)

```kotlin
fun lowestCommonAncestor(root: TreeNode?, p: TreeNode?, q: TreeNode?): TreeNode? {
    var node = root

    while (node != null) {
        when {
            p!!.`val` < node.`val` && q!!.`val` < node.`val` -> node = node.left
            p.`val` > node.`val` && q.`val` > node.`val` -> node = node.right
            else -> return node   // split point found — this is the LCA
        }
    }

    return null  // unreachable given problem guarantees, but keeps the compiler happy
}
```

### Approach 2 — Recursive

```kotlin
fun lowestCommonAncestor(root: TreeNode?, p: TreeNode?, q: TreeNode?): TreeNode? {
    if (root == null) return null

    return when {
        p!!.`val` < root.`val` && q!!.`val` < root.`val` -> lowestCommonAncestor(root.left, p, q)
        p.`val` > root.`val` && q.`val` > root.`val` -> lowestCommonAncestor(root.right, p, q)
        else -> root
    }
}
```

Same logic, expressed recursively. Slightly more elegant to read, at the
cost of O(h) call stack space versus the iterative version's O(1).

---

## Why BST Ordering Eliminates the Need to Search Both Sides

In a general binary tree, finding the LCA requires checking **both**
left and right subtrees at every node, because there's no information
about where any value "should" be — you'd need O(n) in the worst case,
visiting every node.

In a BST, comparing `p.val` and `q.val` against the current node's value
tells you **definitively** which subtree (if any) could possibly contain
both — no need to search the other side at all:

```kotlin
p.`val` < node.`val` && q.`val` < node.`val` -> node = node.left
// If BOTH values are smaller, the LCA can only be in the left subtree —
// the right subtree is guaranteed to contain neither p nor q
```

**The `else` branch covers two distinct cases** that both mean "we've
found the split point":
```kotlin
else -> return node
// Case A: p and q are on different sides (one smaller, one larger)
// Case B: node.val equals p.val or q.val itself (since a node can be
//         its own ancestor per the problem statement)
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Iterative (Approach 1) | Always preferred — O(1) space, no stack overflow risk even on deep trees |
| Recursive (Approach 2) | Slightly more readable, fine for typical balanced BST depths |

---

## Complexity

| | |
|---|---|
| **Time** | O(h) — h = tree height; O(log n) for a balanced BST, O(n) worst case for a skewed one |
| **Space** | O(1) iterative / O(h) recursive |

---

## Key Takeaway

> A BST's ordering invariant turns an O(n) general-tree search problem
> into an O(h) value-comparison walk. At every node, ask "are both targets
> on the same side?" — if yes, descend that way; if no, you've found the
> split point, which is the LCA. This value-comparison technique is
> specific to BSTs and is a great example of how structural guarantees
> (sorted order) can dramatically simplify what looks like a general
> tree-traversal problem.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)

---

{% include kotlin-dsa-nav.html %}
