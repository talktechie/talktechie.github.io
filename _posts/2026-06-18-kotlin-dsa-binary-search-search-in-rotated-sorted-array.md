---
title: "Binary Search: Search In Rotated Sorted Array — Kotlin Solution"
date: 2026-06-18
categories: [Kotlin DSA, Binary Search]
tags: [search-rotated-sorted-array, binary-search, array, medium, leetcode, leetcode-33]
leetcode_number: 33
leetcode_url: https://leetcode.com/problems/search-in-rotated-sorted-array/
difficulty: Medium
permalink: /posts/kotlin-dsa-binary-search-search-in-rotated-sorted-array/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [33 — Search In Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/) |
| **Difficulty** | Medium |
| **Topic** | Binary Search, Array |

> There is an integer array `nums` sorted in ascending order with
> **distinct** values, possibly rotated at an unknown pivot. Given the
> rotated array and an integer `target`, return its index, or `-1` if
> not present.
>
> You must write an algorithm with **O(log n)** runtime.

**Example:**
```
Input:  nums = [4,5,6,7,0,1,2], target = 0
Output: 4

Input:  nums = [4,5,6,7,0,1,2], target = 3
Output: -1

Input:  nums = [1], target = 0
Output: -1
```

**Constraints:**
- `1 <= nums.length <= 5000`
- `-10⁴ <= nums[i] <= 10⁴`
- All values are unique
- `nums` is an ascending array possibly rotated
- `-10⁴ <= target <= 10⁴`

---

## Approach

Building on Find Minimum In Rotated Sorted Array — instead of finding the
seam, we use the same "which half is sorted" idea to navigate directly to
the target.

**Key insight:** At any point during binary search, **at least one side of
mid is guaranteed to be properly sorted** (even though the whole array isn't).
Determine which side is sorted by comparing `nums[left]` and `nums[mid]`.
Then check if `target` falls within that sorted side's value range — if so,
search there; otherwise it must be in the other half.

**Walk through `nums = [4,5,6,7,0,1,2], target = 0`:**
```
left=0, right=6 → mid=3 (value=7)
  nums[mid]=7 == target? No
  nums[left]=4 <= nums[mid]=7 → LEFT half [4,5,6,7] is sorted
  Is target=0 within [nums[left]=4, nums[mid]=7]? No (0 < 4)
  → search the right half → left=4

left=4, right=6 → mid=5 (value=1)
  nums[mid]=1 == target? No
  nums[left]=0 <= nums[mid]=1 → LEFT half [0,1] is sorted
  Is target=0 within [nums[left]=0, nums[mid]=1]? Yes
  → search left half → right=4

left=4, right=4 → mid=4 (value=0) → nums[mid]=0 == target → found! return 4 ✓
```

---

## Kotlin Solution

### Approach 1 — Identify sorted half, check range (optimal)

```kotlin
fun search(nums: IntArray, target: Int): Int {
    var left = 0
    var right = nums.lastIndex

    while (left <= right) {
        val mid = left + (right - left) / 2

        if (nums[mid] == target) return mid

        if (nums[left] <= nums[mid]) {
            // Left half [left..mid] is sorted
            if (nums[left] <= target && target < nums[mid]) {
                right = mid - 1   // target is in the sorted left half
            } else {
                left = mid + 1    // target must be in the right half
            }
        } else {
            // Right half [mid..right] is sorted
            if (nums[mid] < target && target <= nums[right]) {
                left = mid + 1    // target is in the sorted right half
            } else {
                right = mid - 1   // target must be in the left half
            }
        }
    }

    return -1
}
```

### Approach 2 — Find pivot first, then binary search the correct segment

```kotlin
fun search(nums: IntArray, target: Int): Int {
    val n = nums.size

    // Step 1 — find the index of the minimum (rotation pivot)
    var left = 0
    var right = n - 1
    while (left < right) {
        val mid = left + (right - left) / 2
        if (nums[mid] > nums[right]) left = mid + 1 else right = mid
    }
    val pivot = left

    // Step 2 — decide which segment target could be in, then binary search it
    fun binarySearch(lo: Int, hi: Int): Int {
        var l = lo
        var r = hi
        while (l <= r) {
            val mid = l + (r - l) / 2
            when {
                nums[mid] == target -> return mid
                nums[mid] < target  -> l = mid + 1
                else                 -> r = mid - 1
            }
        }
        return -1
    }

    return if (target >= nums[pivot]) {
        binarySearch(pivot, n - 1)   // target lies in the right (post-pivot) segment
    } else {
        binarySearch(0, pivot - 1)   // target lies in the left (pre-pivot) segment
    }
}
```

Two binary searches — one to find the pivot (reusing the LC 153 technique),
one to search the correct segment. More code, but builds directly on the
previous problem's solution.

---

## Why Checking "Is the Left Half Sorted" Matters

```kotlin
if (nums[left] <= nums[mid]) {
    // Left half is sorted
} else {
    // Right half must be sorted instead
}
```

Why must one side always be sorted? Because there's only **one** rotation
point in the whole array. If it falls in the right half, the left half is
untouched and fully sorted. If it falls in the left half, the right half is
untouched and fully sorted. It can never affect both halves simultaneously.

**The range check decides where to search next:**
```kotlin
if (nums[left] <= target && target < nums[mid]) {
    right = mid - 1   // target fits within the sorted left half's range
} else {
    left = mid + 1    // target is outside that range — must be elsewhere
}
```

This is the crux of the whole problem: once you know a half is sorted, you
can definitively say whether `target` belongs there using simple range
comparison — exactly like normal binary search within that segment.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Single-pass sorted-half check (Approach 1) | Always — one pass, O(log n), most efficient |
| Find-pivot-then-search (Approach 2) | Educational — explicitly reuses LC 153's pivot-finding logic |

---

## Complexity

| | |
|---|---|
| **Time** | O(log n) |
| **Space** | O(1) |

---

## Key Takeaway

> In a rotated sorted array, at least one half around `mid` is always fully
> sorted. Identify which half that is by comparing `nums[left]` to `nums[mid]`,
> then use a simple range check to see if `target` could be in that sorted
> half. If yes, recurse there; if no, the other half must contain it (or it
> doesn't exist). One pass, O(log n) — no need to find the rotation point first.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/search-in-rotated-sorted-array/)

---

{% include kotlin-dsa-nav.html %}
