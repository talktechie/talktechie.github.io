---
title: "Binary Search: Find Minimum In Rotated Sorted Array — Kotlin Solution"
date: 2026-06-18
categories: [Kotlin DSA, Binary Search]
tags: [find-minimum-rotated-sorted-array, binary-search, array, medium, leetcode, leetcode-153]
leetcode_number: 153
leetcode_url: https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/
difficulty: Medium
permalink: /posts/kotlin-dsa-binary-search-find-minimum-in-rotated-sorted-array/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [153 — Find Minimum In Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/) |
| **Difficulty** | Medium |
| **Topic** | Binary Search, Array |

> Suppose an array of length `n`, sorted in ascending order, is **rotated**
> between `1` and `n` times. Given the rotated array `nums` with all
> **unique** elements, return the minimum element.
>
> You must write an algorithm that runs in **O(log n)** time.

**Example:**
```
Input:  nums = [3,4,5,1,2]
Output: 1

Input:  nums = [4,5,6,7,0,1,2]
Output: 0

Input:  nums = [11,13,15,17]
Output: 11   // not rotated (or rotated n times, same result)
```

**Constraints:**
- `1 <= n <= 5000`
- `-5000 <= nums[i] <= 5000`
- All values are unique
- `nums` is sorted and rotated between `1` and `n` times

---

## Approach

A rotated sorted array splits into **two sorted segments** with the minimum
sitting exactly at the boundary between them. Linear scan finds it in O(n) —
we want O(log n), so we binary search for the boundary directly.

**Key insight:** Compare `nums[mid]` to `nums[right]`.
- If `nums[mid] > nums[right]`, the minimum must be **somewhere to the right
  of mid** (the left half, including mid, is the "high" sorted segment —
  the drop happens later).
- If `nums[mid] <= nums[right]`, the right half (including mid) is already
  sorted with no drop — the minimum is at `mid` or to its **left**.

This works regardless of where the rotation point is, and handles the
"not rotated at all" case naturally (the whole array is one sorted segment).

**Walk through `nums = [4,5,6,7,0,1,2]`:**
```
left=0, right=6 → mid=3 (value=7) → 7 > nums[6]=2 → min is right of mid → left=4
left=4, right=6 → mid=5 (value=1) → 1 <= nums[6]=2 → min is at mid or left → right=5
left=4, right=5 → mid=4 (value=0) → 0 <= nums[5]=1 → min is at mid or left → right=4
left=4, right=4 → loop ends (left == right)

Answer: nums[4] = 0 ✓
```

---

## Kotlin Solution

### Approach 1 — Binary search comparing mid to right (optimal)

```kotlin
fun findMin(nums: IntArray): Int {
    var left = 0
    var right = nums.lastIndex

    while (left < right) {
        val mid = left + (right - left) / 2

        if (nums[mid] > nums[right]) {
            left = mid + 1   // minimum is strictly right of mid
        } else {
            right = mid       // minimum is at mid or to its left
        }
    }

    return nums[left]  // left == right at this point
}
```

### Approach 2 — Compare mid to left boundary value instead

```kotlin
fun findMin(nums: IntArray): Int {
    var left = 0
    var right = nums.lastIndex

    if (nums[left] <= nums[right]) return nums[left]  // not rotated

    while (left < right) {
        val mid = left + (right - left) / 2

        if (nums[mid] >= nums[left]) {
            left = mid + 1
        } else {
            right = mid
        }
    }

    return nums[left]
}
```

Comparing against `nums[left]` instead of `nums[right]` works too, but
needs an extra check upfront for the "array isn't rotated at all" case.
Comparing to `nums[right]` (Approach 1) handles that case for free.

---

## Why Comparing to `nums[right]` Avoids an Edge Case

If the array isn't rotated (or rotated a full `n` times, same thing), there's
no "drop" anywhere. Let's see why Approach 1 still works without a special check:

```
nums = [11,13,15,17] (no rotation)

left=0, right=3 → mid=1 (13) → 13 <= nums[3]=17 → right=1
left=0, right=1 → mid=0 (11) → 11 <= nums[1]=13 → right=0
left=0, right=0 → loop ends

Answer: nums[0] = 11 ✓ — correctly identifies the array's own first element
```

Because the whole array is sorted, `nums[mid] <= nums[right]` is always true,
so `right` keeps shrinking toward `left` without ever taking the "jump right"
branch — naturally converging to index 0.

**`left < right` (not `<=`)** — different loop condition than LC 704:
```kotlin
while (left < right) { ... }
return nums[left]
// We're not searching for an exact value match — we're narrowing down to
// a single index. The loop stops when left == right, and that index is the answer.
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Compare to `nums[right]` (Approach 1) | Always — handles the unrotated case without special-casing |
| Compare to `nums[left]` (Approach 2) | Educational — shows the alternative, needs an upfront check |

---

## Complexity

| | |
|---|---|
| **Time** | O(log n) |
| **Space** | O(1) |

---

## Key Takeaway

> A rotated sorted array is two sorted segments glued together, and the
> minimum sits at the seam. Compare `nums[mid]` to `nums[right]` — if mid
> is bigger, the seam is to the right; otherwise it's at or before mid.
> The loop converges `left` and `right` onto the seam itself. This
> "compare against an endpoint to figure out which half holds the boundary"
> idea is the foundation for Search In Rotated Sorted Array next.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/)

---

{% include kotlin-dsa-nav.html %}
