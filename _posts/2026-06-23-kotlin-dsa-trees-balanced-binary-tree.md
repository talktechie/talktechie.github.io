---
title: "Trees: Balanced Binary Tree — Kotlin Solution"
date: 2026-06-21
categories: [Kotlin DSA, Trees]
tags: [balanced-binary-tree, tree, dfs, recursion, easy, leetcode, leetcode-110]
leetcode_number: 110
leetcode_url: https://leetcode.com/problems/balanced-binary-tree/
difficulty: Easy
permalink: /posts/kotlin-dsa-trees-balanced-binary-tree/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [110 — Balanced Binary Tree](https://leetcode.com/problems/balanced-binary-tree/) |
| **Difficulty** | Easy |
| **Topic** | Tree, DFS, Recursion |

> Given a binary tree, determine if it is **height-balanced** — for
> every node, the height difference between its left and right subtrees
> is at most 1.

**Example:**
```
Input:      3
          /   \
         9     20
              /   \
             15    7
Output: true

Input:      1
          /   \
         2     2
        / \
       3   3
      / \
     4   4
Output: false
Explanation: at node 2 (left side), the subtrees have heights 2 and 0 — 
difference of 2, exceeds the allowed 1
```

**Constraints:**
- The number of nodes is in the range `[0, 5000]`
- `-10⁴ <= Node.val <= 10⁴`

---

## Approach

The naive approach computes depth at every node, and separately checks
balance at every node — calling the depth function repeatedly leads to
O(n²) in the worst case (computing depth from scratch at every level).

**Key insight — compute depth and check balance in the same pass:**
While computing a subtree's depth, simultaneously check if it's balanced.
If any subtree is found unbalanced, **short-circuit** — propagate a
sentinel value (like `-1`) upward immediately instead of continuing to
compute meaningless depths.

**Walk through the unbalanced example `[1,2,2,3,3,null,null,4,4]`:**
```
depth(4) = 0 (leaf), depth(4) = 0 (leaf)
At node 3 (left): |0-0|<=1 ✓ balanced so far → depth = 1
At node 3 (right, the other one): leaf-like, depth = 0

At node 2 (left side): leftDepth=depth(3 with children)=1, rightDepth=depth(3 leaf)=0
                        |1-0|=1 <=1 ✓ still balanced → depth = 2

Wait — let's recheck the actual structure: node 2(left) has children 3,3.
The left "3" has children 4,4 (depth 1), the right "3" is a leaf (depth 0).
|1-0| = 1, balanced, depth(2-left) = 2

At node 2 (right side, leaf): depth = 0

At root 1: leftDepth=depth(2-left)=2, rightDepth=depth(2-right)=0
           |2-0| = 2 > 1 → UNBALANCED → return false ✓
```

---

## Kotlin Solution

### Approach 1 — Single DFS, sentinel value for early exit (optimal)

```kotlin
fun isBalanced(root: TreeNode?): Boolean {
    fun checkHeight(node: TreeNode?): Int {
        if (node == null) return 0

        val leftHeight = checkHeight(node.left)
        if (leftHeight == -1) return -1   // left subtree already unbalanced — bail out

        val rightHeight = checkHeight(node.right)
        if (rightHeight == -1) return -1  // right subtree already unbalanced — bail out

        if (Math.abs(leftHeight - rightHeight) > 1) return -1  // this node is unbalanced

        return 1 + maxOf(leftHeight, rightHeight)
    }

    return checkHeight(root) != -1
}
```

### Approach 2 — Naive O(n²): separate depth function called repeatedly

```kotlin
fun isBalanced(root: TreeNode?): Boolean {
    fun depth(node: TreeNode?): Int {
        if (node == null) return 0
        return 1 + maxOf(depth(node.left), depth(node.right))
    }

    fun check(node: TreeNode?): Boolean {
        if (node == null) return true

        val diff = Math.abs(depth(node.left) - depth(node.right))
        return diff <= 1 && check(node.left) && check(node.right)
    }

    return check(root)
}
```

Correct, but `depth()` re-traverses entire subtrees repeatedly from every
node in `check()` — O(n) work done O(n) times, giving O(n²) overall on
unbalanced/skewed trees. Shown only to highlight what *not* to do.

---

## Why `-1` as a Sentinel Avoids Wasted Work

This is the central optimization. Once any subtree is found unbalanced,
there's no point continuing to compute heights elsewhere — the answer is
already determined to be `false`. Using `-1` (an impossible real height
value, since heights are always ≥ 0) lets that signal propagate upward
through ordinary integer returns, with no extra boolean flag needed:

```kotlin
val leftHeight = checkHeight(node.left)
if (leftHeight == -1) return -1   // propagate failure immediately, skip rightHeight entirely
```

This early-exit is what brings the complexity down from O(n²) to O(n) —
each node's height is computed **exactly once**, and unbalanced subtrees
short-circuit the rest of the traversal above them.

**Why check `leftHeight` before computing `rightHeight`:**
```kotlin
val leftHeight = checkHeight(node.left)
if (leftHeight == -1) return -1
val rightHeight = checkHeight(node.right)  // only computed if left side is still OK
```
No correctness difference either way, but this ordering avoids unnecessary
work once a failure is already known.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Sentinel value, single pass (Approach 1) | Always — true O(n), the correct production answer |
| Naive separate depth calls (Approach 2) | Never for submission — included only to illustrate the O(n²) pitfall |

---

## Complexity

| | Approach 1 | Approach 2 |
|---|---|---|
| **Time** | O(n) | O(n²) worst case |
| **Space** | O(h) | O(h) |

---

## Key Takeaway

> When a tree problem needs both "compute X" and "validate a condition
> about X" at every node, do both in the **same** traversal rather than
> calling a helper function repeatedly — that's the difference between
> O(n) and O(n²). A sentinel value like `-1` lets a single integer return
> type carry both "the height" and "already failed" information, enabling
> clean early-exit propagation up the call stack.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/balanced-binary-tree/)

---

{% include kotlin-dsa-nav.html %}
