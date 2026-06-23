---
title: "Trees: Construct Binary Tree From Preorder And Inorder Traversal — Kotlin Solution"
date: 2026-06-21
categories: [Kotlin DSA, Trees]
tags: [construct-binary-tree-preorder-inorder, tree, dfs, hashmap, recursion, medium, leetcode, leetcode-105]
leetcode_number: 105
leetcode_url: https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/
difficulty: Medium
permalink: /posts/kotlin-dsa-trees-construct-binary-tree-from-preorder-and-inorder-traversal/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [105 — Construct Binary Tree From Preorder And Inorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/) |
| **Difficulty** | Medium |
| **Topic** | Tree, DFS, HashMap, Recursion |

> Given two integer arrays `preorder` and `inorder` representing the
> preorder and inorder traversal of a binary tree (with unique values),
> construct and return the binary tree.

**Example:**
```
Input:  preorder = [3,9,20,15,7], inorder = [9,3,15,20,7]
Output:     3
          /   \
         9     20
              /   \
             15    7

Input:  preorder = [-1], inorder = [-1]
Output: [-1]
```

**Constraints:**
- `1 <= preorder.length <= 3000`
- `inorder.length == preorder.length`
- All values are unique

---

## Approach

**Key insight:** Preorder visits `[root, ...left subtree, ...right subtree]`
— so **the very first element of `preorder` is always the root**. Once we
know the root's value, we find that same value in `inorder` — everything
to its **left** in `inorder` belongs to the left subtree, everything to
its **right** belongs to the right subtree. The sizes of those two
partitions tell us exactly how to slice the remaining `preorder` array
for the recursive calls.

**Walk through `preorder = [3,9,20,15,7], inorder = [9,3,15,20,7]`:**
```
preorder[0] = 3 → root value is 3
Find 3 in inorder → index 1
  → left subtree's inorder slice: [9]           (1 element, indices 0..0)
  → right subtree's inorder slice: [15,20,7]     (3 elements, indices 2..4)

Left subtree uses next 1 element of preorder after the root: [9]
Right subtree uses the remaining 3 elements: [20,15,7]

Recurse left:  preorder=[9], inorder=[9] → single node "9"
Recurse right: preorder=[20,15,7], inorder=[15,20,7]
  preorder[0]=20 → root of this subtree
  find 20 in inorder=[15,20,7] → index 1
    left slice: [15] (1 element)   right slice: [7] (1 element)
  preorder for left: [15] → node "15"
  preorder for right: [7] → node "7"
  → 20's children: left=15, right=7

Final tree:     3
              /   \
             9     20
                  /   \
                 15    7   ✓
```

---

## Kotlin Solution

### Approach 1 — HashMap for O(1) inorder index lookup, shared mutable preorder pointer (optimal)

```kotlin
fun buildTree(preorder: IntArray, inorder: IntArray): TreeNode? {
    val inorderIndex = HashMap<Int, Int>()
    for (i in inorder.indices) inorderIndex[inorder[i]] = i

    var preIdx = 0

    fun build(inLeft: Int, inRight: Int): TreeNode? {
        if (inLeft > inRight) return null

        val rootVal = preorder[preIdx++]
        val root = TreeNode(rootVal)

        val mid = inorderIndex[rootVal]!!

        root.left = build(inLeft, mid - 1)
        root.right = build(mid + 1, inRight)

        return root
    }

    return build(0, inorder.lastIndex)
}
```

### Approach 2 — No HashMap, linear search for root in inorder (simpler, slower)

```kotlin
fun buildTree(preorder: IntArray, inorder: IntArray): TreeNode? {
    var preIdx = 0

    fun build(inLeft: Int, inRight: Int): TreeNode? {
        if (inLeft > inRight) return null

        val rootVal = preorder[preIdx++]
        val root = TreeNode(rootVal)

        var mid = inLeft
        while (inorder[mid] != rootVal) mid++

        root.left = build(inLeft, mid - 1)
        root.right = build(mid + 1, inRight)

        return root
    }

    return build(0, inorder.lastIndex)
}
```

Skips the HashMap, but the linear search for `rootVal` inside `inorder`
makes this O(n²) overall in the worst case — useful for understanding the
core idea, but the HashMap version is the one to actually use.

---

## Why `preIdx` Must Be Shared, Mutable State (Not a Parameter)

This is the trickiest detail in the implementation. `preorder` is
consumed **strictly left to right** across the *entire* construction,
regardless of which subtree we're currently building — the next
unconsumed preorder element is always the next root, whether it belongs
to a left or right subtree.

```kotlin
var preIdx = 0   // declared OUTSIDE the recursive function — shared across all calls

fun build(inLeft: Int, inRight: Int): TreeNode? {
    // ...
    val rootVal = preorder[preIdx++]   // consume the next preorder value globally
    // ...
    root.left = build(inLeft, mid - 1)    // this call may consume several preorder elements
    root.right = build(mid + 1, inRight)  // this call picks up exactly where left subtree left off
}
```

If `preIdx` were passed as a parameter and returned/threaded explicitly,
it would work too, but the shared mutable closure variable is simpler to
write correctly in Kotlin.

**The HashMap trades O(n) space for O(1) lookups**, turning what would
otherwise be an O(n) search (finding `rootVal` in `inorder`) into a
single map access — this is what brings the total complexity down from
O(n²) to O(n):
```kotlin
val inorderIndex = HashMap<Int, Int>()
for (i in inorder.indices) inorderIndex[inorder[i]] = i
// built once, used n times — O(n) total instead of O(n) per lookup × n lookups
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| HashMap lookup (Approach 1) | Always — O(n) overall, the correct production approach |
| Linear search (Approach 2) | Building intuition only; O(n²) worst case |

---

## Complexity

| | Approach 1 | Approach 2 |
|---|---|---|
| **Time** | O(n) | O(n²) worst case |
| **Space** | O(n) — HashMap + recursion stack | O(h) — recursion stack only |

---

## Key Takeaway

> Preorder tells you the root first; inorder tells you how to split into
> left and right subtrees once you know the root. Combining both
> traversals lets you reconstruct the tree uniquely (given unique node
> values). The shared, globally-advancing `preIdx` pointer is the key
> implementation insight — preorder is consumed in one continuous left-
> to-right sweep across the *entire* recursion, never reset or
> re-partitioned per subtree.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

---

{% include kotlin-dsa-nav.html %}
