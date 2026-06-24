---
title: "Trie: Design Add And Search Words Data Structure — Kotlin Solution"
date: 2026-06-22
categories: [Kotlin DSA, Trie]
tags: [design-add-search-words-data-structure, trie, dfs, backtracking, medium, leetcode, leetcode-211]
leetcode_number: 211
leetcode_url: https://leetcode.com/problems/design-add-and-search-words-data-structure/
difficulty: Medium
permalink: /posts/kotlin-dsa-trie-design-add-and-search-words-data-structure/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [211 — Design Add And Search Words Data Structure](https://leetcode.com/problems/design-add-and-search-words-data-structure/) |
| **Difficulty** | Medium |
| **Topic** | Trie, DFS, Backtracking |

> Design a data structure that supports adding new words and searching
> for a string, where the search string may contain the **wildcard**
> character `'.'` which matches any single letter.
>
> - `addWord(word)` — adds `word` to the data structure.
> - `search(word)` — returns `true` if there is a previously added string
>   that matches `word`, where `.` can match any letter.

**Example:**
```
Input:
["WordDictionary","addWord","addWord","addWord","search","search","search","search"]
[[],["bad"],["dad"],["mad"],["pad"],["bad"],[".ad"],["b.."]]

Output:
[null,null,null,null,false,true,true,true]

Explanation:
addWord("bad"); addWord("dad"); addWord("mad");
search("pad");  // false — no exact match, no wildcard used
search("bad");  // true  — exact match
search(".ad");  // true  — '.' matches 'b' (or 'd' or 'm')
search("b..");  // true  — '.' matches 'a', '.' matches 'd'
```

**Constraints:**
- `1 <= word.length <= 25`
- `word` in `addWord` consists of lowercase English letters
- `word` in `search` consists of `'.'` or lowercase English letters
- At most 2 dots in any search word
- At most `10⁴` calls to `addWord` and `search`

---

## Approach

`addWord` is identical to a standard Trie `insert` (LC 208). The new
challenge is `search` — when a `.` is encountered, it could match **any**
of the current node's children, so a single deterministic path walk isn't
enough anymore. We need **backtracking**: try every possible branch at a
wildcard position, and succeed if **any** branch leads to a full match.

**Key insight:** Use DFS with backtracking. At each character position:
- If it's a regular letter, follow that one specific child (same as
  normal trie traversal) — if it doesn't exist, this path fails.
- If it's `.`, **try every child** of the current node — succeed if any
  one of them leads to a successful match for the rest of the word.

**Walk through `search(".ad")` after adding `"bad"`, `"dad"`, `"mad"`:**
```
dfs(node=root, index=0, char='.'):
  '.' matches any child → try ALL children of root: 'b', 'd', 'm'
    try 'b' → dfs(node=root.children['b'], index=1, char='a')
       'a' is a literal → follow root.children['b'].children['a']
       dfs(..., index=2, char='d') → follow to the 'd' node
       dfs(..., index=3) → index reached word.length, check isWord → true!
    → "bad" path succeeds → return true immediately, no need to try 'd' or 'm'

Result: true ✓
```

---

## Kotlin Solution

### Approach 1 — Trie + DFS backtracking for wildcards (optimal)

```kotlin
class WordDictionary {
    private class TrieNode {
        val children = HashMap<Char, TrieNode>()
        var isWord = false
    }

    private val root = TrieNode()

    fun addWord(word: String) {
        var node = root
        for (c in word) {
            node = node.children.getOrPut(c) { TrieNode() }
        }
        node.isWord = true
    }

    fun search(word: String): Boolean {
        fun dfs(node: TrieNode, index: Int): Boolean {
            if (index == word.length) return node.isWord

            val c = word[index]

            if (c == '.') {
                // Wildcard — try every child, succeed if any path works
                for (child in node.children.values) {
                    if (dfs(child, index + 1)) return true
                }
                return false
            } else {
                // Regular letter — follow the one specific matching child
                val next = node.children[c] ?: return false
                return dfs(next, index + 1)
            }
        }

        return dfs(root, 0)
    }
}
```

### Approach 2 — Same idea, iterative-style with explicit early exits

```kotlin
class WordDictionary {
    private class TrieNode {
        val children = arrayOfNulls<TrieNode>(26)
        var isWord = false
    }

    private val root = TrieNode()

    fun addWord(word: String) {
        var node = root
        for (c in word) {
            val idx = c - 'a'
            if (node.children[idx] == null) node.children[idx] = TrieNode()
            node = node.children[idx]!!
        }
        node.isWord = true
    }

    fun search(word: String): Boolean = search(word, 0, root)

    private fun search(word: String, index: Int, node: TrieNode): Boolean {
        if (index == word.length) return node.isWord

        val c = word[index]

        if (c != '.') {
            val child = node.children[c - 'a'] ?: return false
            return search(word, index + 1, child)
        }

        for (child in node.children) {
            if (child != null && search(word, index + 1, child)) return true
        }
        return false
    }
}
```

Uses the array-based trie node from LC 208's Approach 2, with the same
backtracking search logic — slightly faster child iteration since it's a
fixed array scan rather than a HashMap values iteration.

---

## Why This Is Backtracking, Not Plain Traversal

A normal trie search follows exactly **one** path — character by
character, deterministically. A `.` breaks that determinism: at that
position, multiple paths might be valid, and we don't know which one (if
any) leads to a full match without trying them.

```kotlin
if (c == '.') {
    for (child in node.children.values) {
        if (dfs(child, index + 1)) return true   // try this branch
        // if it returns false, the loop naturally moves to the NEXT
        // child — this implicit "undo and try something else" is backtracking
    }
    return false   // no child led to a match
}
```

**Why we return `true` immediately on the first successful branch:** we
only need to know *if* a match exists, not enumerate every possible
match — so short-circuiting on the first success avoids exploring
unnecessary branches.

**Why the base case checks `node.isWord`, not just `index ==
word.length`:** reaching the end of the search string doesn't
automatically mean success — the current node must also have been marked
as a complete word during some `addWord` call, exactly like `search` in
LC 208.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| HashMap children + DFS (Approach 1) | General-purpose, supports any character set |
| Array(26) children + DFS (Approach 2) | Known lowercase-only constraint, marginally faster iteration |

---

## Complexity

| | addWord | search (no wildcards) | search (with wildcards) |
|---|---|---|---|
| **Time** | O(L) | O(L) | O(26^d × L) worst case, where d = number of dots |
| **Space** | O(L) per word | O(L) recursion stack | O(L) recursion stack |

The problem's constraint of "at most 2 dots" keeps the wildcard
worst-case manageable in practice (26² branching at most, not 26^L).

---

## Key Takeaway

> A trie search becomes a backtracking search the moment any part of the
> query is ambiguous — here, a wildcard that could match multiple
> children. The pattern is: deterministic character → follow one path;
> ambiguous character → try every available path, short-circuiting on
> the first success. This "trie + backtracking for ambiguous queries"
> combination is the direct precursor to Word Search II, which adds grid
> traversal on top of this same idea.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/design-add-and-search-words-data-structure/)

---

{% include kotlin-dsa-nav.html %}
