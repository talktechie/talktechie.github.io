---
title: "Arrays: Top K Frequent Elements — Kotlin Solution"
date: 2026-06-16
categories: [Kotlin DSA, Arrays]
tags: [top-k-frequent-elements, hashmap, bucket-sort, heap, medium, leetcode, leetcode-347]
leetcode_number: 347
leetcode_url: https://leetcode.com/problems/top-k-frequent-elements/
difficulty: Medium
permalink: /posts/kotlin-dsa-arrays-top-k-frequent-elements/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [347 — Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) |
| **Difficulty** | Medium |
| **Topic** | Arrays, HashMap, Bucket Sort, Heap |

> Given an integer array `nums` and an integer `k`, return the `k` most
> frequent elements. You may return the answer in any order.

**Example:**
```
Input:  nums = [1,1,1,2,2,3], k = 2
Output: [1,2]

Input:  nums = [1], k = 1
Output: [1]
```

**Constraints:**
- `1 <= nums.length <= 10⁵`
- `-10⁴ <= nums[i] <= 10⁴`
- `k` is in the range `[1, number of unique elements in nums]`
- It is guaranteed that the answer is unique
- **Follow-up:** Your algorithm's time complexity must be better than O(n log n)

---

## Approach

Every solution starts the same way — **count frequencies with a HashMap**.
The approaches differ in how they find the top `k` from those counts.

**Key insight (Bucket Sort):** The frequency of any element is at most `n` (if all
elements are the same). So we can create a `freq` array of size `n + 1` where
index `i` holds all elements that appear exactly `i` times. Then walk the
array from the end to collect the top `k`.

**Walk through `nums = [1,1,1,2,2,3], k = 2`:**
```
Step 1 — count frequencies:
  map = {1→3, 2→2, 3→1}

Step 2 — bucket by frequency (size n+1 = 7):
  freq[1] = [3]
  freq[2] = [2]
  freq[3] = [1]

Step 3 — read buckets from the end, collect until we have k=2:
  freq[6] → empty
  freq[5] → empty
  freq[4] → empty
  freq[3] → [1]  → result = [1]
  freq[2] → [2]  → result = [1, 2]  ← k reached, stop

Output: [1, 2] ✓
```

This avoids sorting entirely — O(n) time, O(n) space.

---

## Kotlin Solution

### Approach 1 — Bucket Sort (optimal, O(n))

```kotlin
fun topKFrequent(nums: IntArray, k: Int): IntArray {
    val count = HashMap<Int, Int>()
    for (n in nums) count[n] = (count[n] ?: 0) + 1

    // freq[i] = list of numbers that appear exactly i times
    val freq = Array(nums.size + 1) { mutableListOf<Int>() }
    for ((num, cnt) in count) freq[cnt].add(num)

    val result = mutableListOf<Int>()
    for (i in freq.indices.reversed()) {
        for (num in freq[i]) {
            result.add(num)
            if (result.size == k) return result.toIntArray()
        }
    }

    return result.toIntArray()
}
```

### Approach 2 — Min-Heap / Priority Queue (O(n log k))

```kotlin
fun topKFrequent(nums: IntArray, k: Int): IntArray {
    val count = HashMap<Int, Int>()
    for (n in nums) count[n] = (count[n] ?: 0) + 1

    // Min-heap ordered by frequency — keeps only the k most frequent
    val heap = PriorityQueue<Int>(compareBy { count[it] })

    for (num in count.keys) {
        heap.add(num)
        if (heap.size > k) heap.poll()  // evict least frequent
    }

    return heap.toIntArray()
}
```

### Approach 3 — Sort by frequency (O(n log n), simplest)

```kotlin
fun topKFrequent(nums: IntArray, k: Int): IntArray {
    val count = HashMap<Int, Int>()
    for (n in nums) count[n] = (count[n] ?: 0) + 1

    return count.keys
        .sortedByDescending { count[it] }
        .take(k)
        .toIntArray()
}
```

Simple and readable, but violates the follow-up constraint (better than O(n log n)).

---

## Why Bucket Sort Beats the Heap

The follow-up asks for better than O(n log n). Here's how each approach stacks up:

| | Approach | Time | Space |
|---|---|---|---|
| **Approach 1** | Bucket Sort | O(n) | O(n) |
| **Approach 2** | Min-Heap | O(n log k) | O(n + k) |
| **Approach 3** | Sort | O(n log n) | O(n) |

**Why bucket sort is O(n):**

The key observation — frequency is bounded by `n`. So instead of sorting
unbounded values, we index directly into a fixed-size array:

```kotlin
val freq = Array(nums.size + 1) { mutableListOf<Int>() }
for ((num, cnt) in count) freq[cnt].add(num)
// Direct index write — O(1) per element, O(n) total
```

**Why the heap is O(n log k) not O(n log n):**

```kotlin
heap.add(num)
if (heap.size > k) heap.poll()  // heap never grows beyond size k
// log k instead of log n — big difference when k << n
```

**`for ((num, cnt) in count)`** — Kotlin destructuring on map entries:
```kotlin
for ((num, cnt) in count) freq[cnt].add(num)
// vs Java: for (Map.Entry<Integer,Integer> e : count.entrySet())
//              freq[e.getValue()].add(e.getKey());
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Bucket Sort | Maximum performance, follow-up constraint must be met |
| Min-Heap | `k` is very small relative to `n`, clean and general |
| Sort | Readability matters more than performance, no follow-up constraint |

---

## Follow-up — Better than O(n log n)?

Yes — Bucket Sort achieves O(n). The trick is exploiting the constraint that
frequency can never exceed `n`, turning what looks like a sorting problem into
a direct array indexing problem.

---

## Complexity

| | Bucket Sort | Min-Heap | Sort |
|---|---|---|---|
| **Time** | O(n) | O(n log k) | O(n log n) |
| **Space** | O(n) | O(n + k) | O(n) |

Where `n` = length of `nums`, `k` = number of elements to return.

---

## Key Takeaway

> Count frequencies first — always. Then ask: do I need a full sort, or
> can I exploit bounds on the values? Here frequency is bounded by `n`,
> so bucket sort gives O(n) without sorting anything.
> Min-heap is the safe interview answer; bucket sort is the optimal one.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/top-k-frequent-elements/)

---

{% include kotlin-dsa-nav.html %}
