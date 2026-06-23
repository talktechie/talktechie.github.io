---
title: "Trees: Kth Smallest Element In a BST — Kotlin Solution"
date: 2026-06-21
categories: [Kotlin DSA, Trees]
tags: [kth-smallest-element-bst, tree, bst, dfs, recursion, medium, leetcode, leetcode-230]
leetcode_number: 230
leetcode_url: https://leetcode.com/problems/kth-smallest-element-in-a-bst/
difficulty: Medium
permalink: /posts/kotlin-dsa-trees-kth-smallest-element-in-a-bst/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [230 — Kth Smallest Element In a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/) |
| **Difficulty** | Medium |
| **Topic** | Tree, BST, DFS, Recursion |

> Given the root of a binary search tree and an integer `k`, return the
> `k`-th **smallest** value in the tree (1-indexed).

**Example:**
```
Input:      3
          /   \
         1     4
          \
           2
k = 1
Output: 1

Input:      5
          /   \
         3     6
        / \
       2   4
      /
     1
k = 3
Output: 3
```

**Constraints:**
- Nodes: `[1, 10⁴]`
- `0 <= Node.val <= 10⁴`
- **Follow-up:** if the BST is modified often and you must find the kth
  smallest frequently, how would you optimize?

---

## Approach

**Key insight — inorder traversal of a BST visits nodes in sorted order.**
That's the entire trick: do an inorder traversal (left, node, right), and
the `k`-th value visited is the answer. No sorting needed — the BST's
structure already gives us sorted order for free.

**Walk through `[5,3,6,2,4,1], k = 3`:**
```
Inorder traversal order: 1, 2, 3, 4, 5, 6
                          ↑  ↑  ↑
                         1st 2nd 3rd ← k=3, answer is 3

Answer: 3 ✓
```

---

## Kotlin Solution

### Approach 1 — Iterative inorder traversal with explicit stack, early exit (optimal)

```kotlin
fun kthSmallest(root: TreeNode?, k: Int): Int {
    val stack = ArrayDeque<TreeNode>()
    var curr = root
    var count = 0

    while (curr != null || stack.isNotEmpty()) {
        while (curr != null) {
            stack.addLast(curr)
            curr = curr.left
        }

        curr = stack.removeLast()
        count++

        if (count == k) return curr.`val`

        curr = curr.right
    }

    throw IllegalArgumentException("k is larger than the number of nodes")
}
```

Stops as soon as the `k`-th node is found — no need to traverse the
entire tree if `k` is small relative to tree size.

### Approach 2 — Recursive inorder, collect into a list

```kotlin
fun kthSmallest(root: TreeNode?, k: Int): Int {
    val values = mutableListOf<Int>()

    fun inorder(node: TreeNode?) {
        if (node == null) return
        inorder(node.left)
        values.add(node.`val`)
        inorder(node.right)
    }

    inorder(root)
    return values[k - 1]
}
```

Simple and readable, but always visits the entire tree even if `k` is
small — less efficient than Approach 1's early exit, though still O(n)
overall.

---

## Why Iterative Inorder with Early Exit Is Preferred

The recursive version (Approach 2) is easier to write but **must**
traverse the whole tree, because the recursive calls have already
returned by the time we'd want to "stop early" — there's no clean way to
short-circuit deeply nested recursive calls without extra signaling.

The iterative version (Approach 1) processes nodes one at a time using
an explicit stack, mimicking what the call stack would do — but because
we control the loop directly, we can `return` the instant we find the
`k`-th node:

```kotlin
curr = stack.removeLast()
count++
if (count == k) return curr.`val`   // immediate exit — no need to visit the rest
```

**The "push left children, then process, then go right" pattern** is the
standard iterative inorder template:
```kotlin
while (curr != null) {
    stack.addLast(curr)
    curr = curr.left      // dive as far left as possible first
}
curr = stack.removeLast()  // process the smallest unvisited node
// ... then move to its right subtree on the next outer loop iteration
curr = curr.right
```

---

## Addressing the Follow-Up: Frequent Modifications

If the BST is modified (inserted/deleted) often and `kthSmallest` is
called repeatedly, recomputing from scratch every time is wasteful.
A common optimization: augment each node with a **count of nodes in its
left subtree**. Then `kthSmallest` becomes an O(h) descent — at each
node, compare `k` to the left subtree's count to decide whether to go
left, return the current node, or go right (adjusting `k` accordingly).
Maintaining this count requires updating it on every insert/delete, but
turns each query into O(h) instead of O(n).

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Iterative with early exit (Approach 1) | Always — can stop as soon as the answer is found |
| Recursive, collect all values (Approach 2) | Simpler to write, fine if k is close to n anyway |

---

## Complexity

| | Approach 1 | Approach 2 |
|---|---|---|
| **Time** | O(h + k) — h to reach the leftmost node, then k steps | O(n) — always visits every node |
| **Space** | O(h) | O(n) |

---

## Key Takeaway

> A BST's inorder traversal is sorted order — this single fact converts
> "find the kth smallest value" into "do an inorder traversal and stop at
> the kth step." The iterative version with an explicit stack is worth
> knowing well, since it's the only way to get the early-exit
> optimization (stopping as soon as the answer is found, rather than
> always processing the whole tree).

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/kth-smallest-element-in-a-bst/)

---

{% include kotlin-dsa-nav.html %}
