---
title: "Trees: Maximum Depth of Binary Tree — Kotlin Solution"
date: 2026-06-21
categories: [Kotlin DSA, Trees]
tags: [maximum-depth-of-binary-tree, tree, dfs, bfs, recursion, easy, leetcode, leetcode-104]
leetcode_number: 104
leetcode_url: https://leetcode.com/problems/maximum-depth-of-binary-tree/
difficulty: Easy
permalink: /posts/kotlin-dsa-trees-maximum-depth-of-binary-tree/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [104 — Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/) |
| **Difficulty** | Easy |
| **Topic** | Tree, DFS, BFS, Recursion |

> Given the root of a binary tree, return its **maximum depth** — the
> number of nodes along the longest path from the root down to the
> farthest leaf.

**Example:**
```
Input:      3
          /   \
         9     20
              /   \
             15    7
Output: 3

Input:  [1,null,2]
Output: 2
```

**Constraints:**
- The number of nodes is in the range `[0, 10⁴]`
- `-100 <= Node.val <= 100`

---

## Approach

**Key insight:** The depth of a tree rooted at any node is `1 + the larger
of its two subtrees' depths`. A `null` node has depth 0. This recursive
definition translates directly into code with almost no extra logic.

**Walk through `[3,9,20,null,null,15,7]`:**
```
depth(3):
  depth(9) = 1 + max(depth(null), depth(null)) = 1 + max(0,0) = 1
  depth(20):
    depth(15) = 1 + max(0,0) = 1
    depth(7)  = 1 + max(0,0) = 1
    depth(20) = 1 + max(1,1) = 2
  depth(3) = 1 + max(1, 2) = 3

Answer: 3 ✓
```

---

## Kotlin Solution

### Approach 1 — Recursive DFS (optimal, most common)

```kotlin
fun maxDepth(root: TreeNode?): Int {
    if (root == null) return 0
    return 1 + maxOf(maxDepth(root.left), maxDepth(root.right))
}
```

### Approach 2 — Iterative BFS, level by level

```kotlin
fun maxDepth(root: TreeNode?): Int {
    if (root == null) return 0

    val queue: ArrayDeque<TreeNode> = ArrayDeque()
    queue.addLast(root)
    var depth = 0

    while (queue.isNotEmpty()) {
        val levelSize = queue.size
        repeat(levelSize) {
            val node = queue.removeFirst()
            node.left?.let { queue.addLast(it) }
            node.right?.let { queue.addLast(it) }
        }
        depth++
    }

    return depth
}
```

Processes one full level at a time — `depth` increments once per level,
naturally counting the total number of levels.

### Approach 3 — Iterative DFS with an explicit stack

```kotlin
fun maxDepth(root: TreeNode?): Int {
    if (root == null) return 0

    val stack = ArrayDeque<Pair<TreeNode, Int>>()
    stack.addLast(root to 1)
    var maxDepth = 0

    while (stack.isNotEmpty()) {
        val (node, depth) = stack.removeLast()
        maxDepth = maxOf(maxDepth, depth)

        node.left?.let { stack.addLast(it to depth + 1) }
        node.right?.let { stack.addLast(it to depth + 1) }
    }

    return maxDepth
}
```

Tracks depth alongside each node explicitly in the stack, rather than
relying on the call stack to encode it implicitly.

---

## Why `1 + max(left, right)` Is the Entire Solution

Depth is inherently a "look at both children, take the better one, add
one for the current level" computation — there's no extra bookkeeping
needed beyond what recursion already gives for free:

```kotlin
if (root == null) return 0   // empty subtree contributes 0 depth
return 1 + maxOf(maxDepth(root.left), maxDepth(root.right))
// +1 accounts for the current node itself
```

**The level-by-level BFS approach (Approach 2) snapshots `queue.size`
before the inner loop** — this is essential, because the queue keeps
growing as children are added during the loop, and we only want to
process exactly the nodes that existed in the current level:
```kotlin
val levelSize = queue.size   // snapshot BEFORE adding any children
repeat(levelSize) { ... }     // process exactly that many nodes this round
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Recursive DFS (Approach 1) | Always — simplest, most idiomatic |
| Iterative BFS (Approach 2) | Want explicit level tracking, or avoiding recursion |
| Iterative DFS with stack (Approach 3) | Want DFS order without recursion stack risk |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — every node visited once |
| **Space** | O(h) recursive / O(w) BFS (w = max width) |

---

## Key Takeaway

> Depth is a textbook "combine results from both children" recursion:
> `1 + max(left depth, right depth)`, with `null` as the natural base
> case returning 0. This three-line recursive solution is worth
> memorizing exactly as-is — it's the seed pattern for Balanced Binary
> Tree, Diameter of Binary Tree, and many other problems later in this
> phase that build on "depth of a subtree" as a sub-computation.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/maximum-depth-of-binary-tree/)

---

{% include kotlin-dsa-nav.html %}
