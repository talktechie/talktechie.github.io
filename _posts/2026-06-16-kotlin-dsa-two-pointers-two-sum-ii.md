---
title: "Two Pointers: Two Sum II — Kotlin Solution"
date: 2026-06-16
categories: [Kotlin DSA, Two Pointers]
tags: [two-sum-ii, two-pointers, binary-search, sorted-array, medium, leetcode, leetcode-167]
leetcode_number: 167
leetcode_url: https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/
difficulty: Medium
permalink: /posts/kotlin-dsa-two-pointers-two-sum-ii/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [167 — Two Sum II — Input Array Is Sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) |
| **Difficulty** | Medium |
| **Topic** | Two Pointers, Binary Search, Array |

> Given a **1-indexed** array of integers `numbers` that is already sorted
> in non-decreasing order, find two numbers that add up to `target`.
>
> Return their indices as `[index1, index2]` where `1 <= index1 < index2`.
>
> You must use **only constant extra space**.
> There is exactly one solution.

**Example:**
```
Input:  numbers = [2,7,11,15], target = 9
Output: [1,2]   // numbers[0] + numbers[1] = 2 + 7 = 9

Input:  numbers = [2,3,4], target = 6
Output: [1,3]   // numbers[0] + numbers[2] = 2 + 4 = 6

Input:  numbers = [-1,0], target = -1
Output: [1,2]   // numbers[0] + numbers[1] = -1 + 0 = -1
```

**Constraints:**
- `2 <= numbers.length <= 3 * 10⁴`
- `-1000 <= numbers[i] <= 1000`
- `numbers` is sorted in non-decreasing order
- `-1000 <= target <= 1000`
- Exactly one solution exists
- Must use O(1) extra space

---

## Approach

The HashMap approach from Two Sum (LC 1) costs O(n) space — the constraint here
forbids it. But there's something the original Two Sum didn't have: **sorted order**.

**Key insight:** Start `left` at index 0 and `right` at the last index.
- If `numbers[left] + numbers[right] == target` → found it
- If sum is **too small** → we need a bigger number → move `left` right
- If sum is **too big** → we need a smaller number → move `right` left

Sorted order guarantees every move brings us strictly closer to the target.
No element is revisited. Exactly one valid pair exists.

**Walk through `[2, 7, 11, 15], target = 9`:**
```
left=0 (2)  right=3 (15) → sum=17 > 9  → right--
left=0 (2)  right=2 (11) → sum=13 > 9  → right--
left=0 (2)  right=1 (7)  → sum=9  == 9 → return [1, 2] ✓
```

---

## Kotlin Solution

### Approach 1 — Two pointers (optimal, O(1) space)

```kotlin
fun twoSum(numbers: IntArray, target: Int): IntArray {
    var left = 0
    var right = numbers.lastIndex

    while (left < right) {
        val sum = numbers[left] + numbers[right]
        when {
            sum == target -> return intArrayOf(left + 1, right + 1)  // 1-indexed
            sum < target  -> left++
            else          -> right--
        }
    }

    return intArrayOf()  // guaranteed to find a solution
}
```

### Approach 2 — Binary search (O(n log n), for reference)

```kotlin
fun twoSum(numbers: IntArray, target: Int): IntArray {
    for (i in numbers.indices) {
        val complement = target - numbers[i]
        var lo = i + 1
        var hi = numbers.lastIndex

        while (lo <= hi) {
            val mid = lo + (hi - lo) / 2
            when {
                numbers[mid] == complement -> return intArrayOf(i + 1, mid + 1)
                numbers[mid] < complement  -> lo = mid + 1
                else                       -> hi = mid - 1
            }
        }
    }

    return intArrayOf()
}
```

O(n log n) — valid but inferior to the two-pointer O(n) solution.
Shown here to illustrate that binary search is an option when sorted,
but two pointers exploits the structure more fully.

---

## Why Two Pointers Works — The Proof

Every time we move a pointer, we eliminate a row or column of possibilities:

```
numbers = [2, 7, 11, 15], target = 9

Consider all pairs:
    2+7=9  ← answer
    2+11   2+15
    7+11   7+15
           11+15

When sum > target (e.g. 2+15=17): moving right-- eliminates all pairs
  containing 15 as the right element (7+15, 11+15) — all even larger.
When sum < target: moving left++ eliminates all pairs containing that
  left element with smaller right values — all even smaller.
```

At no point do we skip the correct pair. Correctness is guaranteed.

**`when` expression** — idiomatic Kotlin branching on the sum:
```kotlin
when {
    sum == target -> return intArrayOf(left + 1, right + 1)
    sum < target  -> left++
    else          -> right--
}
// Cleaner than if/else if/else — reads like a decision table
```

**`left + 1, right + 1`** — the output is 1-indexed:
```kotlin
return intArrayOf(left + 1, right + 1)
// Easy to forget — the array is 0-indexed internally but the answer is 1-indexed
```

---

## Two Sum vs Two Sum II

| | Two Sum (LC 1) | Two Sum II (LC 167) |
|---|---|---|
| **Array sorted?** | No | Yes |
| **Space allowed** | O(n) HashMap | O(1) only |
| **Approach** | HashMap complement | Two pointers |
| **Time** | O(n) | O(n) |

The sorted constraint is what makes two pointers possible — and what makes
it the "right" solution here. A HashMap would technically work but wastes
the sorted property entirely.

---

## Complexity

| | Two Pointers | Binary Search |
|---|---|---|
| **Time** | O(n) | O(n log n) |
| **Space** | O(1) | O(1) |

---

## Key Takeaway

> Sorted array + find a pair = two pointers from opposite ends.
> Sum too small → advance left. Sum too big → retreat right.
> Each move eliminates a guaranteed non-answer. O(n) time, O(1) space.
> This pattern — squeeze inward based on sorted comparison — is the
> foundation for 3Sum and Container With Most Water.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/)

---

{% include kotlin-dsa-nav.html %}
