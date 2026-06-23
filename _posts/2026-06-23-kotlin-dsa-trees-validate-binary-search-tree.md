---
title: "Trees: Validate Binary Search Tree — Kotlin Solution"
date: 2026-06-21
categories: [Kotlin DSA, Trees]
tags: [validate-binary-search-tree, tree, bst, dfs, recursion, medium, leetcode, leetcode-98]
leetcode_number: 98
leetcode_url: https://leetcode.com/problems/validate-binary-search-tree/
difficulty: Medium
permalink: /posts/kotlin-dsa-trees-validate-binary-search-tree/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [98 — Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/) |
| **Difficulty** | Medium |
| **Topic** | Tree, BST, DFS, Recursion |

> Given the root of a binary tree, determine if it is a **valid binary
> search tree (BST)**.
>
> A valid BST requires, for every node: all values in the left subtree
> are strictly less than the node's value, all values in the right
> subtree are strictly greater, and both subtrees are themselves valid BSTs.

**Example:**
```
Input:      2
          /   \
         1     3
Output: true

Input:      5
          /   \
         1     4
              / \
             3   6
Output: false
Explanation: node 3 is in the right subtree of 5, so it must be > 5,
             but 3 < 5
```

**Constraints:**
- Nodes: `[1, 10⁴]`
- `-2³¹ <= Node.val <= 2³¹ - 1`

---

## Approach

The common mistake: checking only `node.left.val < node.val < node.right.val`
locally at each node. That's **not enough** — a node deep in the left
subtree must be less than **every** ancestor it's a left-descendant of,
not just its immediate parent.

**Key insight — pass down a valid range:** Each recursive call carries a
`(min, max)` bound that the current node's value must fall within. When
recursing left, the **max** bound tightens to the current node's value
(everything in the left subtree must be smaller). When recursing right,
the **min** bound tightens to the current node's value.

**Walk through the invalid example `[5,1,4,null,null,3,6]`:**
```
validate(5, min=-∞, max=+∞): 5 is within (-∞,+∞) ✓
  validate(1, min=-∞, max=5): 1 is within (-∞,5) ✓ (leaf, no children)
  validate(4, min=5, max=+∞): 4 is within (5,+∞)? NO — 4 < 5 → INVALID

Answer: false ✓ (correctly caught — even though 4's own children 3,6
                  might look locally fine, 4 itself violates being in
                  the right subtree of 5)
```

---

## Kotlin Solution

### Approach 1 — DFS with min/max bounds passed down (optimal)

```kotlin
fun isValidBST(root: TreeNode?): Boolean {
    fun validate(node: TreeNode?, min: Long, max: Long): Boolean {
        if (node == null) return true

        if (node.`val` <= min || node.`val` >= max) return false

        return validate(node.left, min, node.`val`.toLong()) &&
               validate(node.right, node.`val`.toLong(), max)
    }

    return validate(root, Long.MIN_VALUE, Long.MAX_VALUE)
}
```

### Approach 2 — Inorder traversal must be strictly increasing

```kotlin
fun isValidBST(root: TreeNode?): Boolean {
    var prev: Long? = null
    var isValid = true

    fun inorder(node: TreeNode?) {
        if (node == null || !isValid) return

        inorder(node.left)

        if (prev != null && node.`val` <= prev!!) {
            isValid = false
            return
        }
        prev = node.`val`.toLong()

        inorder(node.right)
    }

    inorder(root)
    return isValid
}
```

A BST's inorder traversal visits nodes in strictly ascending order if and
only if the tree is valid. This approach leans on that property directly
instead of tracking explicit bounds.

---

## Why `Long` Bounds (Not `Int`) Avoid an Overflow Trap

```kotlin
validate(root, Long.MIN_VALUE, Long.MAX_VALUE)
```

The constraint allows `Node.val` to be as extreme as `Int.MIN_VALUE` or
`Int.MAX_VALUE`. If we used `Int.MIN_VALUE`/`Int.MAX_VALUE` as our initial
bounds and a node's actual value happened to equal one of those extremes,
the strict inequality check (`<=` / `>=`) could misbehave at the boundary.
Using `Long` for the bounds sidesteps this entirely — there's always
room beyond any valid `Int` value.

**Why bounds tighten in only one direction per recursive call:**
```kotlin
validate(node.left, min, node.`val`.toLong())   // max tightens, min unchanged
validate(node.right, node.`val`.toLong(), max)  // min tightens, max unchanged
```
Going left only adds an upper constraint (everything must be less than
the current node) — the lower bound inherited from further up the tree
still applies unchanged. Going right is the mirror image.

**Why inorder traversal validity (Approach 2) is equivalent:** a BST's
defining property — left < node < right, recursively — is exactly the
condition that produces a sorted sequence when visited in left-root-right
order. If the sequence isn't strictly increasing, the BST property must
have been violated somewhere.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Min/max bounds (Approach 1) | Always — directly encodes the BST definition, easy to explain |
| Inorder traversal check (Approach 2) | Elegant alternative, useful if you already think in terms of inorder properties |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — every node visited once |
| **Space** | O(h) — recursion stack |

---

## Key Takeaway

> Checking only immediate parent-child relationships is the classic trap
> in this problem — BST validity is a **global** constraint that must
> account for every ancestor, not just the direct parent. Threading a
> `(min, max)` range down through recursion, tightened appropriately at
> each left/right step, correctly encodes that global constraint. This
> bounds-passing technique is the same "carry context downward" idea
> from Count Good Nodes, applied to range constraints instead of a
> running maximum.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/validate-binary-search-tree/)

---

{% include kotlin-dsa-nav.html %}
