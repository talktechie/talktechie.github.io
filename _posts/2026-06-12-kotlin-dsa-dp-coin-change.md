---
title: "Dynamic Programming: Coin Change — Kotlin Solution"
date: 2026-06-12
categories: [Kotlin DSA, Dynamic Programming]
tags: [coin-change, dynamic-programming, bfs, medium, leetcode, leetcode-322]
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
| **Topic** | Dynamic Programming, BFS |

> Given an array of coin denominations and a target `amount`,
> return the fewest number of coins needed to make up that amount.
> Return `-1` if it cannot be made.

**Example:**
```
Input:  coins = [1, 5, 10], amount = 11
Output: 2   // 10 + 1

Input:  coins = [2], amount = 3
Output: -1  // cannot make 3 with only 2s
```

---

## Approach

Build a `dp` array where `dp[i]` = minimum coins to make amount `i`.

Start with `dp[0] = 0` (zero coins to make amount 0).
For every amount from 1 to `amount`, try every coin.
If `amount - coin >= 0`, you can use this coin:
`dp[i] = min(dp[i], dp[i - coin] + 1)`

**Walk through `coins=[1,5,10], amount=11`:**
```
dp[0]  = 0
dp[1]  = min(dp[0]+1) = 1          (use coin 1)
dp[5]  = min(dp[4]+1, dp[0]+1) = 1 (use coin 5)
dp[10] = min(dp[9]+1, dp[5]+1, dp[0]+1) = 1  (use coin 10)
dp[11] = min(dp[10]+1, dp[6]+1, dp[1]+1) = 2 (use coin 1: dp[10]+1=2) ✓
```

---

## Kotlin Solution

### Approach 1 — Bottom-up DP

```kotlin
fun coinChange(coins: IntArray, amount: Int): Int {
    val dp = IntArray(amount + 1) { amount + 1 }  // init to "infinity"
    dp[0] = 0

    for (i in 1..amount) {
        for (coin in coins) {
            if (coin <= i) {
                dp[i] = minOf(dp[i], dp[i - coin] + 1)
            }
        }
    }

    return if (dp[amount] > amount) -1 else dp[amount]
}
```

### Approach 2 — BFS (treat as shortest path)

```kotlin
fun coinChange(coins: IntArray, amount: Int): Int {
    if (amount == 0) return 0

    val visited = BooleanArray(amount + 1)
    val queue = ArrayDeque<Int>()
    queue.add(0)
    visited[0] = true
    var steps = 0

    while (queue.isNotEmpty()) {
        steps++
        repeat(queue.size) {
            val current = queue.removeFirst()
            for (coin in coins) {
                val next = current + coin
                if (next == amount) return steps
                if (next < amount && !visited[next]) {
                    visited[next] = true
                    queue.add(next)
                }
            }
        }
    }

    return -1
}
```

---

## Why Kotlin Shines Here

**`IntArray(amount + 1) { amount + 1 }`** — initialise all values in one line:
```kotlin
val dp = IntArray(amount + 1) { amount + 1 }
// Lambda initialiser sets every element to amount+1 (acts as infinity)
// vs Java: int[] dp = new int[amount + 1]; Arrays.fill(dp, amount + 1);
```

**`minOf()`** — cleaner than `Math.min`:
```kotlin
dp[i] = minOf(dp[i], dp[i - coin] + 1)
```

**`1..amount`** — inclusive range, reads naturally:
```kotlin
for (i in 1..amount)
// vs Java: for (int i = 1; i <= amount; i++)
```

**`repeat(queue.size)`** — process one BFS level at a time:
```kotlin
repeat(queue.size) { ... }
// Captures queue.size before the loop starts — correct level-by-level BFS
```

---

## Why `amount + 1` as Infinity?

```kotlin
val dp = IntArray(amount + 1) { amount + 1 }
```

The maximum number of coins to make `amount` is `amount` itself (using all 1s).
So `amount + 1` is safely larger than any valid answer — acts as infinity.
At the end, if `dp[amount] > amount` it means no solution was found → return -1.

---

## Two Approaches Compared

| Approach | Time | Space | Notes |
|---|---|---|---|
| Bottom-up DP | O(amount × coins) | O(amount) | Standard, easy to explain |
| BFS | O(amount × coins) | O(amount) | Finds shortest path naturally |

---

## Complexity

| | |
|---|---|
| **Time** | O(amount × number of coins) |
| **Space** | O(amount) |

---

## Key Takeaway

> Build up the answer for every amount from 0 to target.
> For each amount, try every coin — if it fits, can it give you fewer coins?
> Use `amount + 1` as a safe "infinity" initialiser.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/coin-change/)

---

{% include kotlin-dsa-nav.html %}
