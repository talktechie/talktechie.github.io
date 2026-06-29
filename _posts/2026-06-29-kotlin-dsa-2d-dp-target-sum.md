---
title: "2-D DP: Target Sum — Kotlin Solution"
date: 2026-06-29
categories: [Kotlin DSA, 2-D Dynamic Programming]
tags: [target-sum, dynamic-programming, knapsack, array, backtracking, medium, leetcode, leetcode-494]
leetcode_number: 494
leetcode_url: https://leetcode.com/problems/target-sum/
difficulty: Medium
permalink: /posts/kotlin-dsa-2d-dp-target-sum/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [494 — Target Sum](https://leetcode.com/problems/target-sum/) |
| **Difficulty** | Medium |
| **Topic** | Dynamic Programming, Knapsack, Array, Backtracking |

> Given integers `nums` and a `target`, assign a `+` or `-` sign to each
> number such that the sum equals `target`. Return the number of ways
> to do so.

**Example:**
```
Input:  nums = [1,1,1,1,1], target = 3
Output: 5
Explanation: 5 different ways to choose which two get '-' (since
             5×1 - 2×(sum of negated) = 3 means exactly 2 must be negative)

Input:  nums = [1], target = 1
Output: 1
```

**Constraints:**
- `1 <= nums.length <= 20`
- `0 <= nums[i] <= 1000`
- `0 <= sum(nums) <= 1000`
- `-1000 <= target <= 1000`

---

## Approach

The naive backtracking — try `+` and `-` for every number — is O(2ⁿ),
correct but exponential. The key insight transforms this into a
**subset-sum** counting problem, directly reusable from Partition Equal
Subset Sum's technique.

**Key insight — algebraic transformation:** split `nums` into a
"positive" subset `P` and "negative" subset `N`. We need
`sum(P) - sum(N) = target`. Since `sum(P) + sum(N) = totalSum` (every
number is in exactly one subset), substituting gives:
`sum(P) = (totalSum + target) / 2`. This converts the problem into
"count the number of subsets of `nums` that sum to exactly
`(totalSum + target) / 2`" — pure subset-sum counting, just like
Partition Equal Subset Sum, but **counting** ways instead of returning
a boolean.

**Walk through `nums = [1,1,1,1,1], target = 3`:**
```
totalSum = 5
required subset sum = (5 + 3) / 2 = 4

Count subsets of [1,1,1,1,1] summing to exactly 4:
  choosing any 4 of the 5 ones sums to 4 → C(5,4) = 5 ways

Answer: 5 ✓ (matches expected output)
```

---

## Kotlin Solution

### Approach 1 — Transform to subset-sum counting, 1-D DP (optimal)

```kotlin
fun findTargetSumWays(nums: IntArray, target: Int): Int {
    val totalSum = nums.sum()

    // If (totalSum + target) is odd or target is out of achievable range, no valid split exists
    if ((totalSum + target) % 2 != 0 || Math.abs(target) > totalSum) return 0

    val subsetTarget = (totalSum + target) / 2
    val dp = IntArray(subsetTarget + 1)
    dp[0] = 1   // exactly one way to make sum 0: pick nothing

    for (num in nums) {
        for (s in subsetTarget downTo num) {
            dp[s] += dp[s - num]
        }
    }

    return dp[subsetTarget]
}
```

### Approach 2 — Top-down memoization on (index, currentSum)

```kotlin
fun findTargetSumWays(nums: IntArray, target: Int): Int {
    val memo = HashMap<Pair<Int, Int>, Int>()

    fun dp(index: Int, currentSum: Int): Int {
        if (index == nums.size) return if (currentSum == target) 1 else 0

        val key = index to currentSum
        return memo.getOrPut(key) {
            dp(index + 1, currentSum + nums[index]) + dp(index + 1, currentSum - nums[index])
        }
    }

    return dp(0, 0)
}
```

Directly mirrors the backtracking brute force, but memoizes on
`(index, currentSum)` pairs to avoid recomputing identical subproblems
— a more literal translation of "try plus and minus at every step,"
without needing the algebraic subset-sum transformation.

---

## Why the Algebraic Transformation Is the Key Unlock

This is worth deriving carefully, since it's not an obvious leap:

```
Let P = sum of numbers assigned '+', N = sum of numbers assigned '-'
We need: P - N = target
We also know: P + N = totalSum (every number gets exactly one sign)

Adding these two equations: 2P = totalSum + target
→ P = (totalSum + target) / 2
```

This converts "assign signs to satisfy an equation" into "find subsets
summing to a specific value" — a problem we already know how to count
efficiently via 0/1 knapsack DP (the same downward-iteration technique
from Partition Equal Subset Sum).

**Why we check `(totalSum + target) % 2 != 0` as an immediate
disqualifier:** since `P` must be a whole number of actual elements'
sum, `(totalSum + target)` must be **even** — if it's odd, no valid
subset split could possibly satisfy the equation, and we can return `0`
immediately without any DP computation.

**Why iterating `s` downward (`subsetTarget downTo num`)** is the same
0/1 knapsack trick as before: it ensures each number from `nums` is only
counted **once** per subset, preventing the same element from being
"reused" within a single combination during the same pass.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Subset-sum transformation (Approach 1) | Always — the elegant, efficient solution once the algebra is understood |
| Memoized backtracking (Approach 2) | More intuitive direct translation of the problem statement, useful as a stepping stone |

---

## Complexity

| | Subset-Sum DP | Memoized Backtracking |
|---|---|---|
| **Time** | O(n × totalSum) | O(n × range of possible sums) |
| **Space** | O(totalSum) | O(n × range of possible sums) |

---

## Key Takeaway

> Target Sum looks like a sign-assignment combinatorics problem, but a
> clean algebraic substitution (`P - N = target`, `P + N = totalSum`)
> reveals it's secretly Partition Equal Subset Sum's counting cousin —
> count subsets summing to `(totalSum + target) / 2`. Recognizing when a
> problem can be **transformed** into an already-solved DP shape, rather
> than solved from scratch, is one of the highest-leverage skills in
> this entire phase — the underlying 0/1 knapsack counting technique
> doesn't need to be reinvented, just correctly mapped onto the new
> problem's structure.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/target-sum/)

---

{% include kotlin-dsa-nav.html %}
