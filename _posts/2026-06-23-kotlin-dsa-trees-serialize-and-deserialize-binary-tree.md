---
title: "Trees: Serialize And Deserialize Binary Tree — Kotlin Solution"
date: 2026-06-21
categories: [Kotlin DSA, Trees]
tags: [serialize-deserialize-binary-tree, tree, dfs, bfs, design, hard, leetcode, leetcode-297]
leetcode_number: 297
leetcode_url: https://leetcode.com/problems/serialize-and-deserialize-binary-tree/
difficulty: Hard
permalink: /posts/kotlin-dsa-trees-serialize-and-deserialize-binary-tree/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [297 — Serialize And Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/) |
| **Difficulty** | Hard |
| **Topic** | Tree, DFS, BFS, Design |

> Design an algorithm to **serialize** a binary tree to a string, and
> **deserialize** that string back to the exact same tree structure.
> There's no constraint on how serialization must work — only that
> `deserialize(serialize(tree))` reproduces the original tree exactly.

**Example:**
```
Input:      1
          /   \
         2     3
              / \
             4   5
Output: [1,2,3,null,null,4,5]
(serialize then deserialize reproduces this exact structure)

Input:  []
Output: []
```

**Constraints:**
- Nodes: `[0, 10⁴]`
- `-1000 <= Node.val <= 1000`

---

## Approach

The closing problem of this phase, and it ties together two recurring
ideas: tree traversal (DFS, similar to LC 105's preorder reconstruction)
and the **explicit null-marker encoding** trick from Encode and Decode
Strings (LC 271) — except here we mark missing *children*, not delimit
strings.

**Key insight — preorder with explicit null markers:** A preorder
traversal (root, then left, then right) normally loses structural
information once flattened to a string — you can't tell where one
subtree ends and another begins. The fix: **explicitly record every
`null` child** as a placeholder token. With nulls marked, the string
unambiguously encodes the tree's exact shape, and deserializing becomes a
straightforward recursive "consume one token at a time" process.

**Walk through `[1,2,3,null,null,4,5]`:**
```
Serialize (preorder, with "#" for null):
  visit 1 → "1"
  visit 2 → "2", visit 2.left(null) → "#", visit 2.right(null) → "#"
  visit 3 → "3"
    visit 4 → "4", visit 4.left(null)→"#", visit 4.right(null)→"#"
    visit 5 → "5", visit 5.left(null)→"#", visit 5.right(null)→"#"

Result string: "1,2,#,#,3,4,#,#,5,#,#"

Deserialize (consume tokens left to right, same preorder order):
  read "1" → create node(1)
    read "2" → create node(2)
      read "#" → null (2's left)
      read "#" → null (2's right)
    read "3" → create node(3)
      read "4" → create node(4)
        read "#" → null, read "#" → null
      read "5" → create node(5)
        read "#" → null, read "#" → null

Reconstructed tree matches the original exactly ✓
```

---

## Kotlin Solution

### Approach 1 — Preorder DFS with explicit null markers (optimal)

```kotlin
class Codec {
    fun serialize(root: TreeNode?): String {
        val sb = StringBuilder()

        fun dfs(node: TreeNode?) {
            if (node == null) {
                sb.append("#,")
                return
            }
            sb.append(node.`val`).append(",")
            dfs(node.left)
            dfs(node.right)
        }

        dfs(root)
        return sb.toString()
    }

    fun deserialize(data: String): TreeNode? {
        val tokens = ArrayDeque(data.split(",").filter { it.isNotEmpty() })

        fun build(): TreeNode? {
            val token = tokens.removeFirst()
            if (token == "#") return null

            val node = TreeNode(token.toInt())
            node.left = build()
            node.right = build()
            return node
        }

        return build()
    }
}
```

### Approach 2 — BFS-based level order with null markers

```kotlin
class Codec {
    fun serialize(root: TreeNode?): String {
        if (root == null) return ""

        val sb = StringBuilder()
        val queue: ArrayDeque<TreeNode?> = ArrayDeque()
        queue.addLast(root)

        while (queue.isNotEmpty()) {
            val node = queue.removeFirst()
            if (node == null) {
                sb.append("#,")
            } else {
                sb.append(node.`val`).append(",")
                queue.addLast(node.left)
                queue.addLast(node.right)
            }
        }

        return sb.toString()
    }

    fun deserialize(data: String): TreeNode? {
        if (data.isEmpty()) return null

        val tokens = data.split(",").filter { it.isNotEmpty() }
        val root = TreeNode(tokens[0].toInt())
        val queue: ArrayDeque<TreeNode> = ArrayDeque()
        queue.addLast(root)

        var i = 1
        while (queue.isNotEmpty()) {
            val node = queue.removeFirst()

            if (tokens[i] != "#") {
                node.left = TreeNode(tokens[i].toInt())
                queue.addLast(node.left!!)
            }
            i++

            if (tokens[i] != "#") {
                node.right = TreeNode(tokens[i].toInt())
                queue.addLast(node.right!!)
            }
            i++
        }

        return root
    }
}
```

Same null-marker idea, applied level by level instead of preorder. Both
approaches are O(n) — pick whichever traversal style you find more
intuitive.

---

## Why Marking Nulls Explicitly Is Necessary

Without null markers, a preorder sequence like `"1,2,3"` is structurally
ambiguous — is `2` the left child of `1` with `3` as `2`'s child, or is
`2` the left child and `3` the right child of `1`? Multiple trees can
share the same preorder sequence if you don't also record where the
`null`s are:

```kotlin
if (node == null) {
    sb.append("#,")   // explicit marker — preserves exact structure
    return
}
```

This is the same insight as Encode and Decode Strings (LC 271) — "no
delimiter is inherently safe without extra structural information" — just
applied to tree shape instead of string boundaries.

**Why deserialize must consume tokens in the exact same order they were
written:**
```kotlin
val node = TreeNode(token.toInt())
node.left = build()    // recurse immediately — consumes the NEXT tokens
node.right = build()   // continues exactly where left's recursion left off
```
This mirrors the shared-pointer trick from Construct Binary Tree From
Preorder And Inorder Traversal — a single shared, continuously-advancing
position in the token stream, consumed in the same order it was produced.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Preorder DFS (Approach 1) | Simpler recursive logic, most commonly seen |
| BFS level order (Approach 2) | Prefer iterative style, or want to avoid recursion depth on deep trees |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) for both serialize and deserialize |
| **Space** | O(n) — the encoded string, plus O(h) (DFS) or O(w) (BFS) for traversal |

---

## Key Takeaway

> Closing out the Trees phase, this problem combines two ideas you've
> now seen multiple times: a traversal order (preorder or level order)
> determines how to walk the tree, and **explicit null markers** are
> required to preserve structure when flattening a tree to a flat
> sequence — the same lesson as Encode and Decode Strings, applied here
> to tree shape rather than string boundaries. Deserializing is simply
> running that same traversal in reverse, consuming tokens in the exact
> order they were produced.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/)

---

{% include kotlin-dsa-nav.html %}
