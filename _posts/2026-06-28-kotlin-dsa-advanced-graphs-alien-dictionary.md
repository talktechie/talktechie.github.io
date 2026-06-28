---
title: "Advanced Graphs: Alien Dictionary — Kotlin Solution"
date: 2026-06-28
categories: [Kotlin DSA, Advanced Graphs]
tags: [alien-dictionary, graph, topological-sort, dfs, bfs, hard, leetcode]
leetcode_number: 269
leetcode_url: https://leetcode.com/problems/alien-dictionary/
difficulty: Hard
permalink: /posts/kotlin-dsa-advanced-graphs-alien-dictionary/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [269 — Alien Dictionary](https://leetcode.com/problems/alien-dictionary/) (Premium — also on NeetCode) |
| **Difficulty** | Hard |
| **Topic** | Graph, Topological Sort, DFS, BFS |

> Given a list of `words` sorted lexicographically according to an
> **unknown alien alphabet's ordering**, derive that ordering. Return any
> valid character ordering, or `""` if the input is invalid (contradicts
> itself).

**Example:**
```
Input:  words = ["wrt","wrf","er","ett","rftt"]
Output: "wertf"
Explanation: comparing adjacent words letter-by-letter where they first
             differ reveals: w<e (from wrt vs er... wait let's recheck:
             actually w comes before e because "wrt" < "wrf" gives t<f
             at position 2; "wrf" < "er" gives w<e at position 0;
             "er" < "ett" gives r<t at position 1; "ett" < "rftt" gives
             e<r at position 0) — combining all constraints: w<e, e<r,
             r<t, t<f → valid order "wertf"

Input:  words = ["z","x","z"]
Output: ""
Explanation: "z" < "x" implies z<x, but "x" < "z" implies x<z — contradiction
```

**Constraints:**
- `1 <= words.length <= 100`
- `1 <= words[i].length <= 100`
- `words[i]` consists of lowercase English letters

---

## Approach

This closes a loop back to Course Schedule's topological sort technique
from the Graphs phase — except here, we must first **derive** the graph
itself from indirect clues (the relative ordering of adjacent words),
before running topological sort on it.

**Key insight — extract ordering constraints from adjacent word pairs
only:** for each pair of **consecutive** words in the sorted list,
compare them character by character until the **first** differing
position. That difference reveals exactly one ordering constraint
(`word1[i]` comes before `word2[i]`). Comparing non-adjacent words
provides no new information beyond what's already implied transitively
by the adjacent comparisons. Build a directed graph from these
constraints, then topologically sort it — if a cycle exists, the input
is contradictory and invalid.

**Walk through `words = ["wrt","wrf","er","ett","rftt"]`:**
```
Compare "wrt" vs "wrf": first difference at index 2: 't' vs 'f' → edge t->f
Compare "wrf" vs "er": first difference at index 0: 'w' vs 'e' → edge w->e
Compare "er" vs "ett": first difference at index 1: 'r' vs 't' → edge r->t
Compare "ett" vs "rftt": first difference at index 0: 'e' vs 'r' → edge e->r

Graph edges: t->f, w->e, r->t, e->r
Topological order must respect: w before e, e before r, r before t, t before f
→ "w","e","r","t","f" → "wertf" ✓
```

---

## Kotlin Solution

### Approach 1 — Build graph from adjacent word comparisons, Kahn's algorithm (BFS topological sort)

```kotlin
fun alienOrder(words: Array<String>): String {
    val graph = HashMap<Char, MutableSet<Char>>()
    val inDegree = HashMap<Char, Int>()

    for (word in words) {
        for (c in word) {
            graph.putIfAbsent(c, mutableSetOf())
            inDegree.putIfAbsent(c, 0)
        }
    }

    for (i in 0 until words.size - 1) {
        val w1 = words[i]
        val w2 = words[i + 1]
        val minLen = minOf(w1.length, w2.length)
        var foundDifference = false

        for (j in 0 until minLen) {
            if (w1[j] != w2[j]) {
                if (w2[j] !in graph[w1[j]]!!) {
                    graph[w1[j]]!!.add(w2[j])
                    inDegree[w2[j]] = inDegree[w2[j]]!! + 1
                }
                foundDifference = true
                break
            }
        }

        // Edge case: "abc" before "ab" is invalid (a prefix that's
        // longer can't come after its own shorter prefix alphabetically)
        if (!foundDifference && w1.length > w2.length) return ""
    }

    val queue: ArrayDeque<Char> = ArrayDeque()
    for ((c, degree) in inDegree) {
        if (degree == 0) queue.addLast(c)
    }

    val result = StringBuilder()
    while (queue.isNotEmpty()) {
        val c = queue.removeFirst()
        result.append(c)

        for (next in graph[c]!!) {
            inDegree[next] = inDegree[next]!! - 1
            if (inDegree[next] == 0) queue.addLast(next)
        }
    }

    return if (result.length == graph.size) result.toString() else ""   // shorter than expected = cycle detected
}
```

### Approach 2 — DFS-based topological sort with cycle detection

```kotlin
fun alienOrder(words: Array<String>): String {
    val graph = HashMap<Char, MutableSet<Char>>()
    for (word in words) for (c in word) graph.putIfAbsent(c, mutableSetOf())

    for (i in 0 until words.size - 1) {
        val w1 = words[i]
        val w2 = words[i + 1]
        val minLen = minOf(w1.length, w2.length)
        var foundDifference = false

        for (j in 0 until minLen) {
            if (w1[j] != w2[j]) {
                graph[w1[j]]!!.add(w2[j])
                foundDifference = true
                break
            }
        }
        if (!foundDifference && w1.length > w2.length) return ""
    }

    val UNVISITED = 0; val VISITING = 1; val VISITED = 2
    val state = HashMap<Char, Int>()
    val result = StringBuilder()
    var hasCycle = false

    fun dfs(c: Char) {
        if (hasCycle || state[c] == VISITED) return
        if (state[c] == VISITING) { hasCycle = true; return }

        state[c] = VISITING
        for (next in graph[c]!!) dfs(next)
        state[c] = VISITED

        result.append(c)   // append AFTER all descendants — builds reverse topological order
    }

    for (c in graph.keys) dfs(c)

    if (hasCycle) return ""

    return result.reverse().toString()
}
```

---

## Why Only ADJACENT Word Pairs Need to Be Compared

This is the key reduction that makes the problem tractable — comparing
every pair of words would be O(n² × wordLength), but comparing only
adjacent pairs is sufficient and far cheaper:

```kotlin
for (i in 0 until words.size - 1) {
    val w1 = words[i]
    val w2 = words[i + 1]
    // ... extract ONE constraint from this adjacent pair
}
```

If the entire word list is sorted, any ordering relationship between
two **non-adjacent** words is already implied **transitively** through
the chain of adjacent comparisons between them. For example, if
`words[0] < words[1]` and `words[1] < words[2]`, then
`words[0] < words[2]` is automatically true without needing to compare
them directly — exactly mirroring how a topologically sorted list
already encodes all transitive ordering relationships, not just direct
ones.

**Why the "prefix" edge case matters**
(`if (!foundDifference && w1.length > w2.length) return ""`): if `w1` is
a strict prefix of `w2` and yet appears in the list believed to come
**after** `w2` (i.e., `w1` is longer but no character differs within the
shared prefix), that's inherently contradictory — a shorter word that's
a prefix of a longer one must always sort before it in any valid
lexicographic ordering.

**Why `result.length == graph.size` (BFS) detects cycles:** if a cycle
exists among some subset of characters, those characters' in-degrees
never reach 0, so they're never enqueued or added to `result` — the
final result coming up short reveals the cycle, exactly like Course
Schedule II's same completeness check.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| BFS / Kahn's algorithm (Approach 1) | Direct extension of Course Schedule II's BFS approach |
| DFS (Approach 2) | Prefer recursive cycle detection, consistent with Course Schedule's DFS approach |

---

## Complexity

| | |
|---|---|
| **Time** | O(C) where C is the total length of all words — extracting constraints; O(V + E) for the topological sort itself, V,E bounded by alphabet size (≤26) |
| **Space** | O(1) — bounded by a fixed alphabet size, plus O(C) for scanning the words |

---

## Key Takeaway

> Alien Dictionary combines two skills from earlier in this phase (and
> the Graphs phase before it): deriving a graph's edges from indirect,
> structural clues (only adjacent word pairs carry new information,
> thanks to the transitivity of sorted order), then running standard
> topological sort — either BFS/Kahn's or DFS — exactly as in Course
> Schedule and Course Schedule II. This problem is an excellent capstone
> example of how "building the graph" is sometimes the harder half of a
> graph problem, with the actual traversal algorithm being something
> you've already mastered.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/alien-dictionary/)

---

{% include kotlin-dsa-nav.html %}
