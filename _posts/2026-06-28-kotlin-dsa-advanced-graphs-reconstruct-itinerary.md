---
title: "Advanced Graphs: Reconstruct Itinerary — Kotlin Solution"
date: 2026-06-28
categories: [Kotlin DSA, Advanced Graphs]
tags: [reconstruct-itinerary, graph, dfs, eulerian-path, hard, leetcode, leetcode-332]
leetcode_number: 332
leetcode_url: https://leetcode.com/problems/reconstruct-itinerary/
difficulty: Hard
permalink: /posts/kotlin-dsa-advanced-graphs-reconstruct-itinerary/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [332 — Reconstruct Itinerary](https://leetcode.com/problems/reconstruct-itinerary/) |
| **Difficulty** | Hard |
| **Topic** | Graph, DFS, Eulerian Path |

> Given a list of airline `tickets` (each `[from, to]`), reconstruct the
> itinerary starting from `"JFK"` that uses **all** tickets exactly
> once. If multiple valid itineraries exist, return the
> **lexicographically smallest** one.

**Example:**
```
Input:  tickets = [["MUC","LHR"],["JFK","MUC"],["SFO","SJC"],["LHR","SFO"]]
Output: ["JFK","MUC","LHR","SFO","SJC"]

Input:  tickets = [["JFK","SFO"],["JFK","ATL"],["SFO","ATL"],["ATL","JFK"],["ATL","SFO"]]
Output: ["JFK","ATL","JFK","SFO","ATL","SFO"]
Explanation: tickets can be reused as separate entries — the route
             revisits JFK and uses all 5 tickets
```

**Constraints:**
- `1 <= tickets.length <= 300`
- `tickets[i].length == 2`
- All airport codes are 3 uppercase letters
- It's guaranteed that a valid itinerary using all tickets exists

---

## Approach

This is finding an **Eulerian path** — a path using every edge exactly
once — in a directed graph, with the added twist of needing the
**lexicographically smallest** such path among possibly multiple valid
options.

**Key insight — greedy DFS with sorted destinations, backtracking on
dead ends:** sort each airport's list of destinations alphabetically.
DFS from `"JFK"`, always trying the **smallest** available next
destination first (greedy choice for lexicographic minimality). If a
chosen path leads to a dead end **before** all tickets are used, that
specific choice was wrong — backtrack and try the next-smallest
alternative.

**Walk through `tickets = [["JFK","SFO"],["JFK","ATL"],["SFO","ATL"],
["ATL","JFK"],["ATL","SFO"]]`:**
```
adjacency (sorted): JFK->[ATL,SFO], ATL->[JFK,SFO], SFO->[ATL]

dfs("JFK"): try smallest unused destination "ATL" first
  dfs("ATL"): try smallest unused "JFK" first
    dfs("JFK"): try smallest unused "SFO" (ATL already used)
      dfs("SFO"): try "ATL" (only option)
        dfs("ATL"): try "SFO" (only remaining option)
          dfs("SFO"): no more destinations, all tickets used
            → path complete, add to itinerary in REVERSE order as we return

Itinerary (built in reverse, then reversed): 
  JFK -> ATL -> JFK -> SFO -> ATL -> SFO ✓
```

---

## Kotlin Solution

### Approach 1 — Greedy DFS, sorted adjacency, build path in reverse (Hierholzer's-style)

```kotlin
fun findItinerary(tickets: List<List<String>>): List<String> {
    val graph = HashMap<String, MutableList<String>>()
    for ((from, to) in tickets) {
        graph.getOrPut(from) { mutableListOf() }.add(to)
    }
    for (destinations in graph.values) {
        destinations.sortDescending()   // sort descending so we can efficiently removeAt(last)
    }

    val route = mutableListOf<String>()

    fun dfs(airport: String) {
        val destinations = graph[airport]
        while (!destinations.isNullOrEmpty()) {
            val next = destinations.removeAt(destinations.lastIndex)   // smallest remaining (since sorted descending)
            dfs(next)
        }
        route.add(airport)   // add AFTER exhausting all destinations — builds path in reverse
    }

    dfs("JFK")
    return route.reversed()
}
```

### Approach 2 — Greedy DFS with explicit backtracking (more literal, handles dead ends explicitly)

```kotlin
fun findItinerary(tickets: List<List<String>>): List<String> {
    val graph = HashMap<String, MutableList<String>>()
    for ((from, to) in tickets) {
        graph.getOrPut(from) { mutableListOf() }.add(to)
    }
    for (destinations in graph.values) destinations.sort()

    val used = HashMap<String, BooleanArray>()
    for ((airport, destinations) in graph) {
        used[airport] = BooleanArray(destinations.size)
    }

    val totalTickets = tickets.size
    val path = mutableListOf("JFK")

    fun backtrack(airport: String): Boolean {
        if (path.size == totalTickets + 1) return true

        val destinations = graph[airport] ?: return false
        val usedFlags = used[airport]!!

        for (i in destinations.indices) {
            if (!usedFlags[i]) {
                usedFlags[i] = true
                path.add(destinations[i])

                if (backtrack(destinations[i])) return true

                path.removeAt(path.lastIndex)   // undo — this choice didn't lead to a full solution
                usedFlags[i] = false
            }
        }

        return false
    }

    backtrack("JFK")
    return path
}
```

Explicitly backtracks (undoing a choice) if it leads to a dead end before
all tickets are consumed — more directly shows the "try smallest first,
undo on failure" backtracking mechanic, though Approach 1 achieves the
same correct result more elegantly via a different traversal order
property (explained below).

---

## Why Building the Path in REVERSE Order Works (Approach 1)

This is the cleverest and least obvious part of Approach 1, related to a
classical graph theory result (Hierholzer's algorithm for Eulerian
paths):

```kotlin
fun dfs(airport: String) {
    while (!destinations.isNullOrEmpty()) {
        val next = destinations.removeAt(destinations.lastIndex)
        dfs(next)
    }
    route.add(airport)   // only added once ALL this airport's tickets are exhausted
}
```

An airport only gets added to `route` **after** every outgoing ticket
from it has been used (the `while` loop only exits once `destinations`
is empty). This means a "dead-end" airport — one that gets fully
exhausted early because it has no further unused tickets — is recorded
**first**, and the true starting airport `"JFK"` is recorded **last**
(since it's the very last call to fully exhaust its destinations,
sitting at the bottom of the call stack). Reversing the final list
restores the correct forward chronological order. Critically, this
specific construction is mathematically guaranteed to produce a valid
Eulerian path when one exists — no explicit backtracking is needed,
because any "wrong-looking" early exhaustion is automatically handled
correctly by the reverse-order construction.

**Why Approach 2's explicit backtracking is more intuitive but
less elegant:** it directly simulates "try the smallest option, and if
it doesn't pan out, undo and try the next" — which is conceptually
simpler to follow, but requires explicit dead-end detection and undo
logic that Approach 1's clever traversal order avoids entirely.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Reverse-order DFS (Approach 1) | More elegant and efficient once understood; the standard competitive-programming answer |
| Explicit backtracking (Approach 2) | Easier to reason about and verify correctness for newcomers to Eulerian path problems |

---

## Complexity

| | |
|---|---|
| **Time** | O(E log E) — E = number of tickets; dominated by sorting each airport's destination list |
| **Space** | O(E) — the adjacency list and the route/path |

---

## Key Takeaway

> Reconstruct Itinerary is an Eulerian path problem — using every edge
> exactly once — solved greedily by always trying the lexicographically
> smallest next destination. The truly elegant insight (Approach 1) is
> that building the result by appending nodes **after** they're fully
> exhausted, then reversing, automatically produces a correct path
> without any explicit backtracking — a beautiful but non-obvious
> consequence of how Eulerian paths are structured. This problem is a
> great example of how some graph algorithms have deep theoretical
> shortcuts that look almost like a trick until you understand the
> underlying graph theory.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/reconstruct-itinerary/)

---

{% include kotlin-dsa-nav.html %}
