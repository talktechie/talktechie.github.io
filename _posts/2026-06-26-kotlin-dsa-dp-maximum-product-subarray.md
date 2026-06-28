---
title: "1-D DP: Maximum Product Subarray — Kotlin Solution"
date: 2026-06-26
categories: [Kotlin DSA, 1-D Dynamic Programming]
tags: [maximum-product-subarray, dynamic-programming, array, medium, leetcode, leetcode-152]
leetcode_number: 152
leetcode_url: https://leetcode.com/problems/maximum-product-subarray/
difficulty: Medium
permalink: /posts/kotlin-dsa-dp-maximum-product-subarray/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [152 — Maximum Product Subarray](https://leetcode.com/problems/maximum-product-subarray/) |
| **Difficulty** | Medium |
| **Topic** | Dynamic Programming, Array |

> Given an integer array `nums`, find a **contiguous subarray** with the
> largest product, and return that product.

**Example:**
```
Input:  nums = [2,3,-2,4]
Output: 6
Explanation: subarray [2,3] has product 6

Input:  nums = [-2,0,-1]
Output: 0
Explanation: the product can't be positive (mixing the zero and -1
             always yields 0 or requires excluding the zero, but then
             only -2 or -1 alone, both negative) — best achievable is 0
```

**Constraints:**
- `1 <= nums.length <= 2 * 10⁴`
- `-10 <= nums[i] <= 10`

---

## Approach

This looks like Kadane's algorithm (max subarray **sum**), but
**multiplication's sign-flipping behavior** breaks the naive
greedy-max approach: a very **negative** running product can suddenly
become the new **maximum** if multiplied by another negative number.

**Key insight — track BOTH the running maximum AND running minimum
product ending at each position:** at every step, the new maximum could
come from extending the previous maximum (if the current number is
positive) **or** from extending the previous minimum (if the current
number is negative, flipping a very negative product into a very
positive one). Track both possibilities and let them inform each other.

**Walk through `nums = [2,3,-2,4]`:**
```
i=0 (2): maxProd=2, minProd=2                         (initialize with first element)
i=1 (3): candidates: 3*maxProd=6, 3*minProd=6, 3 alone=3
         maxProd=max(6,6,3)=6   minProd=min(6,6,3)=3
i=2 (-2): candidates: -2*maxProd=-12, -2*minProd=-6, -2 alone=-2
          maxProd=max(-12,-6,-2)=-2   minProd=min(-12,-6,-2)=-12
          (negative number FLIPS which extreme becomes which —
           the old minProd, when multiplied by a negative, became
           a candidate for the new... let's recheck: -2*minProd(3)=-6,
           that's a candidate for maxProd; -2*maxProd(6)=-12, candidate
           for minProd)
i=3 (4): candidates: 4*maxProd(-2)=-8, 4*minProd(-12)=-48, 4 alone=4
         maxProd=max(-8,-48,4)=4   minProd=min(-8,-48,4)=-48

Running best answer tracked across all steps: max(2,6,-2,4) = 6 ✓
```

---

## Kotlin Solution

### Approach 1 — Track running max AND min product, O(1) space (optimal)

```kotlin
fun maxProduct(nums: IntArray): Int {
    var maxProd = nums[0]
    var minProd = nums[0]
    var result = nums[0]

    for (i in 1 until nums.size) {
        val num = nums[i]

        if (num < 0) {
            // negative number flips which running extreme is which
            val temp = maxProd
            maxProd = maxOf(num, minProd * num)
            minProd = minOf(num, temp * num)
        } else {
            maxProd = maxOf(num, maxProd * num)
            minProd = minOf(num, minProd * num)
        }

        result = maxOf(result, maxProd)
    }

    return result
}
```

### Approach 2 — Same idea, without the explicit negative-number branch (slightly more concise)

```kotlin
fun maxProduct(nums: IntArray): Int {
    var maxProd = nums[0]
    var minProd = nums[0]
    var result = nums[0]

    for (i in 1 until nums.size) {
        val num = nums[i]
        val candidates = listOf(num, maxProd * num, minProd * num)

        maxProd = candidates.max()
        minProd = candidates.min()
        result = maxOf(result, maxProd)
    }

    return result
}
```

Computes all three candidates (the number alone, extending the running
max, extending the running min) and simply takes the max/min across
them — avoids needing a separate branch for negative numbers, at a
small cost of extra list allocation per iteration.

---

## Why We Must Track BOTH a Running Max AND a Running Min

This is the single most important insight, and it's worth internalizing
deeply since it generalizes to other "running extremes under
sign-sensitive combination" problems:

```kotlin
if (num < 0) {
    val temp = maxProd
    maxProd = maxOf(num, minProd * num)   // multiplying a NEGATIVE by the
                                            // previous MINIMUM (most negative)
                                            // can produce a large POSITIVE result
    minProd = minOf(num, temp * num)       // multiplying a NEGATIVE by the
                                            // previous MAXIMUM (most positive)
                                            // produces a very NEGATIVE result
}
```

If we only tracked a running maximum (as Kadane's algorithm does for
sums), we'd lose track of how negative the running product could get —
and that "very negative" value might be exactly what's needed to produce
the **next** maximum once multiplied by another negative number. Tracking
both extremes ensures neither possibility is ever discarded prematurely.

**Why `result` is updated every iteration, not just once at the end:**
the overall maximum product subarray might end at any position, not
necessarily the last one — continuously comparing `maxProd` against the
running `result` ensures we never miss an earlier peak.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Explicit branch on sign (Approach 1) | Slightly more efficient, no extra allocations |
| Candidates list (Approach 2) | More concise, easier to verify correctness by inspection |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — single pass |
| **Space** | O(1) |

---

## Key Takeaway

> Whenever a DP problem involves multiplication (rather than addition)
> across a sequence, sign flips mean a single running "best" value is
> insufficient — track both a running **maximum** and a running
> **minimum** simultaneously, since either can become the source of the
> next maximum depending on the sign of the next element. This dual-
> tracking technique is the key generalization that separates Maximum
> Product Subarray from simpler "running best" DP problems like House
> Robber or Climbing Stairs.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/maximum-product-subarray/)

---

{% include kotlin-dsa-nav.html %}
