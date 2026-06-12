---
title: "Trees: Validate Binary Search Tree — Kotlin Solution"
date: 2026-06-12
categories: [Kotlin DSA, Trees]
tags: [validate-binary-search-tree, bst, dfs, inorder, medium, leetcode, leetcode-98]
leetcode_number: 98
leetcode_url: https://leetcode.com/problems/validate-binary-search-tree/
difficulty: Medium
permalink: /posts/kotlin-dsa-trees-validate-bst/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [98 — Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/) |
| **Difficulty** | Medium |
| **Topic** | Binary Tree, DFS, In-order Traversal |

> Given the root of a binary tree, determine if it is a valid BST.
>
> A valid BST satisfies:
> - Left subtree contains only nodes with values **less than** the node's value
> - Right subtree contains only nodes with values **greater than** the node's value
> - Both subtrees must also be valid BSTs

**Example:**
```
Valid:          Invalid:
    2               5
   / \             / \
  1   3           1   4
                     / \
                    3   6
// Node 4's left child 3 is < 5 but the rule applies to the WHOLE subtree
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

## The Common Mistake

Most people check only the immediate parent:

```kotlin
// ❌ Wrong — only checks direct parent, not entire subtree bounds
fun isValid(node: TreeNode?): Boolean {
    if (node == null) return true
    if (node.left != null && node.left!!.`val` >= node.`val`) return false
    if (node.right != null && node.right!!.`val` <= node.`val`) return false
    return isValid(node.left) && isValid(node.right)
}
```

This fails for:
```
    5
   / \
  1   4
     / \
    3   6
// 3 < 5 (passes local check) but 3 is in right subtree of 5 — invalid!
```

---

## Approach

Pass **min and max bounds** down the tree. Every node must satisfy:
`min < node.val < max`

- When going **left**, the current node's value becomes the new **max**
- When going **right**, the current node's value becomes the new **min**

**Walk through the invalid example:**
```
isValid(5, min=-∞, max=+∞)   → 5 is valid
  isValid(1, min=-∞, max=5)  → 1 is valid
  isValid(4, min=5, max=+∞)  → 4 < 5 — INVALID! ✗
```

---

## Kotlin Solution

### Approach 1 — Min/Max Bounds (recommended)

```kotlin
fun isValidBST(root: TreeNode?): Boolean {
    return validate(root, Long.MIN_VALUE, Long.MAX_VALUE)
}

private fun validate(node: TreeNode?, min: Long, max: Long): Boolean {
    if (node == null) return true
    if (node.`val` <= min || node.`val` >= max) return false

    return validate(node.left, min, node.`val`.toLong()) &&
           validate(node.right, node.`val`.toLong(), max)
}
```

### Approach 2 — In-order Traversal

A valid BST produces a strictly increasing sequence when traversed in-order.

```kotlin
fun isValidBST(root: TreeNode?): Boolean {
    var prev = Long.MIN_VALUE

    fun inorder(node: TreeNode?): Boolean {
        if (node == null) return true
        if (!inorder(node.left)) return false
        if (node.`val`.toLong() <= prev) return false
        prev = node.`val`.toLong()
        return inorder(node.right)
    }

    return inorder(root)
}
```

---

## Why Kotlin Shines Here

**`Long.MIN_VALUE` / `Long.MAX_VALUE`** — use `Long` not `Int` to handle
edge cases where node values equal `Int.MIN_VALUE` or `Int.MAX_VALUE`:
```kotlin
validate(root, Long.MIN_VALUE, Long.MAX_VALUE)
// If you use Int.MIN_VALUE and a node has value Int.MIN_VALUE,
// the check node.val <= min would wrongly fail
```

**Local function `inorder`** — Kotlin lets you define a function inside
another function, keeping `prev` in scope without a class field:
```kotlin
fun isValidBST(root: TreeNode?): Boolean {
    var prev = Long.MIN_VALUE
    fun inorder(node: TreeNode?): Boolean { ... }  // captures prev
    return inorder(root)
}
// vs Java: needs a class-level field or wrapper class
```

**`&&` short-circuits** — right subtree only validated if left is valid:
```kotlin
return validate(node.left, min, node.`val`.toLong()) &&
       validate(node.right, node.`val`.toLong(), max)
```

---

## Two Approaches Compared

| Approach | Time | Space | Notes |
|---|---|---|---|
| Min/Max bounds | O(n) | O(h) | Explicit, easy to reason about |
| In-order traversal | O(n) | O(h) | Elegant — leverages BST property |

Both are equally valid in interviews. The **min/max approach** is easier
to explain. The **in-order approach** shows deeper BST understanding.

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — visit every node |
| **Space** | O(h) — recursion stack depth |

---

## Key Takeaway

> Don't just check the immediate parent. Pass min and max bounds down
> every branch. Every node must fit strictly within its inherited range.
> Use Long to avoid Int boundary edge cases.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/validate-binary-search-tree/)

---

{% include kotlin-dsa-nav.html %}
