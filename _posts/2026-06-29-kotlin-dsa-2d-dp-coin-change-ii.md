---
title: "2-D DP: Coin Change II — Kotlin Solution"
date: 2026-06-29
categories: [Kotlin DSA, 2-D Dynamic Programming]
tags: [coin-change-ii, dynamic-programming, knapsack, array, medium, leetcode, leetcode-518]
leetcode_number: 518
leetcode_url: https://leetcode.com/problems/coin-change-ii/
difficulty: Medium
permalink: /posts/kotlin-dsa-2d-dp-coin-change-ii/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [518 — Coin Change II](https://leetcode.com/problems/coin-change-ii/) |
| **Difficulty** | Medium |
| **Topic** | Dynamic Programming, Knapsack, Array |

> Given coins of different denominations and an `amount`, return the
> **number of distinct combinations** (order doesn't matter) that make
> up that amount, using unlimited coins of each type.

**Example:**
```
Input:  amount = 5, coins = [1,2,5]
Output: 4
Explanation: 5=5, 5=2+2+1, 5=2+1+1+1, 5=1+1+1+1+1

Input:  amount = 3, coins = [2]
Output: 0
```

**Constraints:**
- `1 <= coins.length <= 300`
- `1 <= coins[i] <= 5000`
- `0 <= amount <= 5000`

---

## Approach

This is the **counting** variant of Coin Change (from the 1-D DP
phase) — instead of minimizing coin count, we count distinct **ways**.
Conceptually it's a 2-D problem: one dimension is the amount, the other
is "which coins have been considered so far" — though we'll see this
collapses elegantly to 1-D space, much like Partition Equal Subset Sum.

**Key insight — process coins one at a time, in the OUTER loop:** unlike
Coin Change (where every coin is tried at every amount in the same
pass), counting **combinations** (not permutations) requires fixing an
order of consideration. Process one coin denomination fully across all
amounts before moving to the next — this guarantees `{1,2}` and `{2,1}`
are never double-counted as different "combinations," since the coin
loop is on the outside.

**Walk through `amount = 5, coins = [1,2,5]`:**
```
dp[0..5] starts as [1,0,0,0,0,0]   (1 way to make amount 0: use no coins)

Process coin=1: for a in 1..5: dp[a] += dp[a-1]
  dp = [1,1,1,1,1,1]   (only ever 1 way using just 1's)

Process coin=2: for a in 2..5: dp[a] += dp[a-2]
  dp[2] += dp[0]=1 → dp[2]=2
  dp[3] += dp[1]=1 → dp[3]=2
  dp[4] += dp[2]=2 → dp[4]=3
  dp[5] += dp[3]=2 → dp[5]=3
  dp = [1,1,2,2,3,3]

Process coin=5: for a in 5..5: dp[a] += dp[a-5]
  dp[5] += dp[0]=1 → dp[5]=4
  dp = [1,1,2,2,3,4]

Answer: dp[5] = 4 ✓
```

---

## Kotlin Solution

### Approach 1 — 1-D DP, coin loop OUTSIDE amount loop (optimal)

```kotlin
fun change(amount: Int, coins: IntArray): Int {
    val dp = IntArray(amount + 1)
    dp[0] = 1   // exactly one way to make amount 0: use nothing

    for (coin in coins) {
        for (a in coin..amount) {
            dp[a] += dp[a - coin]
        }
    }

    return dp[amount]
}
```

### Approach 2 — Explicit 2-D table (clearer to see why ordering matters)

```kotlin
fun change(amount: Int, coins: IntArray): Int {
    val n = coins.size
    val dp = Array(n + 1) { IntArray(amount + 1) }
    dp[0][0] = 1   // using zero coin types, amount 0 — exactly 1 way

    for (i in 1..n) {
        for (a in 0..amount) {
            dp[i][a] = dp[i - 1][a]   // don't use coins[i-1] at all
            if (a >= coins[i - 1]) {
                dp[i][a] += dp[i][a - coins[i - 1]]   // use at least one of coins[i-1]
            }
        }
    }

    return dp[n][amount]
}
```

Makes the "which coins have been considered" dimension explicit —
`dp[i][a]` represents the number of ways to make amount `a` using only
the **first `i`** coin denominations. The 1-D version (Approach 1) is
this same table with the `i` dimension compressed away, since each row
only ever depends on the row above and itself.

---

## Why the Coin Loop Must Be OUTSIDE the Amount Loop (Unlike Coin Change)

This is the single most important detail distinguishing this problem
from Coin Change (LC 322), and it's worth understanding precisely why
the loop order flips:

```kotlin
for (coin in coins) {        // OUTER — process one denomination fully
    for (a in coin..amount) {  // INNER — across all amounts
        dp[a] += dp[a - coin]
    }
}
```

If the loops were swapped (amount outside, coins inside — like Coin
Change's structure), we'd be counting **ordered sequences** of coins,
not unordered combinations — `{1,2}` and `{2,1}` would both get counted
as distinct ways to make `3`, when the problem wants them treated as
the **same** combination. By fully processing one coin denomination
before introducing the next, we guarantee combinations are built up in a
canonical, order-independent way — once coin `2` enters the count, coin
`1` "combinations" already baked into `dp[]` can be extended by `2`s,
but we never re-introduce `1` after `2` in a way that double-counts the
same multiset of coins arranged differently.

**Why `dp[0] = 1` is the correct base case:** there's exactly one way to
make amount `0` — use no coins at all. This seed value is what allows
every other `dp[a] += dp[a - coin]` update to correctly "extend" a known
combination by one more coin of the current denomination.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| 1-D array, coin loop outside (Approach 1) | Always — O(amount) space, the standard optimized solution |
| Explicit 2-D table (Approach 2) | Want to see precisely why the loop ordering matters, before compressing to 1-D |

---

## Complexity

| | |
|---|---|
| **Time** | O(amount × coins.length) |
| **Space** | O(amount) — 1-D version |

---

## Key Takeaway

> Coin Change II reveals a subtle but critical DP lesson: when counting
> **combinations** (unordered) rather than **sequences** (ordered), the
> order in which you iterate your "items" dimension relative to your
> "capacity" dimension determines what you're actually counting. Coin
> loop outside, amount loop inside → combinations (this problem). Amount
> loop outside, coin loop inside → would count ordered sequences instead.
> This loop-ordering sensitivity is one of the most important — and most
> easily overlooked — details across all knapsack-style DP problems.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/coin-change-ii/)

---

{% include kotlin-dsa-nav.html %}
