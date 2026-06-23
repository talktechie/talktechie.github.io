---
title: "Trees: Binary Tree Maximum Path Sum — Kotlin Solution"
date: 2026-06-21
categories: [Kotlin DSA, Trees]
tags: [binary-tree-maximum-path-sum, tree, dfs, recursion, hard, leetcode, leetcode-124]
leetcode_number: 124
leetcode_url: https://leetcode.com/problems/binary-tree-maximum-path-sum/
difficulty: Hard
permalink: /posts/kotlin-dsa-trees-binary-tree-maximum-path-sum/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [124 — Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/) |
| **Difficulty** | Hard |
| **Topic** | Tree, DFS, Recursion |

> Given the root of a binary tree, return the maximum **path sum** of any
> path. A path is any sequence of nodes connected by parent-child edges,
> with **no node repeated**, and it does **not** need to pass through the
> root.

**Example:**
```
Input:  [1,2,3]
Output: 6
Explanation: path 2 → 1 → 3, sum = 2+1+3 = 6

Input:  [-10,9,20,null,null,15,7]
Output: 42
Explanation: path 15 → 20 → 7, sum = 15+20+7 = 42
            (the -10 root and node 9 are excluded — including them
             would only decrease the sum)
```

**Constraints:**
- Nodes: `[1, 3 * 10⁴]`
- `-1000 <= Node.val <= 1000`

---

## Approach

This generalizes Diameter of Binary Tree — same "check at every node, but
return something different to the parent" pattern, with sums replacing
edge counts, and the added twist that negative values mean we sometimes
should **exclude** a subtree entirely.

**Key insight:** At each node, compute the best **single-direction**
extension achievable through it — `node.val + max(0, leftGain, rightGain)`.
This is what gets returned to the parent, since a path continuing through
the parent can only use **one** branch below the current node (a path
can't fork at the parent and still be a valid connected path).

But the actual best path **through** the current node (potentially the
final answer) might use **both** branches — `node.val + leftGain +
rightGain`. We compute this at every node as a side effect and track the
global maximum, exactly like Diameter of Binary Tree's diameter tracking.

The `max(0, ...)` clamp is what handles negative subtrees — if a branch's
best extension is negative, we simply don't take it (treat it as 0,
meaning "don't extend this direction at all").

**Walk through `[-10,9,20,null,null,15,7]`:**
```
At node 15 (leaf): leftGain=0, rightGain=0 → gain=15+0=15
                    path-through-15 = 15+0+0=15 → global max candidate: 15
At node 7  (leaf): gain=7 → path-through-7=7 → global max candidate: still 15

At node 20: leftGain=gain(15)=15, rightGain=gain(7)=7
            gain = 20 + max(0, max(15,7)) = 20+15 = 35
            path-through-20 = 20+15+7 = 42 → global max candidate: 42

At node 9 (leaf): gain=9 → path-through-9=9 → global max stays 42

At node -10 (root): leftGain=gain(9)=9, rightGain=gain(20)=35
                     gain = -10 + max(0, max(9,35)) = -10+35 = 25
                     path-through-root = -10+9+35 = 34 → doesn't beat 42

Global maximum: 42 ✓
```

---

## Kotlin Solution

### Approach 1 — Single DFS, track global max as a side effect (optimal)

```kotlin
fun maxPathSum(root: TreeNode?): Int {
    var maxSum = Int.MIN_VALUE

    fun gain(node: TreeNode?): Int {
        if (node == null) return 0

        val leftGain = maxOf(0, gain(node.left))    // ignore negative branches
        val rightGain = maxOf(0, gain(node.right))

        // Best path THROUGH this node, using both branches — candidate for global answer
        maxSum = maxOf(maxSum, node.`val` + leftGain + rightGain)

        // Best SINGLE-direction extension to return to the parent
        return node.`val` + maxOf(leftGain, rightGain)
    }

    gain(root)
    return maxSum
}
```

### Approach 2 — Return Pair of (bestSingleBranch, bestPathOverall), no captured variable

```kotlin
fun maxPathSum(root: TreeNode?): Int {
    fun helper(node: TreeNode?): Pair<Int, Int> {  // (singleBranchGain, bestPathInSubtree)
        if (node == null) return 0 to Int.MIN_VALUE

        val (leftGain, leftBest) = helper(node.left)
        val (rightGain, rightBest) = helper(node.right)

        val leftClamped = maxOf(0, leftGain)
        val rightClamped = maxOf(0, rightGain)

        val gain = node.`val` + maxOf(leftClamped, rightClamped)
        val throughNode = node.`val` + leftClamped + rightClamped

        val bestHere = maxOf(throughNode, leftBest, rightBest)

        return gain to bestHere
    }

    return helper(root).second
}
```

Avoids a mutable captured variable by threading both pieces of
information through return values — more verbose, but a purer style
consistent with the alternative approach shown for Diameter of Binary Tree.

---

## Why `max(0, gain)` Is the Crucial Clamp

Without this clamp, a subtree with an overwhelmingly negative sum would
still get "added" to its parent's calculation, dragging the result down
when it would clearly be better to just not include that subtree at all:

```kotlin
val leftGain = maxOf(0, gain(node.left))
// If gain(node.left) is -50, we treat it as 0 instead — meaning
// "don't extend the path into the left subtree, it only hurts"
```

**Why the function returns only the single-branch gain, not the best
path overall:** the value returned to a parent represents "if you want
to extend a path through me into your own path, here's the best you can
add" — and a valid path can only continue in **one** direction past any
given node (you can't have a path that forks). The "both branches" case
is only valid as a path that **ends** at the current node, which is why
it's tracked separately as a candidate for the global maximum, never
returned upward.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Captured mutable max (Approach 1) | Simpler, most commonly seen in practice |
| Return Pair, no mutable state (Approach 2) | Prefer pure functions, consistent style with Diameter's alternative |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — single DFS pass |
| **Space** | O(h) — recursion stack |

---

## Key Takeaway

> This is Diameter of Binary Tree's pattern — "track a global best while
> returning a different, more restricted value to the parent" — applied
> to sums instead of edge counts, with one new wrinkle: clamp negative
> branch contributions to zero, since a path is always free to simply not
> extend into an unprofitable subtree. Recognizing that this Hard-rated
> problem is really "Diameter, but with values and a clamp" is the kind
> of pattern transfer that makes the Trees phase increasingly fast to
> work through once the foundational templates click.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/binary-tree-maximum-path-sum/)

---

{% include kotlin-dsa-nav.html %}
