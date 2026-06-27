---
title: "Heap: Last Stone Weight — Kotlin Solution"
date: 2026-06-23
categories: [Kotlin DSA, Heap]
tags: [last-stone-weight, heap, priority-queue, array, easy, leetcode, leetcode-1046]
leetcode_number: 1046
leetcode_url: https://leetcode.com/problems/last-stone-weight/
difficulty: Easy
permalink: /posts/kotlin-dsa-heap-last-stone-weight/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [1046 — Last Stone Weight](https://leetcode.com/problems/last-stone-weight/) |
| **Difficulty** | Easy |
| **Topic** | Heap, Priority Queue, Array |

> You are given an array `stones` of weights. Repeatedly pick the **two
> heaviest** stones and smash them together:
> - If they're equal weight, both are destroyed.
> - Otherwise, the smaller stone is destroyed, and the larger one's new
>   weight becomes `(larger - smaller)`.
>
> Continue until at most one stone remains. Return that stone's weight,
> or `0` if no stones remain.

**Example:**
```
Input:  stones = [2,7,4,1,8,1]
Output: 1
Explanation:
  smash 8,7 → 1   stones: [2,4,1,1,1]
  smash 4,2 → 2   stones: [2,1,1,1]
  smash 2,1 → 1   stones: [1,1,1]
  smash 1,1 → 0   stones: [1]
  → 1 stone remains, weight 1

Input:  stones = [1]
Output: 1
```

**Constraints:**
- `1 <= stones.length <= 30`
- `1 <= stones[i] <= 1000`

---

## Approach

Repeatedly needing "the two largest values" is the textbook use case for a
**max-heap**. After every smash, the new (or removed) weight needs to be
re-inserted into consideration — a heap handles this naturally.

**Key insight:** Push every stone into a max-heap. Repeatedly pop the two
largest, compute the difference, and push the difference back (if
non-zero). Stop when the heap has 0 or 1 stones left.

**Walk through `stones = [2,7,4,1,8,1]`:**
```
max-heap: {8,7,4,2,1,1}

pop 8, pop 7 → diff=1 → push 1 → heap: {4,2,1,1,1}
pop 4, pop 2 → diff=2 → push 2 → heap: {2,1,1,1}
pop 2, pop 1 → diff=1 → push 1 → heap: {1,1,1}
pop 1, pop 1 → diff=0 → push nothing → heap: {1}

heap.size == 1 → return heap.peek() = 1 ✓
```

---

## Kotlin Solution

### Approach 1 — Max-heap via PriorityQueue with reversed comparator (optimal)

```kotlin
fun lastStoneWeight(stones: IntArray): Int {
    val maxHeap = PriorityQueue<Int>(compareByDescending { it })
    for (s in stones) maxHeap.add(s)

    while (maxHeap.size > 1) {
        val first = maxHeap.poll()
        val second = maxHeap.poll()
        val diff = first - second

        if (diff > 0) maxHeap.add(diff)
    }

    return if (maxHeap.isEmpty()) 0 else maxHeap.peek()
}
```

### Approach 2 — Sort and re-insert manually (no PriorityQueue)

```kotlin
fun lastStoneWeight(stones: IntArray): Int {
    val list = stones.toMutableList()

    while (list.size > 1) {
        list.sortDescending()
        val diff = list[0] - list[1]
        list.removeAt(0)
        list.removeAt(0)
        if (diff > 0) list.add(diff)
    }

    return if (list.isEmpty()) 0 else list[0]
}
```

Re-sorts the entire list on every iteration — correct, but O(n log n) per
smash instead of O(log n), making it noticeably slower for larger inputs
(though still fine given this problem's small constraint of ≤30 stones).

---

## Why Kotlin's `PriorityQueue` Needs `compareByDescending` for a Max-Heap

Kotlin's (Java's) `PriorityQueue` is a **min-heap by default** — it
returns the smallest element first. To get max-heap behavior, we flip the
comparator:

```kotlin
val maxHeap = PriorityQueue<Int>(compareByDescending { it })
// "Descending" comparator means the queue treats LARGER values as
// having higher priority — so poll() returns the largest, not smallest
```

This same trick — wrapping the natural ordering with `compareByDescending`
— is the standard way to get max-heap behavior out of any min-heap-by-default
priority queue implementation.

**Why we only push the difference back when it's `> 0`:**
```kotlin
if (diff > 0) maxHeap.add(diff)
// diff == 0 means both stones were equal weight and BOTH are destroyed —
// nothing new enters the heap in that case
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Max-heap (Approach 1) | Always — O(log n) per operation, the correct general solution |
| Sort and re-insert (Approach 2) | Tiny inputs only; included to show the heap-free alternative |

---

## Complexity

| | Max-Heap | Sort-Based |
|---|---|---|
| **Time** | O(n log n) overall — n pops/pushes, each O(log n) | O(n² log n) — n iterations, each re-sorting |
| **Space** | O(n) | O(n) |

---

## Key Takeaway

> "Repeatedly take the two largest (or smallest) values, combine them
> somehow, and put the result back" is exactly what a heap is built for
> — each operation costs O(log n) instead of re-sorting from scratch
> every time. The `compareByDescending` trick for building a max-heap out
> of Kotlin's min-heap-by-default `PriorityQueue` is worth memorizing, as
> it appears throughout this phase whenever "largest first" behavior is
> needed.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/last-stone-weight/)

---

{% include kotlin-dsa-nav.html %}
