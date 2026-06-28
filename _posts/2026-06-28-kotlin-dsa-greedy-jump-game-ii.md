---
title: "Greedy: Jump Game II — Kotlin Solution"
date: 2026-06-28
categories: [Kotlin DSA, Greedy]
tags: [jump-game-ii, greedy, bfs, array, medium, leetcode, leetcode-45]
leetcode_number: 45
leetcode_url: https://leetcode.com/problems/jump-game-ii/
difficulty: Medium
permalink: /posts/kotlin-dsa-greedy-jump-game-ii/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [45 — Jump Game II](https://leetcode.com/problems/jump-game-ii/) |
| **Difficulty** | Medium |
| **Topic** | Greedy, BFS, Array |

> Given an array `nums` where `nums[i]` is the maximum jump length from
> index `i`, return the **minimum number of jumps** to reach the last
> index. It's guaranteed the last index is always reachable.

**Example:**
```
Input:  nums = [2,3,1,1,4]
Output: 2
Explanation: jump 1 step from index 0 to 1, then 3 steps to the last index

Input:  nums = [2,3,0,1,4]
Output: 2
```

**Constraints:**
- `1 <= nums.length <= 10⁴`
- `0 <= nums[i] <= 1000`
- The last index is always reachable

---

## Approach

This extends Jump Game from "can we reach the end" to "what's the
**minimum** number of jumps" — and the natural mental model is **BFS
level-by-level expansion**, even though we never build an explicit graph
or queue.

**Key insight — think in terms of "jump boundaries":** at any point,
you're within some **range** reachable using `k` jumps so far (this is
exactly one "level" of BFS). Track the **current jump's reachable
boundary** (`currentEnd`) and the farthest boundary achievable with
**one more** jump (`farthest`), scanning every position within the
current range. The instant you've scanned through the entire current
range, you must commit to a jump — increment the jump count, and the new
boundary becomes whatever `farthest` had accumulated.

**Walk through `nums = [2,3,1,1,4]`:**
```
jumps=0, currentEnd=0, farthest=0

i=0: farthest = max(0, 0+2) = 2
  i == currentEnd(0) → must jump now: jumps=1, currentEnd=farthest=2

i=1: farthest = max(2, 1+3) = 4
i=2: farthest = max(4, 2+1) = 4
  i == currentEnd(2) → must jump now: jumps=2, currentEnd=farthest=4

i=3: farthest = max(4, 3+1) = 4
i=4: we've reached the last index — stop (no need to process further)

Total jumps: 2 ✓
```

---

## Kotlin Solution

### Approach 1 — Greedy BFS-style level expansion (optimal)

```kotlin
fun jump(nums: IntArray): Int {
    var jumps = 0
    var currentEnd = 0
    var farthest = 0

    for (i in 0 until nums.lastIndex) {   // no need to process the last index itself
        farthest = maxOf(farthest, i + nums[i])

        if (i == currentEnd) {
            jumps++
            currentEnd = farthest
        }
    }

    return jumps
}
```

### Approach 2 — Explicit BFS with a queue (more literal, less efficient)

```kotlin
fun jump(nums: IntArray): Int {
    val n = nums.size
    if (n == 1) return 0

    val visited = BooleanArray(n)
    val queue: ArrayDeque<Int> = ArrayDeque()
    queue.addLast(0)
    visited[0] = true
    var jumps = 0

    while (queue.isNotEmpty()) {
        jumps++
        repeat(queue.size) {
            val curr = queue.removeFirst()
            for (next in curr + 1..minOf(curr + nums[curr], n - 1)) {
                if (next == n - 1) return jumps
                if (!visited[next]) {
                    visited[next] = true
                    queue.addLast(next)
                }
            }
        }
    }

    return jumps
}
```

Treats each index as a graph node and each possible jump as an edge,
running literal BFS to find the shortest path — correct, but O(n × max
jump length) in the worst case, since every reachable index from a given
position gets explicitly enqueued, including many that the greedy
approach implicitly skips over.

---

## Why the Greedy Approach Mirrors BFS Without an Explicit Queue

This is the conceptual leap worth understanding precisely:

```kotlin
if (i == currentEnd) {
    jumps++
    currentEnd = farthest
}
```

`currentEnd` represents the boundary of the current BFS "level" — every
index from the start of this level up through `currentEnd` is reachable
using exactly `jumps` jumps. As we scan through this range, `farthest`
accumulates the best possible boundary for the **next** level (one
additional jump). The moment we've scanned the entirety of the current
level (`i == currentEnd`), we commit to advancing to the next level —
incrementing `jumps` and adopting `farthest` as the new boundary.

This achieves the exact same level-by-level guarantee as explicit BFS —
every index is implicitly assigned to the level corresponding to its
minimum jump count — but without ever maintaining a queue or visited
array, because the **boundaries** themselves are sufficient information.

**Why we stop scanning before `nums.lastIndex`:** once we know the
farthest reachable boundary covers the last index, there's no need to
process it explicitly — we just need to ensure we've committed enough
jumps to cover it, which happens naturally as the loop processes earlier
indices.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Greedy boundary tracking (Approach 1) | Always — O(n) time, O(1) space, the optimal solution |
| Explicit BFS (Approach 2) | Want to see the underlying graph-theoretic justification explicitly |

---

## Complexity

| | Greedy | Explicit BFS |
|---|---|---|
| **Time** | O(n) | O(n × average jump length), can degrade toward O(n²) |
| **Space** | O(1) | O(n) — visited array and queue |

---

## Key Takeaway

> Jump Game II reveals that many greedy array-scanning algorithms are
> secretly performing BFS "in spirit" — maintaining implicit level
> boundaries instead of an explicit queue. The `currentEnd`/`farthest`
> pair tracks exactly what BFS's queue-draining-per-level would track,
> but compressed into O(1) space because we only need boundary
> *positions*, not the full set of nodes at each level. This connection
> between greedy boundary-tracking and BFS level-order traversal is a
> powerful pattern-recognition tool: whenever a problem asks for
> "minimum steps to reach a target" over an implicit reachability
> graph, consider whether the graph's structure allows this kind of
> boundary compression.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/jump-game-ii/)

---

{% include kotlin-dsa-nav.html %}
