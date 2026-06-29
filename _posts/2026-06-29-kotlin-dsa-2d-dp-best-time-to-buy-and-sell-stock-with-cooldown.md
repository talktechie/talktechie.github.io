---
title: "2-D DP: Best Time to Buy And Sell Stock With Cooldown — Kotlin Solution"
date: 2026-06-29
categories: [Kotlin DSA, 2-D Dynamic Programming]
tags: [best-time-buy-sell-stock-cooldown, dynamic-programming, array, state-machine, medium, leetcode, leetcode-309]
leetcode_number: 309
leetcode_url: https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/
difficulty: Medium
permalink: /posts/kotlin-dsa-2d-dp-best-time-to-buy-and-sell-stock-with-cooldown/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [309 — Best Time to Buy And Sell Stock With Cooldown](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/) |
| **Difficulty** | Medium |
| **Topic** | Dynamic Programming, Array, State Machine |

> Given stock `prices`, maximize profit with unlimited transactions, but
> after **selling**, you must wait **one day** (cooldown) before buying
> again.

**Example:**
```
Input:  prices = [1,2,3,0,2]
Output: 3
Explanation: buy(1)→sell(2)=+1, cooldown, buy(0)→sell(2)=+2 → total 3

Input:  prices = [1]
Output: 0
```

**Constraints:**
- `1 <= prices.length <= 5000`
- `0 <= prices[i] <= 1000`

---

## Approach

This is technically a "1-D" array of prices, but the DP **state** at
each day needs **two dimensions** — not just "which day," but "which
of several possible **holding states** am I in" — making this a natural
bridge into 2-D DP's state-machine style of thinking.

**Key insight — model three states per day:** `held` (currently holding
a stock), `sold` (just sold today, entering cooldown), `rest` (not
holding, free to buy — either never bought, or cooldown already
passed). Transitions:
- `held[i] = max(held[i-1], rest[i-1] - prices[i])` — keep holding, or
  buy today from a "free" state.
- `sold[i] = held[i-1] + prices[i]` — sell today (must have been
  holding yesterday).
- `rest[i] = max(rest[i-1], sold[i-1])` — stay free, or finish
  yesterday's cooldown.

**Walk through `prices = [1,2,3,0,2]`:**
```
day0: held=-1, sold=0, rest=0

day1 (price=2): held=max(-1, 0-2)=-1   sold=-1+2=1   rest=max(0,0)=0
day2 (price=3): held=max(-1, 0-3)=-1   sold=-1+3=2   rest=max(0,1)=1
day3 (price=0): held=max(-1, 1-0)=1    sold=-1+0=-1  rest=max(1,2)=2
day4 (price=2): held=max(1, 2-2)=1     sold=1+2=3    rest=max(2,-1)=2

Final answer = max(sold, rest) at the last day = max(3,2) = 3 ✓
```

---

## Kotlin Solution

### Approach 1 — State machine DP, O(1) space rolling variables (optimal)

```kotlin
fun maxProfit(prices: IntArray): Int {
    if (prices.isEmpty()) return 0

    var held = -prices[0]   // bought on day 0
    var sold = 0
    var rest = 0

    for (i in 1 until prices.size) {
        val prevHeld = held
        val prevSold = sold
        val prevRest = rest

        held = maxOf(prevHeld, prevRest - prices[i])
        sold = prevHeld + prices[i]
        rest = maxOf(prevRest, prevSold)
    }

    return maxOf(sold, rest)
}
```

### Approach 2 — Explicit 2-D DP table (clearer state visualization)

```kotlin
fun maxProfit(prices: IntArray): Int {
    val n = prices.size
    if (n == 0) return 0

    // dp[i][0]=held, dp[i][1]=sold, dp[i][2]=rest
    val dp = Array(n) { IntArray(3) }
    dp[0][0] = -prices[0]
    dp[0][1] = 0
    dp[0][2] = 0

    for (i in 1 until n) {
        dp[i][0] = maxOf(dp[i - 1][0], dp[i - 1][2] - prices[i])
        dp[i][1] = dp[i - 1][0] + prices[i]
        dp[i][2] = maxOf(dp[i - 1][2], dp[i - 1][1])
    }

    return maxOf(dp[n - 1][1], dp[n - 1][2])
}
```

---

## Why Three States (Not Two) Are Necessary

A simpler "holding or not holding" model would miss the **cooldown**
constraint entirely. The `sold` state specifically exists to enforce
that you **cannot buy on the very next day** after selling:

```kotlin
held = maxOf(prevHeld, prevRest - prices[i])
// buying today is only allowed from prevREST, NEVER from prevSOLD —
// this is exactly what enforces the one-day cooldown after selling
```

If `held` could transition from `prevSold - prices[i]` too, that would
incorrectly allow buying immediately the day after selling, violating
the cooldown rule. By only allowing the buy transition from `rest` (not
`sold`), the state machine naturally bakes in the required one-day
delay.

**Why the final answer is `max(sold, rest)`, never `held`:** ending the
sequence while still holding a stock means money is "trapped" in an
unsold position — that can never be the optimal final state, since
selling (even at no profit on the very last day, hypothetically) would
never be worse than holding forever.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Rolling variables (Approach 1) | Always — O(1) space, the standard optimized solution |
| Explicit table (Approach 2) | Learning/visualizing the three distinct states clearly |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) |
| **Space** | O(1) rolling / O(n) table |

---

## Key Takeaway

> This problem introduces **state-machine DP**: instead of a single
> value per day, track multiple named states (`held`, `sold`, `rest`)
> and define explicit transition rules between them. The cooldown
> constraint is enforced not by extra bookkeeping, but by **which
> states are allowed to transition into which** — `held` can only come
> from `rest`, never `sold`. This state-machine framing is a powerful
> generalization for "sequential decisions with constraints between
> consecutive actions," bridging naturally into the richer 2-D state
> spaces used throughout the rest of this phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

---

{% include kotlin-dsa-nav.html %}
