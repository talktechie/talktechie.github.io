---
title: "Heap: Kth Largest Element In a Stream — Kotlin Solution"
date: 2026-06-23
categories: [Kotlin DSA, Heap]
tags: [kth-largest-element-in-a-stream, heap, priority-queue, design, easy, leetcode, leetcode-703]
leetcode_number: 703
leetcode_url: https://leetcode.com/problems/kth-largest-element-in-a-stream/
difficulty: Easy
permalink: /posts/kotlin-dsa-heap-kth-largest-element-in-a-stream/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [703 — Kth Largest Element In a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/) |
| **Difficulty** | Easy |
| **Topic** | Heap, Priority Queue, Design |

> Design a class to find the `k`-th largest element in a stream of
> numbers, where new numbers can be **added** over time.
>
> Implement `KthLargest`:
> - `KthLargest(k, nums)` — initializes with the integer `k` and an
>   initial stream `nums`.
> - `add(val)` — adds `val` to the stream and returns the current `k`-th
>   largest element.

**Example:**
```
Input:
["KthLargest","add","add","add","add","add"]
[[3,[4,5,8,2]],[3],[5],[10],[9],[4]]

Output:
[null,4,5,5,8,8]

Explanation:
KthLargest kthLargest = new KthLargest(3, [4,5,8,2]);
kthLargest.add(3);   // stream: [4,5,8,2,3] → 3rd largest is 4
kthLargest.add(5);   // stream: [4,5,8,2,3,5] → 3rd largest is 5
kthLargest.add(10);  // stream: [...,10] → 3rd largest is 5
kthLargest.add(9);   // stream: [...,9] → 3rd largest is 8
kthLargest.add(4);   // stream: [...,4] → 3rd largest is 8
```

**Constraints:**
- `1 <= k <= 10⁴`
- `0 <= nums.length <= 10⁴`
- `-10⁴ <= nums[i] <= 10⁴`
- `-10⁴ <= val <= 10⁴`
- At least `k` elements will exist in the stream when `add` is first called

---

## Approach

This is the entry point to the Heap phase — the problem that motivates
**why** a min-heap of fixed size `k` is the right tool whenever you need
"the kth largest" repeatedly, especially as new data streams in.

**Key insight:** Maintain a **min-heap of size at most k**. Its smallest
element (the heap's top) is always the **k-th largest** value seen so
far — because the heap holds exactly the `k` largest values, and the
smallest among those `k` is, by definition, the `k`-th largest overall.

When a new value arrives: push it in. If the heap now has more than `k`
elements, pop the smallest — it's no longer among the top `k`.

**Walk through `k=3`, stream starts `[4,5,8,2]`:**
```
Initialize: push all 4 values, then trim to size 3
  heap after pushing all: {2,4,5,8} (min-heap, top=2)
  size=4 > k=3 → pop smallest (2) → heap: {4,5,8}, top=4

add(3): push 3 → heap: {3,4,5,8} → size=4 > 3 → pop smallest (3) → heap: {4,5,8}
        top = 4 → return 4 ✓

add(5): push 5 → heap: {4,5,5,8} → pop smallest (4) → heap: {5,5,8}
        top = 5 → return 5 ✓

add(10): push 10 → heap: {5,5,8,10} → pop smallest (5) → heap: {5,8,10}
         top = 5 → return 5 ✓
```

---

## Kotlin Solution

### Approach 1 — Min-heap capped at size k (optimal)

```kotlin
class KthLargest(private val k: Int, nums: IntArray) {
    private val heap = PriorityQueue<Int>()

    init {
        for (n in nums) add(n)
    }

    fun add(`val`: Int): Int {
        heap.add(`val`)
        if (heap.size > k) heap.poll()
        return heap.peek()
    }
}
```

### Approach 2 — Maintain a sorted list (simpler, slower per call)

```kotlin
class KthLargest(private val k: Int, nums: IntArray) {
    private val sorted = nums.toMutableList()

    init {
        sorted.sort()
    }

    fun add(`val`: Int): Int {
        // Insert val into its correct sorted position
        val idx = sorted.binarySearch(`val`).let { if (it < 0) -it - 1 else it }
        sorted.add(idx, `val`)

        return sorted[sorted.size - k]
    }
}
```

Keeps the entire stream in sorted order rather than just the top `k` —
correct, but each `add` costs O(n) for the insertion (shifting elements),
significantly slower than the heap approach as the stream grows.

---

## Why a Min-Heap (Not a Max-Heap) Is the Right Choice Here

This is the detail that trips people up the first time: we want the
**k-th largest**, but we use a **min**-heap. The reasoning: if we keep
only the `k` largest values seen so far in a heap, the **smallest** value
among those `k` is exactly the one we want — anything smaller than that
heap's minimum couldn't possibly be in the top `k`.

```kotlin
heap.add(`val`)
if (heap.size > k) heap.poll()   // removes the SMALLEST — keeping only the top k
return heap.peek()                // the smallest of the top k = the kth largest overall
```

A max-heap, by contrast, would only ever give you the single largest
value efficiently — finding the "kth" largest with a max-heap would
require popping `k` times, which doesn't compose well with a continuously
growing stream.

**Why capping the heap at size `k` keeps each `add` fast:** the heap
never grows beyond `k+1` elements at any point, so every `add` operation
costs O(log k), completely independent of how many numbers have streamed
through in total.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Min-heap capped at k (Approach 1) | Always — O(log k) per add, the intended solution |
| Sorted list (Approach 2) | Understanding the problem only; O(n) per add doesn't scale |

---

## Complexity

| | Min-Heap | Sorted List |
|---|---|---|
| **Time (per add)** | O(log k) | O(n) |
| **Space** | O(k) | O(n) |

---

## Key Takeaway

> Whenever a problem asks for "the kth largest/smallest" and needs to
> stay efficient as new data arrives, a heap **capped at size k** is the
> standard tool — a min-heap for "kth largest," a max-heap for "kth
> smallest." The heap's exposed top is always exactly the answer, and
> capping the size keeps every operation at O(log k) regardless of how
> much data has streamed through. This pattern is reused directly in K
> Closest Points to Origin and Kth Largest Element In An Array later in
> this phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/kth-largest-element-in-a-stream/)

---

{% include kotlin-dsa-nav.html %}
