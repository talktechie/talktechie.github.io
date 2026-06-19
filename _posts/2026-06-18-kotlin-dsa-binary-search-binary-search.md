---
title: "Binary Search: Binary Search — Kotlin Solution"
date: 2026-06-18
categories: [Kotlin DSA, Binary Search]
tags: [binary-search, array, easy, leetcode, leetcode-704]
leetcode_number: 704
leetcode_url: https://leetcode.com/problems/binary-search/
difficulty: Easy
permalink: /posts/kotlin-dsa-binary-search-binary-search/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [704 — Binary Search](https://leetcode.com/problems/binary-search/) |
| **Difficulty** | Easy |
| **Topic** | Binary Search, Array |

> Given an array of integers `nums` sorted in ascending order, and an
> integer `target`, write a function to search `target` in `nums`.
> If found, return its index; otherwise return `-1`.
>
> You must write an algorithm with **O(log n)** runtime complexity.

**Example:**
```
Input:  nums = [-1,0,3,5,9,12], target = 9
Output: 4

Input:  nums = [-1,0,3,5,9,12], target = 2
Output: -1
```

**Constraints:**
- `1 <= nums.length <= 10⁴`
- `-10⁴ < nums[i], target < 10⁴`
- All values in `nums` are unique
- `nums` is sorted in ascending order

---

## Approach

This is the template every other problem in this phase builds on.

**Key insight:** A sorted array lets you eliminate **half the search space**
with a single comparison. Pick the middle element — if it's the target,
you're done. If the target is smaller, it can only exist to the left, so
discard the right half entirely. If larger, discard the left half. Repeat
on the remaining half until the range is empty.

**Walk through `nums = [-1,0,3,5,9,12], target = 9`:**
```
left=0, right=5 → mid=2 (value=3) → 3 < 9 → search right half → left=3
left=3, right=5 → mid=4 (value=9) → 9 == 9 → found! return 4 ✓
```

---

## Kotlin Solution

### Approach 1 — Iterative two pointers (optimal)

```kotlin
fun search(nums: IntArray, target: Int): Int {
    var left = 0
    var right = nums.lastIndex

    while (left <= right) {
        val mid = left + (right - left) / 2

        when {
            nums[mid] == target -> return mid
            nums[mid] < target  -> left = mid + 1
            else                 -> right = mid - 1
        }
    }

    return -1
}
```

### Approach 2 — Recursive

```kotlin
fun search(nums: IntArray, target: Int): Int {
    fun helper(left: Int, right: Int): Int {
        if (left > right) return -1

        val mid = left + (right - left) / 2

        return when {
            nums[mid] == target -> mid
            nums[mid] < target  -> helper(mid + 1, right)
            else                 -> helper(left, mid - 1)
        }
    }

    return helper(0, nums.lastIndex)
}
```

Same logic, expressed recursively. Uses O(log n) stack space — the iterative
version is generally preferred for that reason.

---

## Why `left + (right - left) / 2` Instead of `(left + right) / 2`

This is the single most important detail to get right in binary search:

```kotlin
val mid = left + (right - left) / 2
// vs
val mid = (left + right) / 2   // risk of integer overflow for very large left/right
```

In languages with fixed-size integers, `left + right` can overflow before the
division happens, if both are close to `Int.MAX_VALUE`. The first form avoids
this because the addition only ever adds a value ≤ `(right - left)`, keeping
the intermediate result bounded. It's a defensive habit worth building even
when the array size doesn't make overflow likely today.

**`when` for the three-way comparison** — cleaner than nested `if/else`:
```kotlin
when {
    nums[mid] == target -> return mid
    nums[mid] < target  -> left = mid + 1
    else                 -> right = mid - 1
}
```

**`left <= right` (not `<`)** — using `<=` is what allows a single-element
range to still be checked; using `<` would skip the last candidate.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Iterative (Approach 1) | Always preferred — O(1) space, no stack overflow risk |
| Recursive (Approach 2) | Demonstrating recursion clearly, smaller inputs |

---

## Complexity

| | |
|---|---|
| **Time** | O(log n) |
| **Space** | O(1) iterative / O(log n) recursive (call stack) |

---

## Key Takeaway

> Binary search eliminates half the remaining space with every comparison.
> The template — `left`, `right`, `mid`, three-way comparison — is the
> backbone for every other problem in this phase. Get comfortable with
> `left + (right - left) / 2` to avoid overflow, and remember `left <= right`
> is the correct loop condition, not `<`.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/binary-search/)

---

{% include kotlin-dsa-nav.html %}
