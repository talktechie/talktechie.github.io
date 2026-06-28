---
title: "Greedy: Maximum Subarray — Kotlin Solution"
date: 2026-06-28
categories: [Kotlin DSA, Greedy]
tags: [maximum-subarray, greedy, dynamic-programming, array, kadane, medium, leetcode, leetcode-53]
leetcode_number: 53
leetcode_url: https://leetcode.com/problems/maximum-subarray/
difficulty: Medium
permalink: /posts/kotlin-dsa-greedy-maximum-subarray/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [53 — Maximum Subarray](https://leetcode.com/problems/maximum-subarray/) |
| **Difficulty** | Medium |
| **Topic** | Greedy, Dynamic Programming, Array, Kadane's Algorithm |

> Given an integer array `nums`, find the **contiguous subarray** with
> the largest sum, and return that sum.

**Example:**
```
Input:  nums = [-2,1,-3,4,-1,2,1,-5,4]
Output: 6
Explanation: subarray [4,-1,2,1] has the largest sum, 6

Input:  nums = [1]
Output: 1

Input:  nums = [5,4,-1,7,8]
Output: 23
```

**Constraints:**
- `1 <= nums.length <= 10⁵`
- `-10⁴ <= nums[i] <= 10⁴`

---

## Approach

This is the entry point to the Greedy phase, and it's the classic
**Kadane's Algorithm** — a beautiful example of a greedy decision made
locally at every step that provably produces the global optimum.

**Key insight:** Track a running sum as you scan left to right. The
moment that running sum drops **below zero**, it can only ever hurt any
future subarray that includes it — so greedily **reset** it to zero
(effectively, start fresh from the next element) rather than carrying
forward a negative burden. At every position, compare the running sum
against the best seen so far.

**Walk through `nums = [-2,1,-3,4,-1,2,1,-5,4]`:**
```
running=0, best=-∞

-2: running = max(0,-∞)... let's just track running = running+num, reset if negative
running = 0 + (-2) = -2 → best=-2 → since running<0, reset running=0
1:  running = 0+1 = 1 → best=max(-2,1)=1
-3: running = 1+(-3) = -2 → best stays 1 → reset running=0 (negative)
4:  running = 0+4 = 4 → best=max(1,4)=4
-1: running = 4+(-1) = 3 → best stays 4
2:  running = 3+2 = 5 → best=5
1:  running = 5+1 = 6 → best=6
-5: running = 6+(-5) = 1 → best stays 6
4:  running = 1+4 = 5 → best stays 6

Answer: 6 ✓
```

---

## Kotlin Solution

### Approach 1 — Kadane's Algorithm, greedy reset on negative running sum (optimal)

```kotlin
fun maxSubArray(nums: IntArray): Int {
    var maxSum = nums[0]
    var currentSum = 0

    for (num in nums) {
        currentSum += num
        maxSum = maxOf(maxSum, currentSum)

        if (currentSum < 0) {
            currentSum = 0   // greedy reset — a negative running sum can never help
        }
    }

    return maxSum
}
```

### Approach 2 — Equivalent DP framing: dp[i] = best subarray ending exactly at i

```kotlin
fun maxSubArray(nums: IntArray): Int {
    var dpPrev = nums[0]
    var maxSum = nums[0]

    for (i in 1 until nums.size) {
        val dpCurrent = maxOf(nums[i], dpPrev + nums[i])
        maxSum = maxOf(maxSum, dpCurrent)
        dpPrev = dpCurrent
    }

    return maxSum
}
```

Frames the exact same algorithm as a 1-D DP recurrence:
`dp[i] = max(nums[i], dp[i-1] + nums[i])` — either start a brand new
subarray at `i`, or extend the best subarray ending at `i-1`. This is
mathematically identical to Approach 1's reset logic, just expressed
through DP vocabulary rather than greedy vocabulary — a useful reminder
that Kadane's Algorithm sits at the intersection of both paradigms.

---

## Why Resetting on a Negative Running Sum Is Provably Correct

This is the central greedy insight, and it's worth understanding exactly
why it's safe — not just a heuristic, but a guaranteed-optimal choice:

```kotlin
if (currentSum < 0) {
    currentSum = 0
}
```

If the running sum ever becomes negative, that means everything
accumulated so far is a net negative contribution. **Any** future
subarray that includes this negative prefix would always be **strictly
worse** than the same future subarray **without** it — so there's never
a reason to carry a negative running sum forward. This is what makes the
greedy choice safe: we're not guessing, we're eliminating a definitively
suboptimal option.

**Why `maxSum` is updated every iteration, not just when resetting:** the
best subarray could end at *any* position, not just right before a
reset — continuously comparing `currentSum` against `maxSum` ensures no
potential peak is missed.

**Why Approach 2's DP framing is mathematically the same algorithm:**
`dp[i-1] + nums[i]` failing to beat `nums[i]` alone is exactly equivalent
to "the running sum up through `i-1` was negative" — both express the
same underlying decision, just from different conceptual angles (greedy
"reset when harmful" vs. DP "take the better of two explicit options").

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Greedy reset (Approach 1) | Simpler to write and explain, the standard Kadane's Algorithm form |
| DP recurrence (Approach 2) | Want to see the explicit connection to 1-D DP, useful for recognizing this pattern in DP-framed problems |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — single linear pass |
| **Space** | O(1) |

---

## Key Takeaway

> Kadane's Algorithm is the canonical example of greedy reasoning that's
> also provably a DP recurrence in disguise: a negative running sum can
> never help any future subarray, so resetting it is always safe, never
> just "probably fine." This dual nature — greedy and DP simultaneously
> — is worth remembering as you move through this phase: many greedy
> algorithms have an equivalent DP formulation, and recognizing both
> framings deepens your understanding of *why* the greedy choice is
> actually optimal, not just *that* it happens to work.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/maximum-subarray/)

---

{% include kotlin-dsa-nav.html %}
