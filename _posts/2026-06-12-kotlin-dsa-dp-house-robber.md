---
title: "Dynamic Programming: House Robber — Kotlin Solution"
date: 2026-06-12
categories: [Kotlin DSA, Dynamic Programming]
tags: [house-robber, dynamic-programming, medium, leetcode, leetcode-198]
leetcode_number: 198
leetcode_url: https://leetcode.com/problems/house-robber/
difficulty: Medium
permalink: /posts/kotlin-dsa-dp-house-robber/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [198 — House Robber](https://leetcode.com/problems/house-robber/) |
| **Difficulty** | Medium |
| **Topic** | Dynamic Programming |

> You are a robber planning to rob houses along a street.
> Each house has some money. You cannot rob two adjacent houses
> (it triggers an alarm). Return the maximum amount you can rob.

**Example:**
```
Input:  nums = [2, 7, 9, 3, 1]
Output: 12
// Rob house 1 (2) + house 3 (9) + house 5 (1) = 12
```

---

## Approach

At each house you decide: **rob it or skip it**.

- **Rob it** → you get `nums[i]` + best from two houses back (`dp[i-2]`)
- **Skip it** → you get best from previous house (`dp[i-1]`)

Pick whichever is larger:
> `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`

**Walk through `[2, 7, 9, 3, 1]`:**
```
i=0: dp[0] = 2
i=1: dp[1] = max(2, 7) = 7
i=2: dp[2] = max(7, 2+9) = 11
i=3: dp[3] = max(11, 7+3) = 11
i=4: dp[4] = max(11, 11+1) = 12 ✓
```

---

## Kotlin Solution

### Approach 1 — DP Array

```kotlin
fun rob(nums: IntArray): Int {
    if (nums.size == 1) return nums[0]

    val dp = IntArray(nums.size)
    dp[0] = nums[0]
    dp[1] = maxOf(nums[0], nums[1])

    for (i in 2 until nums.size) {
        dp[i] = maxOf(dp[i - 1], dp[i - 2] + nums[i])
    }

    return dp[nums.size - 1]
}
```

### Approach 2 — O(1) Space (two variables)

```kotlin
fun rob(nums: IntArray): Int {
    var prev2 = 0
    var prev1 = 0

    for (num in nums) {
        val current = maxOf(prev1, prev2 + num)
        prev2 = prev1
        prev1 = current
    }

    return prev1
}
```

---

## Why Kotlin Shines Here

**`for (num in nums)`** — no index needed when you only need the value:
```kotlin
for (num in nums) {
    val current = maxOf(prev1, prev2 + num)
// vs Java: for (int num : nums) — same length but Kotlin feels more natural
```

**`maxOf()`** — expresses the DP recurrence clearly:
```kotlin
dp[i] = maxOf(dp[i - 1], dp[i - 2] + nums[i])
// "Skip or rob" expressed in one line
```

**`IntArray(nums.size)`** — zero-initialised, no boxing:
```kotlin
val dp = IntArray(nums.size)
```

---

## Two Approaches Compared

| Approach | Time | Space | Notes |
|---|---|---|---|
| DP array | O(n) | O(n) | Easy to trace and debug |
| Two variables | O(n) | O(1) | Optimal — interview preferred |

The two-variable approach works because you only ever need the last two values.

---

## Complexity

| | |
|---|---|
| **Time** | O(n) |
| **Space** | O(1) with two-variable approach |

---

## Key Takeaway

> At every house: take it (plus best two back) or skip it (carry best so far).
> You only need the last two values — no array needed.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/house-robber/)

---

{% include kotlin-dsa-nav.html %}
