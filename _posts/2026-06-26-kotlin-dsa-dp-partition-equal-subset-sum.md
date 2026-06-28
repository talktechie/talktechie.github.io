---
title: "1-D DP: Partition Equal Subset Sum — Kotlin Solution"
date: 2026-06-26
categories: [Kotlin DSA, 1-D Dynamic Programming]
tags: [partition-equal-subset-sum, dynamic-programming, knapsack, array, medium, leetcode, leetcode-416]
leetcode_number: 416
leetcode_url: https://leetcode.com/problems/partition-equal-subset-sum/
difficulty: Medium
permalink: /posts/kotlin-dsa-dp-partition-equal-subset-sum/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [416 — Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/) |
| **Difficulty** | Medium |
| **Topic** | Dynamic Programming, Knapsack, Array |

> Given an integer array `nums`, return `true` if it can be partitioned
> into **two subsets** with equal sum.

**Example:**
```
Input:  nums = [1,5,11,5]
Output: true
Explanation: [1,5,5] and [11] both sum to 11

Input:  nums = [1,2,3,5]
Output: false
Explanation: total sum is 11 (odd) — can't split evenly
```

**Constraints:**
- `1 <= nums.length <= 200`
- `1 <= nums[i] <= 100`

---

## Approach

This is the **0/1 Knapsack** problem in disguise — a classic DP shape
distinct from Coin Change's *unbounded* knapsack (where items could be
reused). Here, each element of `nums` can be used **at most once**.

**Key insight:** If the total sum is **odd**, an equal split is
immediately impossible — return `false` right away. Otherwise, the
question becomes: "does some subset of `nums` sum to exactly
**half** the total?" (if such a subset exists, the remaining elements
automatically form the other equal half). This reduces the problem to a
classic **subset-sum** question, solvable with a boolean DP array
representing "is this target sum achievable using some subset of
elements processed so far?"

**Walk through `nums = [1,5,11,5]`, target = sum/2 = 22/2 = 11:**
```
dp = {0: true}  (sum 0 is always achievable — the empty subset)

process 1: for each achievable sum s, sum+1 becomes achievable
  dp now includes: {0, 1}

process 5: for each achievable sum, sum+5 becomes achievable
  dp now includes: {0, 1, 5, 6}

process 11: for each achievable sum, sum+11 becomes achievable
  dp now includes: {0, 1, 5, 6, 11, 12, 16, 17}
  → 11 is now achievable! (this alone already answers the question,
    but let's confirm the algorithm completes correctly)

process 5 (the second one): for each achievable sum, sum+5 becomes achievable
  dp now includes the previous set plus: 5,6,10,11,16,17,21,22

dp[11] = true → a subset summing to 11 exists → answer: true ✓
```

---

## Kotlin Solution

### Approach 1 — Boolean DP array, process each number once, iterate target DOWNWARD (optimal)

```kotlin
fun canPartition(nums: IntArray): Boolean {
    val totalSum = nums.sum()
    if (totalSum % 2 != 0) return false   // odd total — can't split evenly

    val target = totalSum / 2
    val dp = BooleanArray(target + 1)
    dp[0] = true   // sum of 0 is always achievable (the empty subset)

    for (num in nums) {
        for (s in target downTo num) {
            if (dp[s - num]) dp[s] = true
        }
    }

    return dp[target]
}
```

### Approach 2 — Using a HashSet of achievable sums (more intuitive, same complexity)

```kotlin
fun canPartition(nums: IntArray): Boolean {
    val totalSum = nums.sum()
    if (totalSum % 2 != 0) return false

    val target = totalSum / 2
    var achievable = mutableSetOf(0)

    for (num in nums) {
        val newSums = mutableSetOf<Int>()
        for (s in achievable) {
            val candidate = s + num
            if (candidate == target) return true
            if (candidate < target) newSums.add(candidate)
        }
        achievable.addAll(newSums)
    }

    return achievable.contains(target)
}
```

---

## Why the Inner Loop MUST Go Downward (`target downTo num`), Not Upward

This is the single most important — and most commonly gotten wrong —
detail in 0/1 Knapsack-style DP, and it's worth understanding precisely
why:

```kotlin
for (s in target downTo num) {
    if (dp[s - num]) dp[s] = true
}
```

If the inner loop went **upward** instead (`for (s in num..target)`),
processing `num` could update `dp[s]` for some small `s`, and then —
**within the same outer iteration, for the same `num`** — that just-updated
`dp[s]` could incorrectly contribute to updating `dp[s + num]` later in
the same pass. That would effectively allow using the **same** element
**twice**, turning this into an *unbounded* knapsack (like Coin Change)
instead of the *0/1* knapsack this problem actually requires.

Going **downward** ensures that when we compute `dp[s] = dp[s] ||
dp[s-num]`, the `dp[s-num]` value being read is still from the
**previous** outer iteration (before this `num` was considered) — exactly
enforcing "use this element at most once."

**Why checking `totalSum % 2 != 0` first avoids unnecessary work:** if
the total is odd, no two integer subsets can ever sum to exactly half of
it — this O(n) check eliminates an entire class of definitely-impossible
inputs before any DP computation begins.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Boolean DP array (Approach 1) | Always — most space-efficient, the standard 0/1 knapsack template |
| HashSet of achievable sums (Approach 2) | More intuitive to read/explain, slightly more overhead from set operations |

---

## Complexity

| | |
|---|---|
| **Time** | O(n × target) — n = nums.length, target = totalSum / 2 |
| **Space** | O(target) |

---

## Key Takeaway

> Partition Equal Subset Sum reduces to a classic subset-sum question:
> can some subset hit exactly half the total sum? The implementation
> detail that separates correct 0/1 knapsack DP from incorrect unbounded
> reuse is the **direction** of the inner loop — iterating the target
> sum **downward** ensures each element is only ever counted once per
> outer iteration. This downward-iteration trick is the standard
> signature of 0/1 knapsack problems, distinguishing them from Coin
> Change's unbounded variant where the same "item" (coin) could
> legitimately be reused many times.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/partition-equal-subset-sum/)

---

{% include kotlin-dsa-nav.html %}
