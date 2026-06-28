---
title: "1-D DP: Min Cost Climbing Stairs — Kotlin Solution"
date: 2026-06-26
categories: [Kotlin DSA, 1-D Dynamic Programming]
tags: [min-cost-climbing-stairs, dynamic-programming, array, easy, leetcode, leetcode-746]
leetcode_number: 746
leetcode_url: https://leetcode.com/problems/min-cost-climbing-stairs/
difficulty: Easy
permalink: /posts/kotlin-dsa-dp-min-cost-climbing-stairs/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [746 — Min Cost Climbing Stairs](https://leetcode.com/problems/min-cost-climbing-stairs/) |
| **Difficulty** | Easy |
| **Topic** | Dynamic Programming, Array |

> Given an array `cost` where `cost[i]` is the cost of stepping on stair
> `i`, you may start at index `0` or `1`, and from any step `i` you can
> climb to `i+1` or `i+2`. Return the **minimum cost** to reach the top
> (one step past the last index).

**Example:**
```
Input:  cost = [10,15,20]
Output: 15
Explanation: start at index 1 (cost 15), then step directly to the top
             (i+2), skipping index 2 entirely

Input:  cost = [1,100,1,1,1,100,1,1,100,1]
Output: 6
Explanation: start at index 0, then 0->2->3->4->6->7->9->top, paying
             1+1+1+1+1+1=6
```

**Constraints:**
- `2 <= cost.length <= 1000`
- `0 <= cost[i] <= 999`

---

## Approach

Builds directly on Climbing Stairs' recurrence, with one twist: instead
of counting **ways**, we're minimizing **cost**, and we land one step
**beyond** the array's end (the "top").

**Key insight:** Define `minCost[i]` as the minimum cost to **reach**
step `i` (not including the cost of stepping past it). Since you can
arrive at step `i` from either `i-1` or `i-2`, take whichever was
cheaper, then add the cost of actually standing on step `i`:
`minCost[i] = cost[i] + min(minCost[i-1], minCost[i-2])`.

The answer is `minCost` for the position **one past the last index**
(the "top"), which itself costs nothing to stand on — its cost is purely
determined by the cheapest way to have arrived there.

**Walk through `cost = [10,15,20]`:**
```
minCost[0] = 0   (starting here is free — no cost to "arrive" at the start)
minCost[1] = 0   (can also start here for free)

minCost[2] = cost[2] + min(minCost[1], minCost[0]) = 20 + min(0,0) = 20
minCost[3] (the "top", one past index 2) = min(minCost[2], minCost[1])
          = min(20, 0) = 0... 

Wait — let's recheck: minCost[top] should consider stepping from index 1
directly (skipping index 2 with a +2 jump) OR from index 2.
minCost[top] = min(minCost[2] [+0, no cost for being "at the top"],
                    minCost[1] [+0])
             but minCost[1] = 0 doesn't include cost[1]=15 yet!

Corrected understanding: minCost[i] should represent cost TO REACH step i
INCLUDING cost[i] itself, for i within bounds. Let's redo:

minCost[0] = cost[0] = 10
minCost[1] = cost[1] = 15
minCost[2] = cost[2] + min(minCost[0], minCost[1]) = 20 + min(10,15) = 30

Top = min(minCost[1], minCost[2]) = min(15, 30) = 15 ✓ (matches expected output)
```

---

## Kotlin Solution

### Approach 1 — Bottom-up DP, O(1) space, two rolling variables (optimal)

```kotlin
fun minCostClimbingStairs(cost: IntArray): Int {
    var prev2 = 0   // cost to reach index 0 from "before the start" (free)
    var prev1 = 0   // cost to reach index 1 from "before the start" (free)

    for (i in cost.indices) {
        val current = cost[i] + minOf(prev1, prev2)
        prev2 = prev1
        prev1 = current
    }

    return minOf(prev1, prev2)   // the "top" can be reached from either of the last two steps
}
```

### Approach 2 — Top-down memoization

```kotlin
fun minCostClimbingStairs(cost: IntArray): Int {
    val n = cost.size
    val memo = HashMap<Int, Int>()

    fun dp(i: Int): Int {
        if (i >= n) return 0   // reached or passed the top — no more cost
        return memo.getOrPut(i) { cost[i] + minOf(dp(i + 1), dp(i + 2)) }
    }

    return minOf(dp(0), dp(1))   // starting choice: index 0 or index 1
}
```

---

## Why the Base Cases Start at 0, Not `cost[0]`/`cost[1]`

This is the trickiest part of getting the iterative solution right.
`prev2` and `prev1` represent the cost to arrive **at** index 0 and index
1 respectively, **before** paying for standing on them — both are free
starting points per the problem statement, hence both initialize to `0`:

```kotlin
var prev2 = 0   // arriving "at" index 0 costs nothing extra (free start)
var prev1 = 0   // arriving "at" index 1 costs nothing extra (free start)

for (i in cost.indices) {
    val current = cost[i] + minOf(prev1, prev2)
    // NOW we pay cost[i] — this models "the cost of actually being on step i"
}
```

**Why the final answer is `minOf(prev1, prev2)`, not just the last
computed value:** after the loop finishes, `prev1` holds the cost to
reach the **last** index, and `prev2` holds the cost to reach the
**second-to-last** index. The "top" (one step past the end) can be
reached from either of these two positions — whichever is cheaper:
```kotlin
return minOf(prev1, prev2)
// top can be reached via a +1 jump from prev1's position,
// or a +2 jump from prev2's position — no additional cost for the
// jump itself, only the cost already accumulated to REACH that position
```

**Why the top-down version's base case is `i >= n`, not `i == n`:** since
jumps of size 2 are allowed, recursion could land exactly on `n` (one
past the last valid index) or could theoretically be checked from `n-1`
jumping by 2 to `n+1` — using `>=` safely covers both, treating "anything
at or beyond the top" as having zero remaining cost.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Bottom-up, O(1) space (Approach 1) | Always — most efficient |
| Top-down memoization (Approach 2) | Prefer recursive framing, comfortable with O(n) space |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) |
| **Space** | O(1) bottom-up / O(n) memoized |

---

## Key Takeaway

> Min Cost Climbing Stairs takes Climbing Stairs' "depends on the two
> previous states" recurrence and swaps the **aggregation** from sum (
> counting ways) to min (minimizing cost) — the recurrence shape stays
> identical, only the combining operation changes. This is a recurring
> theme across DP problems: once you've identified the *states* and how
> they relate, the same skeleton can answer "how many ways," "minimum
> cost," or "maximum value" questions just by swapping `+` for `min` or
> `max` at the combination step.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/min-cost-climbing-stairs/)

---

{% include kotlin-dsa-nav.html %}
