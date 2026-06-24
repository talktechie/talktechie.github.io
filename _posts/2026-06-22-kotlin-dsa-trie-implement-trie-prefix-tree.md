---
title: "Trie: Implement Trie (Prefix Tree) — Kotlin Solution"
date: 2026-06-22
categories: [Kotlin DSA, Trie]
tags: [implement-trie-prefix-tree, trie, design, hashmap, medium, leetcode, leetcode-208]
leetcode_number: 208
leetcode_url: https://leetcode.com/problems/implement-trie-prefix-tree/
difficulty: Medium
permalink: /posts/kotlin-dsa-trie-implement-trie-prefix-tree/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [208 — Implement Trie (Prefix Tree)](https://leetcode.com/problems/implement-trie-prefix-tree/) |
| **Difficulty** | Medium |
| **Topic** | Trie, Design, HashMap |

> A trie (pronounced "try") is a tree data structure used to efficiently
> store and retrieve keys in a dataset of strings. Implement a `Trie`
> class with:
> - `insert(word)` — inserts the string `word` into the trie.
> - `search(word)` — returns `true` if `word` was previously inserted
>   (exact match).
> - `startsWith(prefix)` — returns `true` if any previously inserted word
>   has `prefix` as a prefix.

**Example:**
```
Input:
["Trie","insert","search","search","startsWith","insert","search"]
[[],["apple"],["apple"],["app"],["app"],["app"],["app"]]

Output:
[null,null,true,false,true,null,true]

Explanation:
Trie trie = new Trie();
trie.insert("apple");
trie.search("apple");    // true  — exact match
trie.search("app");      // false — "app" was never inserted, only "apple"
trie.startsWith("app");  // true  — "apple" starts with "app"
trie.insert("app");
trie.search("app");      // true  — now "app" itself was inserted
```

**Constraints:**
- `1 <= word.length, prefix.length <= 2000`
- `word` and `prefix` consist only of lowercase English letters
- At most `3 * 10⁴` total calls to `insert`, `search`, and `startsWith`

---

## Approach

This is the foundational data structure for the entire Trie phase. A trie
is essentially a tree where **each edge represents one character**, and
words sharing a common prefix share the same path from the root.

**Key insight:** Each node holds a map of `character → child node`, plus a
boolean flag marking "a complete word ends here." Both `insert` and the
traversal-prefix of `search`/`startsWith` follow the exact same walk:
start at the root, and for each character, either follow an existing
child or create a new one (on insert) — character lookups never need to
scan sibling characters, since each is a distinct map key.

**Walk through `insert("apple")`, then `search("app")`, `startsWith("app")`:**
```
insert("apple"):
  root -a-> node1 -p-> node2 -p-> node3 -l-> node4 -e-> node5(isWord=true)

search("app"):
  root -a-> node1 -p-> node2 -p-> node3
  reached node3, but node3.isWord == false → return false ✓

startsWith("app"):
  same walk, reaches node3 — we don't care about isWord here,
  we only care that the path exists → return true ✓
```

---

## Kotlin Solution

### Approach 1 — HashMap of children per node (optimal, general-purpose)

```kotlin
class Trie {
    private class TrieNode {
        val children = HashMap<Char, TrieNode>()
        var isWord = false
    }

    private val root = TrieNode()

    fun insert(word: String) {
        var node = root
        for (c in word) {
            node = node.children.getOrPut(c) { TrieNode() }
        }
        node.isWord = true
    }

    fun search(word: String): Boolean {
        val node = traverse(word) ?: return false
        return node.isWord
    }

    fun startsWith(prefix: String): Boolean {
        return traverse(prefix) != null
    }

    private fun traverse(word: String): TrieNode? {
        var node = root
        for (c in word) {
            node = node.children[c] ?: return null
        }
        return node
    }
}
```

### Approach 2 — Fixed-size Array(26) per node (faster, lowercase-only)

```kotlin
class Trie {
    private class TrieNode {
        val children = arrayOfNulls<TrieNode>(26)
        var isWord = false
    }

    private val root = TrieNode()

    fun insert(word: String) {
        var node = root
        for (c in word) {
            val idx = c - 'a'
            if (node.children[idx] == null) node.children[idx] = TrieNode()
            node = node.children[idx]!!
        }
        node.isWord = true
    }

    fun search(word: String): Boolean {
        val node = traverse(word) ?: return false
        return node.isWord
    }

    fun startsWith(prefix: String): Boolean {
        return traverse(prefix) != null
    }

    private fun traverse(word: String): TrieNode? {
        var node = root
        for (c in word) {
            node = node.children[c - 'a'] ?: return null
        }
        return node
    }
}
```

Avoids HashMap overhead entirely — direct array indexing via `c - 'a'` is
faster in practice, at the cost of being restricted to lowercase English
letters specifically (which matches this problem's constraints exactly).

---

## Why `traverse` Is Shared Between `search` and `startsWith`

The two operations differ by exactly one thing: `search` additionally
checks `isWord` at the end, while `startsWith` only cares that the path
exists at all. Factoring out the common walk avoids duplicating logic:

```kotlin
private fun traverse(word: String): TrieNode? {
    var node = root
    for (c in word) {
        node = node.children[c] ?: return null   // path breaks → not found
    }
    return node   // path exists — caller decides what to do with it
}

fun search(word: String): Boolean = traverse(word)?.isWord ?: false
fun startsWith(prefix: String): Boolean = traverse(prefix) != null
```

**Why `getOrPut` is the right tool for `insert`:**
```kotlin
node = node.children.getOrPut(c) { TrieNode() }
// Creates a new child only if one doesn't already exist for this character,
// otherwise reuses the existing one — exactly the "share common prefixes" behavior
```

**Why `isWord` is necessary, not just "reaching a leaf":** a word like
`"app"` could be a complete word **and** a prefix of `"apple"` at the same
time — the node for `'p'` (second one) needs to be markable as "a word
ends here" even though it also has children leading deeper.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| HashMap children (Approach 1) | General-purpose, supports any character set (Unicode, digits, etc.) |
| Array(26) children (Approach 2) | Known lowercase-only constraint, want maximum speed |

---

## Complexity

| | insert | search | startsWith |
|---|---|---|---|
| **Time** | O(L) | O(L) | O(L) |
| **Space** | O(L) per word inserted (shared prefixes reduce actual usage) | | |

Where `L` is the length of the word/prefix.

---

## Key Takeaway

> A trie trades space for prefix-query speed: every operation is O(L) —
> proportional only to the word's length, completely independent of how
> many other words are stored. Shared prefixes between words naturally
> collapse into shared tree paths, which is exactly what makes
> `startsWith` so efficient — it's just a path-existence check, no
> scanning of the full word list required. This `TrieNode` structure
> (children map + `isWord` flag) is the building block for every other
> problem in this phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/implement-trie-prefix-tree/)

---

{% include kotlin-dsa-nav.html %}
