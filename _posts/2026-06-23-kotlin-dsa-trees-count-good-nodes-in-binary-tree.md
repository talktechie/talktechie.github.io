---
title: "Trees: Count Good Nodes In Binary Tree — Kotlin Solution"
date: 2026-06-21
categories: [Kotlin DSA, Trees]
tags: [count-good-nodes-in-binary-tree, tree, dfs, recursion, medium, leetcode, leetcode-1448]
leetcode_number: 1448
leetcode_url: https://leetcode.com/problems/count-good-nodes-in-binary-tree/
difficulty: Medium
permalink: /posts/kotlin-dsa-trees-count-good-nodes-in-binary-tree/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [1448 — Count Good Nodes In Binary Tree](https://leetcode.com/problems/count-good-nodes-in-binary-tree/) |
| **Difficulty** | Medium |
| **Topic** | Tree, DFS, Recursion |

> Given a binary tree, a node `X` is **good** if, on the path from the
> root to `X`, no node has a value **greater than** `X`'s value.
>
> Return the number of good nodes.

**Example:**
```
Input:      3
          /   \
         1     4
        /     / \
       3     1   5
Output: 4
Explanation: nodes 3(root), 3(left-left), 4, 5 are good.
             Node 1 (left child of root) is NOT good — 3 > 1 on its path.
             Node 1 (left child of 4) is NOT good — 4 > 1 on its path.

Input:  [3,3,null,4,2]
Output: 3
```

**Constraints:**
- Nodes: `[1, 10⁵]`
- `Node.val` is in `[-10⁴, 10⁴]`

---

## Approach

**Key insight:** Pass the **maximum value seen so far** down through the
recursion as a parameter. At each node, compare its value to that running
maximum:
- If the current node's value ≥ the max seen so far, it's a **good node**
  — count it, and update the max passed down to children.
- Otherwise, it's not good, but the max passed down to children stays
  the same (the larger ancestor value still "blocks" descendants).

**Walk through `[3,1,4,3,null,1,5]`:**
```
dfs(node=3, maxSoFar=-∞): 3 >= -∞ → GOOD, count=1, pass maxSoFar=3 to children

dfs(node=1, maxSoFar=3): 1 < 3 → not good, pass maxSoFar=3 (unchanged) to children
  (node 1 has no children here in this shape)

dfs(node=4, maxSoFar=3): 4 >= 3 → GOOD, count=2, pass maxSoFar=4 to children

  dfs(node=1, maxSoFar=4): 1 < 4 → not good, pass maxSoFar=4 to children

  dfs(node=5, maxSoFar=4): 5 >= 4 → GOOD, count=3, pass maxSoFar=5 to children

Wait, the example tree also has a "3" as left child of root's left child — 
let's also count that:
dfs(node=3 [left-left grandchild], maxSoFar=3 from its parent path): 3>=3 → GOOD, count=4

Total good nodes: 4 ✓
```

---

## Kotlin Solution

### Approach 1 — DFS, thread max-so-far as a parameter (optimal)

```kotlin
fun goodNodes(root: TreeNode?): Int {
    fun dfs(node: TreeNode?, maxSoFar: Int): Int {
        if (node == null) return 0

        var count = 0
        val newMax: Int

        if (node.`val` >= maxSoFar) {
            count = 1
            newMax = node.`val`
        } else {
            newMax = maxSoFar
        }

        count += dfs(node.left, newMax)
        count += dfs(node.right, newMax)

        return count
    }

    return dfs(root, Int.MIN_VALUE)
}
```

### Approach 2 — DFS with a captured counter variable

```kotlin
fun goodNodes(root: TreeNode?): Int {
    var count = 0

    fun dfs(node: TreeNode?, maxSoFar: Int) {
        if (node == null) return

        val newMax = maxOf(maxSoFar, node.`val`)
        if (node.`val` >= maxSoFar) count++

        dfs(node.left, newMax)
        dfs(node.right, newMax)
    }

    dfs(root, Int.MIN_VALUE)
    return count
}
```

Functionally identical, but uses a captured mutable variable instead of
accumulating and returning counts explicitly — a matter of style
preference.

---

## Why `Int.MIN_VALUE` Is the Correct Starting Max

```kotlin
return dfs(root, Int.MIN_VALUE)
```

The root has no ancestors, so by definition there's no value on its path
that could be larger than it — the root is **always** a good node. Using
`Int.MIN_VALUE` as the initial "max seen so far" guarantees `node.val >=
maxSoFar` is true for the root regardless of what value it holds
(including negative values, given the constraint `-10⁴ <= Node.val`).

**Threading state down through parameters (not up through return
values)** is the key technique here, different from Diameter of Binary
Tree's "track a global best while returning depth" pattern:
```kotlin
dfs(node.left, newMax)   // newMax flows DOWN to children
dfs(node.right, newMax)
```
This is the "carry context down the tree" pattern — useful whenever a
node's evaluation depends on something accumulated from its ancestors,
rather than something computed from its descendants.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Return accumulated count (Approach 1) | More functional style, no mutable captured state |
| Captured counter variable (Approach 2) | Slightly simpler to write, common in practice |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — every node visited once |
| **Space** | O(h) — recursion stack |

---

## Key Takeaway

> Not every tree problem needs information flowing **up** from children
> to parents (like depth or diameter) — some need information flowing
> **down** from ancestors to descendants. Here, "the maximum value on the
> path so far" is exactly that kind of top-down context. Passing it as an
> extra recursion parameter, updated only when a new maximum is found, is
> the clean way to implement this "carry ancestor information downward"
> pattern.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/count-good-nodes-in-binary-tree/)

---

{% include kotlin-dsa-nav.html %}
