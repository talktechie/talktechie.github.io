---
title: "Heap: Find Median From Data Stream — Kotlin Solution"
date: 2026-06-23
categories: [Kotlin DSA, Heap]
tags: [find-median-from-data-stream, heap, priority-queue, design, hard, leetcode, leetcode-295]
leetcode_number: 295
leetcode_url: https://leetcode.com/problems/find-median-from-data-stream/
difficulty: Hard
permalink: /posts/kotlin-dsa-heap-find-median-from-data-stream/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [295 — Find Median From Data Stream](https://leetcode.com/problems/find-median-from-data-stream/) |
| **Difficulty** | Hard |
| **Topic** | Heap, Priority Queue, Design |

> Design a data structure that supports adding numbers from a stream and
> finding the **median** of all numbers added so far, at any point.
>
> - `addNum(num)` — adds an integer to the data structure.
> - `findMedian()` — returns the median of all elements so far.

**Example:**
```
Input:
["MedianFinder","addNum","addNum","findMedian","addNum","findMedian"]
[[],[1],[2],[],[3],[]]

Output:
[null,null,null,1.5,null,2.0]

Explanation:
MedianFinder mf = new MedianFinder();
mf.addNum(1);          // stream: [1]
mf.addNum(2);          // stream: [1,2]
mf.findMedian();       // (1+2)/2 = 1.5
mf.addNum(3);          // stream: [1,2,3]
mf.findMedian();       // middle element = 2.0
```

**Constraints:**
- `-10⁵ <= num <= 10⁵`
- At most `5 * 10⁴` calls to `addNum` and `findMedian` combined
- **Follow-up:** if all numbers are in range `[0,100]`, or if 99% of
  numbers are in range `[0,100]`, can you optimize?

---

## Approach

This is the capstone problem of the Heap phase, combining a max-heap and
a min-heap **simultaneously** — the "two heaps" pattern.

**Key insight — split the stream into two balanced halves:** maintain a
**max-heap** holding the smaller half of all numbers seen so far, and a
**min-heap** holding the larger half. Keep both heaps balanced in size
(differing by at most 1). With this split:
- The max-heap's top is the **largest of the small half**.
- The min-heap's top is the **smallest of the large half**.
- The median is either the max-heap's top alone (odd total count), or the
  average of both tops (even total count).

**Why this works:** by construction, every value in the max-heap (small
half) is ≤ every value in the min-heap (large half) — so the boundary
between the two heaps is exactly where the median sits.

**Walk through `addNum(1)`, `addNum(2)`, `findMedian()`, `addNum(3)`,
`findMedian()`:**
```
addNum(1): maxHeap=[1], minHeap=[]    (balanced: sizes 1,0)
addNum(2): tentatively maxHeap=[1,2]... but 2 should go to minHeap since
           it's >= maxHeap's top after a balance check
           → maxHeap=[1], minHeap=[2]  (sizes 1,1 — balanced)

findMedian(): sizes equal (1,1) → average of both tops = (1+2)/2 = 1.5 ✓

addNum(3): 3 >= maxHeap.top(1) → goes toward minHeap → minHeap=[2,3]
           now minHeap.size(2) > maxHeap.size(1)+1 → rebalance:
           pop minHeap's smallest (2) → push to maxHeap
           → maxHeap=[1,2], minHeap=[3]  (sizes 2,1 — balanced)

findMedian(): maxHeap larger by 1 → median = maxHeap.top = 2.0 ✓
```

---

## Kotlin Solution

### Approach 1 — Two heaps, kept balanced after every insertion (optimal)

```kotlin
class MedianFinder {
    private val small = PriorityQueue<Int>(compareByDescending { it })  // max-heap, lower half
    private val large = PriorityQueue<Int>()                             // min-heap, upper half

    fun addNum(num: Int) {
        // Decide which heap this number belongs to
        if (small.isEmpty() || num <= small.peek()) {
            small.add(num)
        } else {
            large.add(num)
        }

        // Rebalance — sizes must differ by at most 1
        if (small.size > large.size + 1) {
            large.add(small.poll())
        } else if (large.size > small.size + 1) {
            small.add(large.poll())
        }
    }

    fun findMedian(): Double {
        return when {
            small.size > large.size -> small.peek().toDouble()
            large.size > small.size -> large.peek().toDouble()
            else -> (small.peek() + large.peek()) / 2.0
        }
    }
}
```

### Approach 2 — Sorted insertion into a list (simpler, slower per insert)

```kotlin
class MedianFinder {
    private val nums = mutableListOf<Int>()

    fun addNum(num: Int) {
        val idx = nums.binarySearch(num).let { if (it < 0) -it - 1 else it }
        nums.add(idx, num)
    }

    fun findMedian(): Double {
        val n = nums.size
        return if (n % 2 == 1) {
            nums[n / 2].toDouble()
        } else {
            (nums[n / 2 - 1] + nums[n / 2]) / 2.0
        }
    }
}
```

Binary search finds the insertion point in O(log n), but the actual
`add(idx, ...)` still shifts elements, costing O(n) per insertion overall
— far slower than the two-heap approach as the stream grows large, though
`findMedian` itself is a trivially fast O(1) index lookup.

---

## Why Two Heaps, Not One

A single heap could give you the min or the max efficiently, but the
**median** requires knowing about the "middle" of the distribution — which
isn't something either a pure min-heap or max-heap can expose directly.
Splitting into two balanced heaps turns "find the middle" into "look at
the boundary between two halves," which both heap tops give you for free:

```kotlin
if (small.isEmpty() || num <= small.peek()) {
    small.add(num)   // belongs to the lower half
} else {
    large.add(num)   // belongs to the upper half
}
```

**Why the rebalancing step matters:** without it, one heap could grow
much larger than the other, and the "boundary" assumption (that
`small`'s top and `large`'s top straddle the true median) would break.
Keeping sizes within 1 of each other guarantees the median is always
exactly at that boundary:

```kotlin
if (small.size > large.size + 1) {
    large.add(small.poll())   // move the excess element to the other heap
} else if (large.size > small.size + 1) {
    small.add(large.poll())
}
```

**Why `findMedian` checks size first:** if one heap has one more element
than the other, that heap's top alone is the median (the "extra" middle
element). If they're equal in size, the median sits exactly between both
tops, hence the average.

---

## Addressing the Follow-Up: Bounded Value Ranges

If all numbers are guaranteed to be in `[0, 100]`, a **counting array**
of size 101 can track frequency directly — no heap needed. Finding the
median becomes a linear (or binary-search-augmented) scan over the
counting array to locate the middle index, achievable in O(100) =
effectively O(1) per query. If 99% of numbers fall in `[0,100]` with
rare outliers, a hybrid approach — a counting array for the common range
plus two small heaps for outliers — balances simplicity and correctness.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Two heaps (Approach 1) | Always — O(log n) insert, O(1) median, the standard solution |
| Sorted list (Approach 2) | Small streams only; O(n) insert doesn't scale |

---

## Complexity

| | addNum | findMedian |
|---|---|---|
| **Time (Two Heaps)** | O(log n) | O(1) |
| **Time (Sorted List)** | O(n) | O(1) |
| **Space** | O(n) | |

---

## Key Takeaway

> The "two heaps" pattern — a max-heap for the lower half, a min-heap for
> the upper half, kept balanced in size — is the standard technique
> whenever a streaming median (or more generally, any "running middle
> value") is needed. This closes out the Heap phase by combining
> everything: max-heap and min-heap behavior from earlier problems, used
> together rather than in isolation, demonstrating that the most powerful
> heap-based solutions often come from combining multiple heaps rather
> than reaching for just one.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/find-median-from-data-stream/)

---

{% include kotlin-dsa-nav.html %}
