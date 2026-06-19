---
title: "Binary Search: Median of Two Sorted Arrays — Kotlin Solution"
date: 2026-06-18
categories: [Kotlin DSA, Binary Search]
tags: [median-of-two-sorted-arrays, binary-search, array, partition, hard, leetcode, leetcode-4]
leetcode_number: 4
leetcode_url: https://leetcode.com/problems/median-of-two-sorted-arrays/
difficulty: Hard
permalink: /posts/kotlin-dsa-binary-search-median-of-two-sorted-arrays/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [4 — Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/) |
| **Difficulty** | Hard |
| **Topic** | Binary Search, Array, Partition |

> Given two sorted arrays `nums1` and `nums2` of size `m` and `n`, return
> the median of the two arrays combined.
>
> **The overall run time complexity should be O(log(m+n)).**

**Example:**
```
Input:  nums1 = [1,3], nums2 = [2]
Output: 2.0
Explanation: merged = [1,2,3], median = 2

Input:  nums1 = [1,2], nums2 = [3,4]
Output: 2.5
Explanation: merged = [1,2,3,4], median = (2+3)/2 = 2.5
```

**Constraints:**
- `nums1.length == m`, `nums2.length == n`
- `0 <= m, n <= 1000`
- `1 <= m + n <= 2000`
- `-10⁶ <= nums1[i], nums2[i] <= 10⁶`

---

## Approach

Merging both arrays and finding the middle is O(m+n) — too slow for the
required O(log(m+n)). We need to find the median **without fully merging**.

**Key insight — binary search on the partition point.** The median splits
the combined array into a left half and a right half of (almost) equal
size. Instead of merging, binary search for a partition index in the
**smaller** array such that:

```
leftMax (max of left parts of both arrays) <= rightMin (min of right parts)
```

If that condition holds, we've found the correct partition — the median can
be computed directly from the four boundary values, with no merging required.

**Walk through `nums1 = [1,3], nums2 = [2]`:**
```
Always binary search on the smaller array. Here nums1 (size 2) is smaller... 
actually let's binary search on nums1, size m=2, n=1, total=3 (odd).

partition1 ranges 0..2 (how many elements of nums1 go to the left half)
partition2 = (m+n+1)/2 - partition1 = 2 - partition1

Try partition1=1:
  partition2 = 2 - 1 = 1
  left1 = nums1[0] = 1     right1 = nums1[1] = 3
  left2 = nums2[0] = 2     right2 = ∞ (no more elements in nums2)

  Is left1 <= right2 (1<=∞ ✓) and left2 <= right1 (2<=3 ✓)? Yes — valid partition!

  Total length is odd (3) → median = max(left1, left2) = max(1, 2) = 2 ✓
```

---

## Kotlin Solution

### Approach 1 — Binary search on partition point (optimal, O(log(min(m,n))))

```kotlin
fun findMedianSortedArrays(nums1: IntArray, nums2: IntArray): Double {
    // Always binary search on the smaller array for efficiency
    if (nums1.size > nums2.size) return findMedianSortedArrays(nums2, nums1)

    val m = nums1.size
    val n = nums2.size
    var low = 0
    var high = m

    while (low <= high) {
        val partition1 = low + (high - low) / 2
        val partition2 = (m + n + 1) / 2 - partition1

        val left1  = if (partition1 == 0) Int.MIN_VALUE else nums1[partition1 - 1]
        val right1 = if (partition1 == m) Int.MAX_VALUE else nums1[partition1]
        val left2  = if (partition2 == 0) Int.MIN_VALUE else nums2[partition2 - 1]
        val right2 = if (partition2 == n) Int.MAX_VALUE else nums2[partition2]

        if (left1 <= right2 && left2 <= right1) {
            // Correct partition found
            return if ((m + n) % 2 == 0) {
                (maxOf(left1, left2) + minOf(right1, right2)) / 2.0
            } else {
                maxOf(left1, left2).toDouble()
            }
        } else if (left1 > right2) {
            high = partition1 - 1   // too many elements from nums1 on the left — shrink
        } else {
            low = partition1 + 1    // too few elements from nums1 on the left — grow
        }
    }

    throw IllegalArgumentException("Input arrays are not sorted")
}
```

### Approach 2 — Merge-based (O(m+n), violates the complexity requirement)

```kotlin
fun findMedianSortedArrays(nums1: IntArray, nums2: IntArray): Double {
    val merged = (nums1 + nums2).also { it.sort() }
    val n = merged.size

    return if (n % 2 == 0) {
        (merged[n / 2 - 1] + merged[n / 2]) / 2.0
    } else {
        merged[n / 2].toDouble()
    }
}
```

Simple and correct, but O((m+n) log(m+n)) due to sorting — fails the
problem's explicit complexity requirement. Shown for contrast only.

---

## Why the Partition Condition Works

We're looking for a split where every element in the combined "left half"
is ≤ every element in the combined "right half" — that's the definition of
a correctly positioned median boundary.

```
left1 <= right2   AND   left2 <= right1
```

If `left1 > right2`, the left side of `nums1` has reached too far — it
contains a value bigger than something that should be on the right. We
need fewer elements from `nums1` on the left → shrink `partition1`.

If `left2 > right1`, the opposite is true → `nums1` needs **more** elements
on the left → grow `partition1`. (This is captured by the `else` branch
since `partition2` is derived from `partition1`.)

**Sentinel values `Int.MIN_VALUE` / `Int.MAX_VALUE`** handle edge partitions
cleanly — no special-casing needed when a partition takes 0 or all elements
from one array:
```kotlin
val left1  = if (partition1 == 0) Int.MIN_VALUE else nums1[partition1 - 1]
val right1 = if (partition1 == m) Int.MAX_VALUE else nums1[partition1]
// If partition1 == 0, there's nothing on the left from nums1 — MIN_VALUE
// guarantees it never wins a "left1 > right2" comparison incorrectly
```

**Always searching the smaller array** keeps complexity at O(log(min(m,n))):
```kotlin
if (nums1.size > nums2.size) return findMedianSortedArrays(nums2, nums1)
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Partition binary search (Approach 1) | Required — meets O(log(m+n)), the only acceptable solution here |
| Merge-based (Approach 2) | Never for submission — included only to build intuition for what "median of merged array" even means |

---

## Complexity

| | Partition Binary Search | Merge-Based |
|---|---|---|
| **Time** | O(log(min(m, n))) | O((m+n) log(m+n)) |
| **Space** | O(1) | O(m+n) |

---

## Key Takeaway

> The hardest binary search problem in this phase generalizes the pattern
> further: instead of searching for a value or an answer, we binary search
> for a **partition point** that satisfies a cross-array ordering condition.
> Sentinel values (`MIN_VALUE`/`MAX_VALUE`) eliminate edge-case branching, and
> always searching the smaller array keeps the log factor tight. Once the
> correct partition is found, the median is read directly off four boundary
> values — no merging required.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/median-of-two-sorted-arrays/)

---

{% include kotlin-dsa-nav.html %}
