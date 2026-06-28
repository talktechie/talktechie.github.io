---
title: "Advanced Graphs: Cheapest Flights Within K Stops — Kotlin Solution"
date: 2026-06-28
categories: [Kotlin DSA, Advanced Graphs]
tags: [cheapest-flights-within-k-stops, graph, bellman-ford, dijkstra, medium, leetcode, leetcode-787]
leetcode_number: 787
leetcode_url: https://leetcode.com/problems/cheapest-flights-within-k-stops/
difficulty: Medium
permalink: /posts/kotlin-dsa-advanced-graphs-cheapest-flights-within-k-stops/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [787 — Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/) |
| **Difficulty** | Medium |
| **Topic** | Graph, Bellman-Ford, Dijkstra-style |

> Given `n` cities, flights `[from, to, price]`, a source `src`,
> destination `dst`, and a limit of **at most `k` stops** (meaning at
> most `k+1` flights/edges total), return the cheapest price, or `-1` if
> unreachable within that constraint.

**Example:**
```
Input:  n = 4, flights = [[0,1,100],[1,2,100],[2,0,100],[1,3,600],[2,3,200]],
        src = 0, dst = 3, k = 1
Output: 700
Explanation: 0->1->3 costs 100+600=700, using exactly 1 stop (city 1).
             0->1->2->3 would be cheaper (100+100+200=400) but uses 2
             stops, exceeding k=1.

Input:  n = 3, flights = [[0,1,100],[1,2,100],[0,2,500]], src = 0, dst = 2, k = 1
Output: 200
```

**Constraints:**
- `1 <= n <= 100`
- `0 <= flights.length <= (n × (n-1)) / 2`
- `0 <= src, dst, k < n`, `src != dst`

---

## Approach

Plain Dijkstra's doesn't directly respect a **hop-count limit** — it
only tracks cumulative cost, not how many edges were used to get there.
This problem needs **Bellman-Ford**, specifically because it naturally
processes the graph in **rounds**, where each round corresponds to
allowing exactly **one more edge** in any path — a perfect match for "at
most k+1 edges."

**Key insight:** Run Bellman-Ford's edge relaxation for exactly `k+1`
rounds (since `k` stops means `k+1` flights). In each round, relax every
edge **using a snapshot of the previous round's distances** — critically,
NOT updating distances in-place mid-round, since that could let a single
round's update "leak" extra hops into the same round, violating the
strict per-round edge-count guarantee.

**Walk through `n = 4, flights = [[0,1,100],[1,2,100],[2,0,100],
[1,3,600],[2,3,200]], src = 0, dst = 3, k = 1`:**
```
dist = [0, ∞, ∞, ∞]   (k+1 = 2 rounds allowed)

Round 1 (using snapshot from round 0): relax all edges using dist[] as-is
  edge 0->1 (100): dist[1] candidate = 0+100=100 (better than ∞)
  edge 1->2 (100): dist[2] candidate = dist[1](∞, OLD snapshot)+100 = ∞ (no improvement)
  edge 2->0 (100): no improvement
  edge 1->3 (600): dist[3] candidate = dist[1](∞ old)+600 = ∞ (no improvement YET)
  edge 2->3 (200): no improvement
  Apply round 1's updates: dist = [0, 100, ∞, ∞]

Round 2 (using snapshot from round 1: [0,100,∞,∞]):
  edge 0->1: dist[1] candidate = 0+100=100 (no change)
  edge 1->2: dist[2] candidate = 100+100=200 (improvement!)
  edge 2->0: no improvement
  edge 1->3: dist[3] candidate = 100+600=700 (improvement!)
  edge 2->3: dist[3] candidate = dist[2](∞ OLD snapshot, not this round's 200)+200 = ∞
  Apply round 2's updates: dist = [0, 100, 200, 700]

After k+1=2 rounds: dist[3] = 700 ✓ (matches expected output)
```

---

## Kotlin Solution

### Approach 1 — Bellman-Ford, exactly k+1 rounds, using a snapshot per round (optimal)

```kotlin
fun findCheapestPrice(n: Int, flights: Array<IntArray>, src: Int, dst: Int, k: Int): Int {
    var dist = IntArray(n) { Int.MAX_VALUE }
    dist[src] = 0

    repeat(k + 1) {
        val newDist = dist.copyOf()   // snapshot — this round reads ONLY from the previous round's results

        for ((u, v, price) in flights) {
            if (dist[u] != Int.MAX_VALUE && dist[u] + price < newDist[v]) {
                newDist[v] = dist[u] + price
            }
        }

        dist = newDist
    }

    return if (dist[dst] == Int.MAX_VALUE) -1 else dist[dst]
}
```

### Approach 2 — Modified Dijkstra's, tracking (cost, stops) pairs in the heap

```kotlin
fun findCheapestPrice(n: Int, flights: Array<IntArray>, src: Int, dst: Int, k: Int): Int {
    val graph = Array(n) { mutableListOf<Pair<Int, Int>>() }
    for ((u, v, price) in flights) graph[u].add(v to price)

    val heap = PriorityQueue<IntArray>(compareBy { it[0] })  // [cost, node, stopsUsed]
    heap.add(intArrayOf(0, src, 0))

    val minStopsAtCost = HashMap<Int, Int>()   // node -> fewest stops used to reach it so far

    while (heap.isNotEmpty()) {
        val (cost, node, stops) = heap.poll()

        if (node == dst) return cost

        if (stops > k) continue   // exceeded the stop limit — prune this path

        if (minStopsAtCost.getOrDefault(node, Int.MAX_VALUE) <= stops) continue   // already found a path here using fewer-or-equal stops
        minStopsAtCost[node] = stops

        for ((next, price) in graph[node]) {
            heap.add(intArrayOf(cost + price, next, stops + 1))
        }
    }

    return -1
}
```

A trickier adaptation of Dijkstra's that also tracks stop count
alongside cost, pruning paths that exceed `k` stops or that revisit a
node with no improvement in stops — more intricate to get exactly right
than the Bellman-Ford approach, but demonstrates that Dijkstra's CAN be
adapted with extra state when a simple cost-only greedy isn't sufficient.

---

## Why Bellman-Ford's Round Structure Is the Natural Fit (And Why the Snapshot Matters)

This is the single most important implementation detail in the entire
problem, and getting it wrong is a very common bug:

```kotlin
val newDist = dist.copyOf()   // CRITICAL — snapshot BEFORE this round's updates

for ((u, v, price) in flights) {
    if (dist[u] != Int.MAX_VALUE && dist[u] + price < newDist[v]) {
        newDist[v] = dist[u] + price
    }
}

dist = newDist   // commit the whole round's results AFTER processing every edge
```

If we updated `dist` **in-place** during a single round (instead of
writing to a separate `newDist` and reading from the unchanged `dist`),
an edge processed later in the **same** round could incorrectly build on
an update from **earlier in that same round** — effectively allowing a
path to use **two** edges within what should count as just **one**
round (one additional hop). This would silently violate the `k` stops
constraint, since the algorithm's entire correctness here hinges on
"each round = exactly one more edge used," not "each round = one or more
edges, however many happen to chain together."

**Why exactly `k + 1` rounds (not `k`):** the problem limits **stops**
(intermediate cities), and "at most `k` stops" means "at most `k+1`
flights/edges" — a direct flight (0 stops) is 1 edge, one stop is 2
edges, and so on.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Bellman-Ford with per-round snapshot (Approach 1) | Always — cleanest direct fit for a hop-limited shortest path, the standard solution |
| Modified Dijkstra's with stop tracking (Approach 2) | Want to see how Dijkstra's can be extended with extra state, though more error-prone to implement correctly |

---

## Complexity

| | Bellman-Ford | Modified Dijkstra's |
|---|---|---|
| **Time** | O(k × E) — k+1 rounds, each O(E) | O(E × k × log(E × k)) worst case, due to expanded state space |
| **Space** | O(n) | O(n × k) — tracking stops per node |

---

## Key Takeaway

> When a shortest-path problem has a **hop-count constraint**, Dijkstra's
> greedy "always expand cheapest" choice can't directly enforce that
> limit, but Bellman-Ford's round-based structure maps onto it
> perfectly — each round corresponds to exactly one additional edge
> across the entire graph. The critical implementation detail is reading
> from a frozen snapshot of the **previous** round's distances while
> writing to a fresh array for the **current** round, ensuring no edge
> accidentally "borrows" progress from later in the same round. This
> closes out the Advanced Graphs phase by showing that choosing between
> Dijkstra's and Bellman-Ford often comes down to which one's *structural
> assumptions* — non-negative weights and greedy frontier expansion vs.
> systematic round-by-round relaxation — actually match the problem's
> specific constraints.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/cheapest-flights-within-k-stops/)

---

{% include kotlin-dsa-nav.html %}
