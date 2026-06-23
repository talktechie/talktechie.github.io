---
title: "Trees: Subtree of Another Tree — Kotlin Solution"
date: 2026-06-21
categories: [Kotlin DSA, Trees]
tags: [subtree-of-another-tree, tree, dfs, recursion, easy, leetcode, leetcode-572]
leetcode_number: 572
leetcode_url: https://leetcode.com/problems/subtree-of-another-tree/
difficulty: Easy
permalink: /posts/kotlin-dsa-trees-subtree-of-another-tree/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [572 — Subtree of Another Tree](https://leetcode.com/problems/subtree-of-another-tree/) |
| **Difficulty** | Easy |
| **Topic** | Tree, DFS, Recursion |

> Given the roots of two binary trees `root` and `subRoot`, return `true`
> if there is a subtree of `root` with the same structure and node values
> as `subRoot`.

**Example:**
```
Input:      3                  subRoot:   4
          /   \                          / \
         4     5                        1   2
        / \
       1   2
Output: true

Input:      3                  subRoot:   4
          /   \                          / \
         4     5                        1   2
        / \                                /
       1   2                              0
Output: false
Explanation: the 4-1-2 subtree in root has no extra "0" node, so it
doesn't exactly match subRoot
```

**Constraints:**
- Nodes in `root`: `[1, 2000]`
- Nodes in `subRoot`: `[1, 1000]`
- `-10⁴ <= Node.val <= 10⁴`

---

## Approach

This builds **directly** on Same Tree (LC 100) — reuse it as a helper.

**Key insight:** A tree contains `subRoot` as a subtree if, for **some**
node in `root`, the subtree starting at that node is identical (via
`isSameTree`) to `subRoot`. So: traverse every node of `root`, and at
each one, check if it matches `subRoot` exactly using the Same Tree logic.

**Walk through `root = [3,4,5,1,2], subRoot = [4,1,2]`:**
```
Check node 3: isSameTree(3-subtree, subRoot)?
  3 != 4 → false, not a match here
  recurse: check left subtree (rooted at 4) OR right subtree (rooted at 5)

Check node 4 (root's left child): isSameTree(4-subtree, subRoot)?
  4==4, recurse left: isSameTree(1, 1) → match
        recurse right: isSameTree(2, 2) → match
  → 4's subtree fully matches subRoot → true!

Answer: true ✓
```

---

## Kotlin Solution

### Approach 1 — Reuse Same Tree as a helper, check every node (optimal)

```kotlin
fun isSubtree(root: TreeNode?, subRoot: TreeNode?): Boolean {
    if (root == null) return subRoot == null  // empty root can only match an empty subRoot
    if (isSameTree(root, subRoot)) return true

    return isSubtree(root.left, subRoot) || isSubtree(root.right, subRoot)
}

private fun isSameTree(p: TreeNode?, q: TreeNode?): Boolean {
    if (p == null && q == null) return true
    if (p == null || q == null) return false
    if (p.`val` != q.`val`) return false

    return isSameTree(p.left, q.left) && isSameTree(p.right, q.right)
}
```

### Approach 2 — Serialize both trees to strings, check substring containment

```kotlin
fun isSubtree(root: TreeNode?, subRoot: TreeNode?): Boolean {
    fun serialize(node: TreeNode?): String {
        if (node == null) return ",#"
        // Markers (",") before each value prevent false matches like
        // "2" matching inside "12" or "23"
        return ",${node.`val`}${serialize(node.left)}${serialize(node.right)}"
    }

    val rootStr = serialize(root)
    val subRootStr = serialize(subRoot)

    return rootStr.contains(subRootStr)
}
```

A clever alternative — preorder-serialize both trees (with `null` markers
to preserve structure), then it's just a substring search. Using a
library string-search function makes this O(m+n) on average, though
worst-case substring search can be O(m×n) without a more advanced
algorithm like KMP.

---

## Why Checking Every Node Is Necessary (Not Just the Root)

```kotlin
if (isSameTree(root, subRoot)) return true
return isSubtree(root.left, subRoot) || isSubtree(root.right, subRoot)
```

`subRoot` might match a subtree buried deep inside `root` — not just at
the top level. So we recursively try **every** node of `root` as a
potential matching point, using `isSameTree` as the "does this exact spot
match?" check, and `isSubtree` recursively as the "keep searching
elsewhere" mechanism.

**Why `root == null` returns `subRoot == null`, not just `false`:**
```kotlin
if (root == null) return subRoot == null
```
If both are empty, that's a vacuous match (an empty tree trivially
"contains" an empty subtree). If `root` is empty but `subRoot` isn't,
there's nothing left to search — correctly returns `false`.

**Why the serialization approach needs `null` markers (`"#"`):**
```kotlin
if (node == null) return ",#"
// Without markers, trees [1,2] and [1,null,2] could serialize to
// confusingly similar strings, causing false positive substring matches
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Reuse Same Tree (Approach 1) | Always — clean, reuses a function you already understand |
| Serialize + substring search (Approach 2) | Interesting alternative, demonstrates string-matching creativity |

---

## Complexity

| | Approach 1 | Approach 2 |
|---|---|---|
| **Time** | O(m × n) worst case (m, n = node counts) | O(m + n) average with efficient substring search |
| **Space** | O(h) recursion | O(m + n) for the serialized strings |

---

## Key Takeaway

> "Is X a subtree of Y" decomposes into "check if X matches Y starting
> at this node" (reusing Same Tree) combined with "try every node of Y as
> a starting point" (a tree traversal). This composability — building a
> new tree algorithm directly on top of a previously solved one — is one
> of the most valuable skills to develop in this phase: most tree
> problems are small variations or combinations of a handful of core
> recursive templates.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/subtree-of-another-tree/)

---

{% include kotlin-dsa-nav.html %}
