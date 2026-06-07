---
title: "Arrays: Maximum Subarray — Kotlin Solution"
date: 2026-06-07
categories: [Kotlin DSA, Arrays]
tags: [maximum-subarray, kadane, dynamic-programming, medium, leetcode, leetcode-53, arrays]
leetcode_number: 53
leetcode_url: https://leetcode.com/problems/maximum-subarray/
difficulty: Medium
permalink: /posts/kotlin-dsa-arrays-maximum-subarray/
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [53 — Maximum Subarray](https://leetcode.com/problems/maximum-subarray/) |
| **Difficulty** | Medium |
| **Topic** | Arrays, Dynamic Programming, Kadane's Algorithm |

> Given an integer array `nums`, find the subarray with the largest sum
> and return its sum.

**Example:**
```
Input:  nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
Output: 6
// [4, -1, 2, 1] has the largest sum = 6
```

---

## Approach

The brute force checks every subarray — O(n²). We can do it in O(n) with
**Kadane's Algorithm**.

**The core idea:**

At each index, you have two choices:
1. Extend the current subarray by including `nums[i]`
2. Start a fresh subarray from `nums[i]`

Which do you pick? Whichever is larger.

If the current running sum becomes negative — it's a liability.
Drop it and start fresh.

**Walk through the example:**
```
nums        = [-2,  1, -3,  4, -1,  2,  1, -5,  4]
currentSum  = [-2,  1, -2,  4,  3,  5,  6,  1,  5]
maxSum      = [-2,  1,  1,  4,  4,  5,  6,  6,  6]
```

At index 0: `currentSum = -2`, negative — next element starts fresh
At index 3: `currentSum = max(4, -2 + 4) = 4` — fresh start wins
Answer: **6**

---

## Kotlin Solution

```kotlin
fun maxSubArray(nums: IntArray): Int {
    var currentSum = nums[0]
    var maxSum = nums[0]

    for (i in 1 until nums.size) {
        currentSum = maxOf(nums[i], currentSum + nums[i])
        maxSum = maxOf(maxSum, currentSum)
    }

    return maxSum
}
```

---

## Why Kotlin Shines Here

**`maxOf()`** — expresses the Kadane decision cleanly in one line:
```kotlin
currentSum = maxOf(nums[i], currentSum + nums[i])
// "Start fresh or extend?" answered in one expression
// vs Java: currentSum = Math.max(nums[i], currentSum + nums[i]);
```

**Starting from `nums[0]`** — handles all-negative arrays correctly:
```kotlin
var currentSum = nums[0]
var maxSum = nums[0]
// If all elements are negative, the answer is the least negative one
// Starting from 0 would give wrong result: maxSum = 0
```

---

## Common Mistake — Initialising to Zero

```kotlin
// ❌ Wrong
var maxSum = 0

// For nums = [-3, -1, -2], this returns 0
// But the correct answer is -1 (the least negative element)
// There's no subarray with sum 0 — the array has no positives
```

Always initialise `maxSum` and `currentSum` to `nums[0]`, not `0`.

---

## Variant — Return the Subarray Itself

If you need the actual subarray (not just the sum), track start and end indices:

```kotlin
fun maxSubArrayWithIndices(nums: IntArray): IntArray {
    var currentSum = nums[0]
    var maxSum = nums[0]
    var start = 0
    var tempStart = 0
    var end = 0

    for (i in 1 until nums.size) {
        if (nums[i] > currentSum + nums[i]) {
            currentSum = nums[i]
            tempStart = i
        } else {
            currentSum += nums[i]
        }

        if (currentSum > maxSum) {
            maxSum = currentSum
            start = tempStart
            end = i
        }
    }

    return nums.sliceArray(start..end)
}
```

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — single pass |
| **Space** | O(1) — only two variables |

---

## Key Takeaway

> At every step ask: "Is it better to extend what I have or start fresh?"
> If the running sum goes negative — it's dead weight. Drop it.
> That's Kadane's algorithm in one sentence.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/maximum-subarray/)

---

{% include kotlin-dsa-nav.html %}
