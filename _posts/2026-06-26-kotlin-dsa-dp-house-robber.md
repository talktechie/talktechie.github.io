---
title: "1-D DP: House Robber — Kotlin Solution"
date: 2026-06-26
categories: [Kotlin DSA, 1-D Dynamic Programming]
tags: [house-robber, dynamic-programming, array, medium, leetcode, leetcode-198]
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
| **Topic** | Dynamic Programming, Array |

> Given an array `nums` representing money in houses along a street, you
> cannot rob two **adjacent** houses (the alarm would trigger). Return
> the maximum amount you can rob.

**Example:**
```
Input:  nums = [1,2,3,1]
Output: 4
Explanation: rob house 0 (1) and house 2 (3) → 1+3=4

Input:  nums = [2,7,9,3,1]
Output: 12
Explanation: rob houses 0, 2, 4 → 2+9+1=12
```

**Constraints:**
- `1 <= nums.length <= 100`
- `0 <= nums[i] <= 400`

---

## Approach

**Key insight:** At every house, you face a binary choice: **rob it**
(adding its value, but forfeiting the option to also rob the previous
house) or **skip it** (carrying forward whatever was the best total up
to the previous house). This gives the recurrence:
`dp[i] = max(dp[i-1], dp[i-2] + nums[i])` — either skip house `i`
(inheriting `dp[i-1]`'s best), or rob it (adding its value to `dp[i-2]`'s
best, since house `i-1` must then be skipped).

**Walk through `nums = [2,7,9,3,1]`:**
```
dp[0] = 2                                    (only one house, must consider robbing it)
dp[1] = max(nums[0], nums[1]) = max(2,7) = 7  (rob the better of the first two)

dp[2] = max(dp[1], dp[0]+nums[2]) = max(7, 2+9) = max(7,11) = 11
dp[3] = max(dp[2], dp[1]+nums[3]) = max(11, 7+3) = max(11,10) = 11
dp[4] = max(dp[3], dp[2]+nums[4]) = max(11, 11+1) = max(11,12) = 12

Answer: 12 ✓
```

---

## Kotlin Solution

### Approach 1 — Bottom-up DP, O(1) space, two rolling variables (optimal)

```kotlin
fun rob(nums: IntArray): Int {
    var prev2 = 0   // best total considering up through 2 houses ago
    var prev1 = 0   // best total considering up through 1 house ago

    for (num in nums) {
        val current = maxOf(prev1, prev2 + num)
        prev2 = prev1
        prev1 = current
    }

    return prev1
}
```

### Approach 2 — Bottom-up with an explicit DP array (clearer for learning)

```kotlin
fun rob(nums: IntArray): Int {
    val n = nums.size
    if (n == 1) return nums[0]

    val dp = IntArray(n)
    dp[0] = nums[0]
    dp[1] = maxOf(nums[0], nums[1])

    for (i in 2 until n) {
        dp[i] = maxOf(dp[i - 1], dp[i - 2] + nums[i])
    }

    return dp[n - 1]
}
```

### Approach 3 — Top-down memoization

```kotlin
fun rob(nums: IntArray): Int {
    val memo = HashMap<Int, Int>()

    fun dp(i: Int): Int {
        if (i < 0) return 0
        return memo.getOrPut(i) { maxOf(dp(i - 1), dp(i - 2) + nums[i]) }
    }

    return dp(nums.lastIndex)
}
```

---

## Why `maxOf(prev1, prev2 + num)` Captures Both Choices Correctly

This recurrence directly encodes the rob-or-skip decision at every house:

```kotlin
val current = maxOf(prev1, prev2 + num)
// prev1        = best total WITHOUT robbing the current house (skip it,
//                inherit the best result that already accounts for
//                whatever happened up to the previous house)
// prev2 + num  = best total ROBBING the current house (add its value
//                to the best result from TWO houses back, since the
//                immediately preceding house must be skipped if we rob
//                this one)
```

There's no need to track *which* houses were actually robbed — only the
best achievable total at each point, which is sufficient because future
decisions only depend on the two most recent totals, not the full
history of choices.

**Why the rolling-variable version (Approach 1) elegantly handles the
`n=1` edge case with no special-casing:** if there's only one house,
the loop runs once with `prev2=0, prev1=0` initially — `current =
max(0, 0+nums[0]) = nums[0]`, correctly returning just that house's
value. Approach 2's explicit array version needs an `if (n == 1)` guard
specifically because it tries to access `dp[1]` unconditionally, which
would crash on a single-element array.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Rolling variables (Approach 1) | Always — O(1) space, handles edge cases cleanly |
| Explicit DP array (Approach 2) | Learning/explaining — makes the table of subproblems visible |
| Top-down memoization (Approach 3) | Prefer recursive framing |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) |
| **Space** | O(1) rolling variables / O(n) array or memoization |

---

## Key Takeaway

> House Robber introduces the "adjacent constraint" DP shape: at each
> position, decide between including the current element (and being
> forced to skip the previous one) or excluding it (carrying forward the
> best result so far). The recurrence `max(skip, include + value)` is
> one of the most reusable templates in DP — it reappears, lightly
> modified, in House Robber II (handling a circular arrangement) and in
> many "maximum sum with a no-adjacent-elements constraint" variants
> beyond this series.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/house-robber/)

---

{% include kotlin-dsa-nav.html %}
