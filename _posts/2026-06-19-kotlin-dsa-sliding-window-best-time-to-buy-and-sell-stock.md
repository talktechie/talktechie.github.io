---
title: "Sliding Window: Best Time to Buy And Sell Stock — Kotlin Solution"
date: 2026-06-19
categories: [Kotlin DSA, Sliding Window]
tags: [best-time-buy-sell-stock, sliding-window, two-pointers, array, easy, leetcode, leetcode-121]
leetcode_number: 121
leetcode_url: https://leetcode.com/problems/best-time-to-buy-and-sell-stock/
difficulty: Easy
permalink: /posts/kotlin-dsa-sliding-window-best-time-to-buy-and-sell-stock/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [121 — Best Time to Buy And Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) |
| **Difficulty** | Easy |
| **Topic** | Sliding Window, Two Pointers, Array |

> You are given an array `prices` where `prices[i]` is the price of a
> stock on day `i`. You want to maximize profit by choosing a single day
> to buy and a different day **in the future** to sell.
>
> Return the maximum profit achievable. If no profit is possible, return `0`.

**Example:**
```
Input:  prices = [7,1,5,3,6,4]
Output: 5
Explanation: Buy on day 2 (price=1), sell on day 5 (price=6) → profit = 5

Input:  prices = [7,6,4,3,1]
Output: 0
Explanation: Prices keep falling — no profitable transaction exists
```

**Constraints:**
- `1 <= prices.length <= 10⁵`
- `0 <= prices[i] <= 10⁴`

---

## Approach

This is the entry point for the Sliding Window phase — and a great way to
see that "window" doesn't always mean a literal substring or subarray of
fixed shape. Here, the window is simply **[buy day, sell day]**, and it
only ever needs to expand to the right.

**Key insight:** Track the **lowest price seen so far** as you scan left to
right. At every day, ask: "if I sold today, having bought at the lowest
price so far, what would my profit be?" Keep the best answer seen.

This works because the buy day must always come **before** the sell day —
we never need to look backward once we've recorded the minimum.

**Walk through `[7,1,5,3,6,4]`:**
```
day0 (7) → minPrice=7  → profit if sold today = 7-7=0  → maxProfit=0
day1 (1) → minPrice=1  → profit if sold today = 1-1=0  → maxProfit=0
day2 (5) → minPrice=1  → profit if sold today = 5-1=4  → maxProfit=4
day3 (3) → minPrice=1  → profit if sold today = 3-1=2  → maxProfit=4 (no improvement)
day4 (6) → minPrice=1  → profit if sold today = 6-1=5  → maxProfit=5
day5 (4) → minPrice=1  → profit if sold today = 4-1=3  → maxProfit=5 (no improvement)

Answer: 5 ✓
```

---

## Kotlin Solution

### Approach 1 — Single pass, track running minimum (optimal)

```kotlin
fun maxProfit(prices: IntArray): Int {
    var minPrice = Int.MAX_VALUE
    var maxProfit = 0

    for (price in prices) {
        minPrice = minOf(minPrice, price)
        maxProfit = maxOf(maxProfit, price - minPrice)
    }

    return maxProfit
}
```

### Approach 2 — Two pointers framing (same logic, window vocabulary)

```kotlin
fun maxProfit(prices: IntArray): Int {
    var left = 0   // represents the "buy" day
    var right = 1  // represents the "sell" day
    var maxProfit = 0

    while (right < prices.size) {
        if (prices[left] < prices[right]) {
            // Profitable — record it, then slide the window forward
            maxProfit = maxOf(maxProfit, prices[right] - prices[left])
        } else {
            // Found a new lower buy point — move left up to right
            left = right
        }
        right++
    }

    return maxProfit
}
```

Both approaches are O(n) — Approach 2 makes the two-pointer/window framing
explicit, which is useful for connecting this problem conceptually to the
rest of the Sliding Window phase.

---

## Why a Single Pass Is Enough

The brute force checks every `(buy, sell)` pair — O(n²). But notice: for any
fixed sell day, the best possible buy day is always the **minimum price seen
before it**. There's never a reason to consider any other earlier price.

```kotlin
minPrice = minOf(minPrice, price)
maxProfit = maxOf(maxProfit, price - minPrice)
```

Both updates happen **every iteration**, in the right order: update the
minimum first (today could be the new best buy day), then check the
profit (today could be the best sell day given everything seen so far,
including the just-updated minimum).

**Why `left = right` instead of `left++`** in Approach 2 — when the current
price is lower than what we're holding, there's no reason to keep the old
buy day around; jump straight to the new lowest point:
```kotlin
} else {
    left = right   // not left++, directly adopt the new lower price as buy day
}
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Running minimum (Approach 1) | Cleanest, most direct — the standard answer |
| Two pointers (Approach 2) | Want to frame it explicitly as a window, for teaching/comparison |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — single pass |
| **Space** | O(1) |

---

## Key Takeaway

> Track the minimum price seen so far; at every step, the best possible
> profit is "today's price minus the lowest price before today." No need
> to check every pair — the running minimum already encodes the best
> possible buy day for any sell day. This single-pass, expand-only window
> is the simplest member of the Sliding Window family, and a natural
> warm-up before the substring-based problems later in this phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)

---

{% include kotlin-dsa-nav.html %}
