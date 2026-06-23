---
title: "Trees: Same Tree — Kotlin Solution"
date: 2026-06-21
categories: [Kotlin DSA, Trees]
tags: [same-tree, tree, dfs, recursion, easy, leetcode, leetcode-100]
leetcode_number: 100
leetcode_url: https://leetcode.com/problems/same-tree/
difficulty: Easy
permalink: /posts/kotlin-dsa-trees-same-tree/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [100 — Same Tree](https://leetcode.com/problems/same-tree/) |
| **Difficulty** | Easy |
| **Topic** | Tree, DFS, Recursion |

> Given the roots of two binary trees `p` and `q`, return `true` if they
> are **structurally identical** and the nodes have the same values.

**Example:**
```
Input:  p = [1,2,3], q = [1,2,3]
Output: true

Input:  p = [1,2], q = [1,null,2]
Output: false
Explanation: same value at root, but different structure
              (2 is a left child in p, a right child in q)

Input:  p = [1,2,1], q = [1,1,2]
Output: false
Explanation: same shape, but different values at the leaves
```

**Constraints:**
- The number of nodes in both trees is in the range `[0, 100]`
- `-10⁴ <= Node.val <= 10⁴`

---

## Approach

A foundational comparison problem — the recursive structure mirrors
exactly how you'd describe "are these the same" in plain English: same
value at this node, AND the left subtrees match, AND the right subtrees
match.

**Key insight:** Compare both trees node by node, simultaneously. At each
pair of nodes:
- If both are `null` → they match (both empty, trivially equal)
- If exactly one is `null` → mismatch, different structure
- If both exist but values differ → mismatch
- Otherwise → recurse into left children and right children, both must match

**Walk through `p = [1,2,1], q = [1,1,2]`:**
```
isSameTree(p=1, q=1): values match (1==1)
  recurse left:  isSameTree(p.left=2, q.left=1) → values differ (2!=1) → false

Since the left comparison already returned false, the overall result is
false — short-circuits without even checking the right subtrees.

Answer: false ✓
```

---

## Kotlin Solution

### Approach 1 — Recursive DFS (optimal, the standard answer)

```kotlin
fun isSameTree(p: TreeNode?, q: TreeNode?): Boolean {
    if (p == null && q == null) return true
    if (p == null || q == null) return false
    if (p.`val` != q.`val`) return false

    return isSameTree(p.left, q.left) && isSameTree(p.right, q.right)
}
```

### Approach 2 — Iterative BFS with paired queue

```kotlin
fun isSameTree(p: TreeNode?, q: TreeNode?): Boolean {
    val queue: ArrayDeque<Pair<TreeNode?, TreeNode?>> = ArrayDeque()
    queue.addLast(p to q)

    while (queue.isNotEmpty()) {
        val (nodeP, nodeQ) = queue.removeFirst()

        if (nodeP == null && nodeQ == null) continue
        if (nodeP == null || nodeQ == null) return false
        if (nodeP.`val` != nodeQ.`val`) return false

        queue.addLast(nodeP.left to nodeQ.left)
        queue.addLast(nodeP.right to nodeQ.right)
    }

    return true
}
```

Compares both trees level by level using paired nodes in the queue —
same logic as the recursive version, expressed iteratively.

---

## Why the Three Checks Must Be in This Order

```kotlin
if (p == null && q == null) return true   // both empty — trivially identical
if (p == null || q == null) return false  // exactly one is empty — definitely different
if (p.`val` != q.`val`) return false       // both exist, but values differ
```

The second check is only reached if the first one fails — meaning at
least one of `p`/`q` is non-null. Combined with `||`, this correctly
catches "exactly one is null" without needing a more complex condition.
The third check is only safe to run once we know **both** `p` and `q`
are non-null — accessing `p.val` before that check could throw a null
pointer exception.

**The final line uses `&&`, not separate statements:**
```kotlin
return isSameTree(p.left, q.left) && isSameTree(p.right, q.right)
```
Kotlin's `&&` short-circuits — if the left subtrees don't match, the
right subtrees are never even checked, since the overall result is
already `false`. This is a small but free optimization that falls out
naturally from idiomatic Kotlin.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Recursive DFS (Approach 1) | Always — most concise, directly mirrors the problem's definition |
| Iterative BFS (Approach 2) | Want to avoid recursion, or asked for an alternative traversal style |

---

## Complexity

| | |
|---|---|
| **Time** | O(min(m, n)) — stops as soon as a mismatch is found; O(n) if trees are identical |
| **Space** | O(h) recursive / O(w) iterative |

---

## Key Takeaway

> Comparing two trees simultaneously is a "zip and compare" recursion —
> walk both trees in lockstep, checking node-level equality and
> delegating to the same check on both pairs of children. The three-line
> base-case sequence (`both null` → `one null` → `values differ`) is a
> reusable template for any tree-comparison problem, including Subtree of
> Another Tree, which builds directly on this exact function.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/same-tree/)

---

{% include kotlin-dsa-nav.html %}
