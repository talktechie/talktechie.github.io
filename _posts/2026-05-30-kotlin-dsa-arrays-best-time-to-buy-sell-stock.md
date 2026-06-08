---
title: "Arrays: Best Time to Buy and Sell Stock — Kotlin Solution"
date: 2026-05-30
categories: [Kotlin DSA, Arrays]
tags: [best-time-to-buy-sell-stock, sliding-window, easy, leetcode, leetcode-121, arrays, greedy]
leetcode_number: 121
leetcode_url: https://leetcode.com/problems/best-time-to-buy-and-sell-stock/
difficulty: Easy
permalink: /posts/kotlin-dsa-arrays-best-time-to-buy-sell-stock/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [121 — Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) |
| **Difficulty** | Easy |
| **Topic** | Arrays, Greedy, Sliding Window |

> Given an array `prices` where `prices[i]` is the price on day `i`,
> return the maximum profit from a single buy-sell transaction.

**Example:**
```
Input:  prices = [7, 1, 5, 3, 6, 4]
Output: 5  // Buy at 1, sell at 6
```

---

## Approach

We can't sell before we buy — so we track the **minimum price seen so far**
and at each step compute potential profit.

No nested loops needed. Single pass.

---

## Kotlin Solution

```kotlin
fun maxProfit(prices: IntArray): Int {
    var minPrice = Int.MAX_VALUE
    var maxProfit = 0

    for (price in prices) {
        if (price < minPrice) {
            minPrice = price
        } else {
            maxProfit = maxOf(maxProfit, price - minPrice)
        }
    }

    return maxProfit
}
```

---

## Why Kotlin Shines Here

**`maxOf()`** — cleaner than `Math.max()`:
```kotlin
maxProfit = maxOf(maxProfit, price - minPrice)
// vs Java: maxProfit = Math.max(maxProfit, price - minPrice);
```

**`for (price in prices)`** — no index needed when you don't need it:
```kotlin
for (price in prices)
// vs Java: for (int price : prices)
```

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — single pass through prices |
| **Space** | O(1) — only two variables |

---

## Key Takeaway

> Track the minimum price seen so far. At each step, profit = current price − minimum so far. Keep the max of all profits.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)

---

{% include kotlin-dsa-nav.html %}
