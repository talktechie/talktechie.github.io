---
title: "Heap: Kth Largest Element In An Array — Kotlin Solution"
date: 2026-06-23
categories: [Kotlin DSA, Heap]
tags: [kth-largest-element-in-an-array, heap, priority-queue, quickselect, array, medium, leetcode, leetcode-215]
leetcode_number: 215
leetcode_url: https://leetcode.com/problems/kth-largest-element-in-an-array/
difficulty: Medium
permalink: /posts/kotlin-dsa-heap-kth-largest-element-in-an-array/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [215 — Kth Largest Element In An Array](https://leetcode.com/problems/kth-largest-element-in-an-array/) |
| **Difficulty** | Medium |
| **Topic** | Heap, Priority Queue, Quickselect, Array |

> Given an integer array `nums` and an integer `k`, return the `k`-th
> **largest** element in the array (not the kth distinct element —
> duplicates count individually).
>
> You must solve it without sorting (or do better than the trivial
> O(n log n) sort, per the implied follow-up).

**Example:**
```
Input:  nums = [3,2,1,5,6,4], k = 2
Output: 5

Input:  nums = [3,2,3,1,2,4,5,5,6], k = 4
Output: 4
```

**Constraints:**
- `1 <= k <= nums.length <= 10⁵`
- `-10⁴ <= nums[i] <= 10⁴`

---

## Approach

This is a static (non-streaming) version of Kth Largest Element In a
Stream — same min-heap-capped-at-k idea works directly. But since the
entire array is available upfront (not arriving incrementally), there's
an even faster average-case approach: **Quickselect**, a partition-based
technique related to Quicksort.

**Key insight (Quickselect):** Quicksort's partition step places a pivot
correctly relative to everything else, then recurses on **both** sides.
Quickselect recognizes we only need **one** side — the side containing
the index we're looking for — and discards the other entirely, giving an
average O(n) instead of O(n log n).

**Walk through `nums = [3,2,1,5,6,4], k = 2`** (looking for the 2nd
largest, which is at sorted-descending index 1):
```
Using quickselect targeting index (n-k) in ascending-sorted terms = 6-2=4:

pivot = nums[last] = 4
partition around 4: elements < 4 go left, >= 4 go right
  → [3,2,1,4,6,5]  (4 ends up at index 3 after partitioning)

pivotIndex=3, target=4 → pivotIndex < target → search RIGHT half only
  subarray [6,5] (indices 4-5), target still 4

pivot = nums[last]=5
partition: → [6,5] stays roughly the same, pivot 5 lands at index 5... 
eventually narrows down to nums[4] = 5

Answer: 5 ✓ (the 2nd largest in [3,2,1,5,6,4] is indeed 5)
```

---

## Kotlin Solution

### Approach 1 — Min-heap capped at size k (simple, O(n log k))

```kotlin
fun findKthLargest(nums: IntArray, k: Int): Int {
    val minHeap = PriorityQueue<Int>()

    for (n in nums) {
        minHeap.add(n)
        if (minHeap.size > k) minHeap.poll()
    }

    return minHeap.peek()
}
```

### Approach 2 — Quickselect (optimal average case, O(n))

```kotlin
fun findKthLargest(nums: IntArray, k: Int): Int {
    val targetIndex = nums.size - k  // index of kth largest in ascending sorted order

    fun partition(left: Int, right: Int): Int {
        val pivot = nums[right]
        var i = left

        for (j in left until right) {
            if (nums[j] < pivot) {
                nums[i] = nums[j].also { nums[j] = nums[i] }
                i++
            }
        }
        nums[i] = nums[right].also { nums[right] = nums[i] }
        return i
    }

    var left = 0
    var right = nums.lastIndex

    while (true) {
        val pivotIndex = partition(left, right)

        when {
            pivotIndex == targetIndex -> return nums[pivotIndex]
            pivotIndex < targetIndex -> left = pivotIndex + 1
            else -> right = pivotIndex - 1
        }
    }
}
```

### Approach 3 — Sort directly (simplest, but technically the "naive" baseline)

```kotlin
fun findKthLargest(nums: IntArray, k: Int): Int {
    nums.sort()
    return nums[nums.size - k]
}
```

Perfectly valid and often fast enough in practice — included as the
baseline the problem implicitly nudges you to beat.

---

## Why Quickselect Achieves O(n) on Average

Quicksort recurses on **both** partitions, giving O(n log n) overall.
Quickselect only ever recurses on **one** side — the side that must
contain the target index — discarding the other half's information
completely:

```kotlin
when {
    pivotIndex == targetIndex -> return nums[pivotIndex]   // found it
    pivotIndex < targetIndex -> left = pivotIndex + 1        // only search right half
    else -> right = pivotIndex - 1                            // only search left half
}
```

Each partition step is O(current range size), and on average the range
shrinks geometrically (similar to binary search's halving, though with a
worse constant), giving an expected O(n) total — though a careful
adversarial input (or always picking the worst possible pivot) can
degrade this to O(n²) in the worst case. Random pivot selection or
median-of-three pivoting mitigates this risk in practice.

**Why `targetIndex = nums.size - k`:** the kth largest, in an
ascending-sorted array, sits at index `n - k` (the kth from the end).
Quickselect's job becomes "find whatever value ends up at this specific
index after partitioning," without needing to fully sort everything else.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Min-heap (Approach 1) | Simple to write, good enough for most cases, O(n log k) |
| Quickselect (Approach 2) | Want the best average-case performance, comfortable with in-place partitioning |
| Sort (Approach 3) | Quick baseline, fine unless explicitly asked to beat O(n log n) |

---

## Complexity

| | Min-Heap | Quickselect | Sort |
|---|---|---|---|
| **Time (average)** | O(n log k) | O(n) | O(n log n) |
| **Time (worst case)** | O(n log k) | O(n²) | O(n log n) |
| **Space** | O(k) | O(1) — in-place | O(1) or O(n) depending on sort |

---

## Key Takeaway

> When the full dataset is available upfront (not streaming), Quickselect
> is the asymptotically best average-case tool for "find the kth
> largest/smallest" — it's Quicksort's partition step, applied
> recursively to only the half that matters. The heap approach from Kth
> Largest Element In a Stream still works here too, and is simpler to
> write correctly — a reasonable trade-off to make depending on whether
> raw performance or implementation simplicity matters more for a given
> situation.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/kth-largest-element-in-an-array/)

---

{% include kotlin-dsa-nav.html %}
