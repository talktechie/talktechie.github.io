---
title: "Arrays: Product of Array Except Self — Kotlin Solution"
date: 2026-06-07
categories: [Kotlin DSA, Arrays]
tags: [product-of-array-except-self, prefix-product, medium, leetcode, leetcode-238, arrays]
leetcode_number: 238
leetcode_url: https://leetcode.com/problems/product-of-array-except-self/
difficulty: Medium
permalink: /posts/kotlin-dsa-arrays-product-except-self/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [238 — Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/) |
| **Difficulty** | Medium |
| **Topic** | Arrays, Prefix Product |

> Given an integer array `nums`, return an array `answer` such that `answer[i]`
> is equal to the product of all elements of `nums` except `nums[i]`.
>
> You must solve it in **O(n)** time and **without using division**.

**Example:**
```
Input:  nums = [1, 2, 3, 4]
Output: [24, 12, 8, 6]

// answer[0] = 2 * 3 * 4 = 24
// answer[1] = 1 * 3 * 4 = 12
// answer[2] = 1 * 2 * 4 = 8
// answer[3] = 1 * 2 * 3 = 6
```

---

## Approach

The obvious answer is divide the total product by each element — but the problem
explicitly forbids division. And even if it didn't, division breaks when there's
a zero in the array.

**Key insight:** For each index `i`, the answer is:
> **(product of everything to the left of i)** × **(product of everything to the right of i)**

We can compute these in two passes — one left to right, one right to left.

**Pass 1 — Left prefix products:**
```
nums    = [1,  2,  3,  4]
prefix  = [1,  1,  2,  6]   // prefix[i] = product of all elements before i
```

**Pass 2 — Multiply right suffix as we go:**
```
Start from the right with suffix = 1
i=3: result[3] = prefix[3] * suffix = 6 * 1 = 6,  suffix = 4
i=2: result[2] = prefix[2] * suffix = 2 * 4 = 8,  suffix = 12
i=1: result[1] = prefix[1] * suffix = 1 * 12 = 12, suffix = 24
i=0: result[0] = prefix[0] * suffix = 1 * 24 = 24, suffix = 24
```

No division. Two passes. O(1) extra space (excluding the output array).

---

## Kotlin Solution

```kotlin
fun productExceptSelf(nums: IntArray): IntArray {
    val n = nums.size
    val result = IntArray(n)

    // Pass 1: fill result with left prefix products
    result[0] = 1
    for (i in 1 until n) {
        result[i] = result[i - 1] * nums[i - 1]
    }

    // Pass 2: multiply right suffix products into result
    var suffix = 1
    for (i in n - 1 downTo 0) {
        result[i] *= suffix
        suffix *= nums[i]
    }

    return result
}
```

---

## Why Kotlin Shines Here

**`downTo`** — reverse iteration reads naturally:
```kotlin
for (i in n - 1 downTo 0)
// vs Java: for (int i = n - 1; i >= 0; i--)
```

**`IntArray(n)`** — creates a zero-initialised primitive int array directly,
no boxing overhead:
```kotlin
val result = IntArray(n)
// vs Java: int[] result = new int[n];
// Same length — Kotlin's version is idiomatic and avoids Integer wrapper
```

**`*=` on array elements** — compact in-place update:
```kotlin
result[i] *= suffix
// Clean, no temp variable needed
```

---

## Why Not Just Divide?

Two reasons:

1. The problem explicitly forbids it
2. It breaks with zeros:

```kotlin
// nums = [1, 0, 3, 4]  total product = 0
// answer[0] should be 0 * 3 * 4 = 0
// answer[1] should be 1 * 3 * 4 = 12  ← can't get this via 0 / 0
```

The prefix/suffix approach handles zeros naturally — no special casing needed.

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — two passes |
| **Space** | O(1) extra — output array doesn't count |

---

## Key Takeaway

> When you need the product of everything except one element — split it into
> what's on the left and what's on the right. Two passes, no division, handles zeros cleanly.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/product-of-array-except-self/)

---

{% include kotlin-dsa-nav.html %}
