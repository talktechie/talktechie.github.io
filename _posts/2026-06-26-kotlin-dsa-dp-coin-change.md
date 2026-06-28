---
title: "1-D DP: Coin Change — Kotlin Solution"
date: 2026-06-26
categories: [Kotlin DSA, 1-D Dynamic Programming]
tags: [coin-change, dynamic-programming, array, bfs, medium, leetcode, leetcode-322]
leetcode_number: 322
leetcode_url: https://leetcode.com/problems/coin-change/
difficulty: Medium
permalink: /posts/kotlin-dsa-dp-coin-change/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [322 — Coin Change](https://leetcode.com/problems/coin-change/) |
| **Difficulty** | Medium |
| **Topic** | Dynamic Programming, Array, BFS |

> Given coin denominations `coins` and a target `amount`, return the
> **fewest number of coins** needed to make up `amount` (unlimited
> supply of each coin), or `-1` if impossible.

**Example:**
```
Input:  coins = [1,2,5], amount = 11
Output: 3
Explanation: 5 + 5 + 1 = 11, using 3 coins

Input:  coins = [2], amount = 3
Output: -1
Explanation: odd amount, only even-valued coin — impossible
```

**Constraints:**
- `1 <= coins.length <= 12`
- `1 <= coins[i] <= 2³¹ - 1`
- `0 <= amount <= 10⁴`

---

## Approach

**Key insight:** Build up the answer for every amount from `0` to
`target`, using already-solved smaller amounts. For amount `a`, try
**every coin** as the "last coin used" — if we used coin `c` last, the
remaining amount `a - c` must have already been optimally solved, so
`dp[a] = 1 + dp[a - c]`, minimized over every valid coin choice.

**Walk through `coins = [1,2,5], amount = 11`** (partial trace):
```
dp[0] = 0   (zero coins needed for amount 0 — base case)

dp[1] = 1 + dp[0] = 1            (using coin 1)
dp[2] = min(1+dp[1], 1+dp[0]) = min(2, 1) = 1    (coin 1 twice vs coin 2 once)
dp[3] = min(1+dp[2], 1+dp[1]) = min(2, 2) = 2
dp[4] = min(1+dp[3], 1+dp[2]) = min(3, 2) = 2
dp[5] = min(1+dp[4], 1+dp[3], 1+dp[0]) = min(3,3,1) = 1   (single coin of 5!)
dp[6] = min(1+dp[5], 1+dp[4], 1+dp[1]) = min(2,3,2) = 2
... continues building up ...
dp[11] = min(1+dp[10], 1+dp[9], 1+dp[6])
       = min(1+2, 1+3, 1+2) = min(3,4,3) = 3

Answer: 3 ✓
```

---

## Kotlin Solution

### Approach 1 — Bottom-up DP, build from amount 0 upward (optimal)

```kotlin
fun coinChange(coins: IntArray, amount: Int): Int {
    val dp = IntArray(amount + 1) { Int.MAX_VALUE }
    dp[0] = 0

    for (a in 1..amount) {
        for (coin in coins) {
            if (coin <= a && dp[a - coin] != Int.MAX_VALUE) {
                dp[a] = minOf(dp[a], dp[a - coin] + 1)
            }
        }
    }

    return if (dp[amount] == Int.MAX_VALUE) -1 else dp[amount]
}
```

### Approach 2 — Top-down memoization

```kotlin
fun coinChange(coins: IntArray, amount: Int): Int {
    val memo = HashMap<Int, Int>()

    fun dp(remaining: Int): Int {
        if (remaining == 0) return 0
        if (remaining < 0) return -1

        return memo.getOrPut(remaining) {
            var best = Int.MAX_VALUE
            for (coin in coins) {
                val result = dp(remaining - coin)
                if (result != -1) best = minOf(best, result + 1)
            }
            if (best == Int.MAX_VALUE) -1 else best
        }
    }

    return dp(amount)
}
```

### Approach 3 — BFS, treating amounts as graph nodes (alternative framing)

```kotlin
fun coinChange(coins: IntArray, amount: Int): Int {
    if (amount == 0) return 0

    val visited = BooleanArray(amount + 1)
    val queue: ArrayDeque<Int> = ArrayDeque()
    queue.addLast(0)
    visited[0] = true
    var steps = 0

    while (queue.isNotEmpty()) {
        steps++
        repeat(queue.size) {
            val curr = queue.removeFirst()
            for (coin in coins) {
                val next = curr + coin
                if (next == amount) return steps
                if (next < amount && !visited[next]) {
                    visited[next] = true
                    queue.addLast(next)
                }
            }
        }
    }

    return -1
}
```

Reframes the problem as a graph: each amount is a node, and adding any
coin is an edge to a new amount. Finding the minimum number of coins is
then exactly "shortest path from 0 to amount" — solvable with BFS,
reusing the same shortest-path intuition from the Graphs phase.

---

## Why `Int.MAX_VALUE` as the "Unreachable" Sentinel Needs a Guard

```kotlin
if (coin <= a && dp[a - coin] != Int.MAX_VALUE) {
    dp[a] = minOf(dp[a], dp[a - coin] + 1)
}
```

Without the `dp[a - coin] != Int.MAX_VALUE` check, adding `1` to an
already-`Int.MAX_VALUE` "unreachable" sentinel could overflow or simply
produce a misleadingly small (wrapped-around) number, incorrectly
suggesting that amount is reachable when it isn't. This guard ensures we
only build on top of *genuinely* reachable sub-amounts.

**Why this is fundamentally an "unbounded knapsack" shape:** unlike
House Robber's "use this item or don't, moving strictly forward,"
Coin Change allows reusing the **same** coin denomination multiple
times — which is why the inner loop tries every coin **without**
advancing past it, similar in spirit to Combination Sum's "pass `i`, not
`i+1`" reuse mechanic from the Backtracking phase, just expressed here
through DP table dependencies instead of explicit recursion indices.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Bottom-up DP (Approach 1) | Always — clean, iterative, the standard solution |
| Top-down memoization (Approach 2) | Prefer recursive framing |
| BFS (Approach 3) | Want to see the graph-theoretic connection; same time complexity, different mental model |

---

## Complexity

| | |
|---|---|
| **Time** | O(amount × coins.length) |
| **Space** | O(amount) |

---

## Key Takeaway

> Coin Change is the classic **unbounded knapsack** DP shape: build up
> answers for every amount from the smallest to the largest, and at each
> amount, try every available "item" (coin) as the most recent addition,
> taking whichever choice minimizes the total count. The BFS framing
> reveals a deeper truth — many "minimum steps/coins/operations to reach
> a target" DP problems are secretly shortest-path problems on an
> implicit graph, and recognizing that connection broadens which tools
> you can reach for, even when a DP table is ultimately the cleanest
> implementation.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/coin-change/)

---

{% include kotlin-dsa-nav.html %}
