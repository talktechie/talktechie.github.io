---
title: "Graphs: Word Ladder — Kotlin Solution"
date: 2026-06-25
categories: [Kotlin DSA, Graphs]
tags: [word-ladder, graph, bfs, hashmap, hard, leetcode, leetcode-127]
leetcode_number: 127
leetcode_url: https://leetcode.com/problems/word-ladder/
difficulty: Hard
permalink: /posts/kotlin-dsa-graphs-word-ladder/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [127 — Word Ladder](https://leetcode.com/problems/word-ladder/) |
| **Difficulty** | Hard |
| **Topic** | Graph, BFS, HashMap |

> Given `beginWord`, `endWord`, and a dictionary `wordList`, find the
> **length** of the shortest transformation sequence from `beginWord` to
> `endWord`, where each step changes exactly **one letter**, and every
> intermediate word must exist in `wordList`. Return `0` if no such
> sequence exists.

**Example:**
```
Input:  beginWord = "hit", endWord = "cog",
        wordList = ["hot","dot","dog","lot","log","cog"]
Output: 5
Explanation: "hit" -> "hot" -> "dot" -> "dog" -> "cog" — 5 words, 4 steps

Input:  beginWord = "hit", endWord = "cog",
        wordList = ["hot","dot","dog","lot","log"]
Output: 0
Explanation: "cog" is not in wordList — unreachable
```

**Constraints:**
- `1 <= beginWord.length <= 10`
- `endWord.length == beginWord.length`
- `1 <= wordList.length <= 5000`
- All words consist of lowercase English letters, same length as `beginWord`

---

## Approach

This closes out the Graphs phase by combining everything: an **implicit
graph** (words are nodes, an edge exists between two words differing by
exactly one letter), and **BFS** to find the shortest path — since BFS
naturally finds shortest paths in unweighted graphs, exactly as it found
shortest distances in Walls And Gates.

**Key insight — don't build the graph explicitly; generate neighbors on
the fly using wildcard patterns:** explicitly checking every pair of
words for a one-letter difference would be O(wordList² × wordLength) —
too slow. Instead, for each word, generate all possible **one-letter
variations** (changing each position to every other letter) and check if
that variation exists in the word set — an O(26 × wordLength) operation
per word, dramatically faster.

**Walk through `beginWord = "hit"`, partial BFS trace:**
```
wordSet = {"hot","dot","dog","lot","log","cog"}
queue: [("hit", 1)]  (word, steps-so-far)

Dequeue ("hit", 1): generate all one-letter variations of "hit":
  change position 0: "ait","bit",...,"zit" → none in wordSet except checking each
  change position 1: "hat","hbt",...,"hot"✓,...,"hzt" → "hot" found in wordSet!
  change position 2: "hia","hib",...,"hiz" → none found

  "hot" is in wordSet and unvisited → enqueue ("hot", 2), mark visited

Dequeue ("hot", 2): generate variations:
  position 1 changes: "dot"✓ found, "lot"✓ found
  → enqueue ("dot", 3), enqueue ("lot", 3)

... continues ...

Eventually reaches "cog" at step 5 → return 5 ✓
```

---

## Kotlin Solution

### Approach 1 — BFS with on-the-fly neighbor generation via wildcard patterns (optimal)

```kotlin
fun ladderLength(beginWord: String, endWord: String, wordList: List<String>): Int {
    val wordSet = wordList.toHashSet()
    if (endWord !in wordSet) return 0

    val queue: ArrayDeque<Pair<String, Int>> = ArrayDeque()
    queue.addLast(beginWord to 1)
    wordSet.remove(beginWord)   // prevent revisiting

    while (queue.isNotEmpty()) {
        val (word, steps) = queue.removeFirst()

        if (word == endWord) return steps

        val chars = word.toCharArray()
        for (i in chars.indices) {
            val original = chars[i]
            for (c in 'a'..'z') {
                if (c == original) continue
                chars[i] = c
                val candidate = String(chars)

                if (candidate in wordSet) {
                    wordSet.remove(candidate)   // mark visited by removing from the set
                    queue.addLast(candidate to steps + 1)
                }
            }
            chars[i] = original   // restore before trying the next position
        }
    }

    return 0
}
```

### Approach 2 — Precompute a pattern map for faster neighbor lookup (better for large wordLists)

```kotlin
fun ladderLength(beginWord: String, endWord: String, wordList: List<String>): Int {
    val allWords = (wordList + beginWord).toHashSet()
    if (endWord !in allWords) return 0

    val patternMap = HashMap<String, MutableList<String>>()
    for (word in allWords) {
        for (i in word.indices) {
            val pattern = word.substring(0, i) + "*" + word.substring(i + 1)
            patternMap.getOrPut(pattern) { mutableListOf() }.add(word)
        }
    }

    val queue: ArrayDeque<Pair<String, Int>> = ArrayDeque()
    queue.addLast(beginWord to 1)
    val visited = HashSet<String>()
    visited.add(beginWord)

    while (queue.isNotEmpty()) {
        val (word, steps) = queue.removeFirst()
        if (word == endWord) return steps

        for (i in word.indices) {
            val pattern = word.substring(0, i) + "*" + word.substring(i + 1)
            for (neighbor in patternMap[pattern] ?: emptyList()) {
                if (neighbor !in visited) {
                    visited.add(neighbor)
                    queue.addLast(neighbor to steps + 1)
                }
            }
        }
    }

    return 0
}
```

Precomputes a map from "wildcard pattern" (e.g., `"h*t"`) to every word
matching that pattern, turning neighbor lookup into a direct map access
instead of generating and checking 26 candidates per character position
— faster when the word list is large relative to the alphabet size.

---

## Why Generating Variations Beats Checking All Pairs

Checking every pair of words for a one-letter difference costs
O(wordList² × wordLength) — for 5000 words, that's potentially 25 million
comparisons. Generating variations instead costs, per word,
O(wordLength × 26) — a small constant — to check candidate strings
against a HashSet:

```kotlin
for (i in chars.indices) {
    for (c in 'a'..'z') {
        if (c == original) continue
        chars[i] = c
        val candidate = String(chars)
        if (candidate in wordSet) { ... }
    }
}
```

This trades "compare against every other word" for "generate every
plausible neighbor and check membership" — a classic technique whenever
the space of *possible* neighbors is small and enumerable, even if the
actual dataset is large.

**Why removing words from `wordSet` (rather than a separate `visited`
set) works as visited-tracking:** once a word has been dequeued and
explored, there's no reason to ever reach it again via a different path
— BFS guarantees the first time we reach any word is via the shortest
path to it. Removing it from the set we're checking against prevents
ever re-enqueueing it, serving the same purpose as marking `visited[word]
= true` would, with one less data structure to maintain.

**Why the pattern-map approach (Approach 2) helps for large inputs:**
generating 26 candidates per character position means redundant work if
many words share long common patterns — precomputing the pattern→words
mapping once upfront amortizes that cost across the entire BFS, paying
off when `wordList` is large enough that repeated regeneration becomes
the bottleneck.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| On-the-fly generation (Approach 1) | Simpler to write, sufficient for this problem's constraints |
| Pattern map precomputation (Approach 2) | Larger word lists, want to avoid repeated candidate generation |

---

## Complexity

| | Approach 1 | Approach 2 |
|---|---|---|
| **Time** | O(wordList × wordLength × 26) | O(wordList × wordLength) for setup + BFS |
| **Space** | O(wordList × wordLength) | O(wordList × wordLength) — pattern map |

---

## Key Takeaway

> Word Ladder closes the Graphs phase by treating an entirely
> non-obvious structure — a dictionary of words — as a graph, where
> "edge" means "differs by exactly one letter." BFS's shortest-path
> guarantee (the same property used in Walls And Gates and Rotting
> Oranges) directly gives us the minimum transformation length. The real
> engineering challenge isn't the BFS itself — it's recognizing that
> explicitly building the graph would be wasteful, and that **generating
> plausible neighbors on demand** (via wildcard substitution) is the key
> technique that makes the whole approach tractable at scale.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/word-ladder/)

---

{% include kotlin-dsa-nav.html %}
