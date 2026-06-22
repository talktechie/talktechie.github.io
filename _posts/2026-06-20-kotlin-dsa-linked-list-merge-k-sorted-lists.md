---
title: "Linked List: Merge K Sorted Lists — Kotlin Solution"
date: 2026-06-20
categories: [Kotlin DSA, Linked List]
tags: [merge-k-sorted-lists, linked-list, heap, priority-queue, divide-and-conquer, hard, leetcode, leetcode-23]
leetcode_number: 23
leetcode_url: https://leetcode.com/problems/merge-k-sorted-lists/
difficulty: Hard
permalink: /posts/kotlin-dsa-linked-list-merge-k-sorted-lists/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [23 — Merge K Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) |
| **Difficulty** | Hard |
| **Topic** | Linked List, Heap, Priority Queue, Divide and Conquer |

> You are given an array of `k` linked lists, each sorted in ascending
> order. Merge all the lists into one sorted linked list and return it.

**Example:**
```
Input:  lists = [[1,4,5],[1,3,4],[2,6]]
Output: [1,1,2,3,4,4,5,6]

Input:  lists = []
Output: []

Input:  lists = [[]]
Output: []
```

**Constraints:**
- `k == lists.length`
- `0 <= k <= 10⁴`
- `0 <= lists[i].length <= 500`
- Total nodes across all lists `<= 10⁴`
- `-10⁴ <= lists[i][j] <= 10⁴`
- Each `lists[i]` is sorted in ascending order

---

## Approach

This generalizes Merge Two Sorted Lists (LC 21) to `k` lists. The naive
extension — repeatedly merge two lists at a time, sequentially — works but
is inefficient: merging list 1 into the accumulator, then list 2, then
list 3... means early lists get re-scanned many times.

**Key insight — min-heap of list heads:** At any point, the smallest
remaining value across all `k` lists is the overall next value in the
result. A min-heap holding the **current head node of every non-empty
list** gives O(log k) access to that minimum. Pop it, append to the
result, and if that list has more nodes, push its new head back onto
the heap.

**Walk through `lists = [[1,4,5],[1,3,4],[2,6]]`:**
```
heap initially: [1(list0), 1(list1), 2(list2)]  (ties broken arbitrarily)

pop 1 (from list0) → result: [1]   → push list0's next: 4 → heap: [1,2,4]
pop 1 (from list1) → result: [1,1] → push list1's next: 3 → heap: [2,3,4]
pop 2 (from list2) → result: [1,1,2] → push list2's next: 6 → heap: [3,4,6]
pop 3 (from list1) → result: [1,1,2,3] → push list1's next: 4 → heap: [4,4,6]
pop 4 (from list0 or list1) → result: [1,1,2,3,4] → push next (5 or none) → heap: [4,5,6] or [4,6]
... continues until heap is empty ...

Result: [1,1,2,3,4,4,5,6] ✓
```

---

## Kotlin Solution

### Approach 1 — Min-heap of list nodes (optimal, most common interview answer)

```kotlin
class ListNode(var `val`: Int) {
    var next: ListNode? = null
}

fun mergeKLists(lists: Array<ListNode?>): ListNode? {
    val heap = PriorityQueue<ListNode>(compareBy { it.`val` })

    for (node in lists) {
        node?.let { heap.add(it) }
    }

    val dummy = ListNode(0)
    var curr = dummy

    while (heap.isNotEmpty()) {
        val smallest = heap.poll()
        curr.next = smallest
        curr = smallest

        smallest.next?.let { heap.add(it) }
    }

    return dummy.next
}
```

### Approach 2 — Divide and conquer (pairwise merge, O(N log k))

```kotlin
fun mergeKLists(lists: Array<ListNode?>): ListNode? {
    if (lists.isEmpty()) return null

    fun mergeTwo(a: ListNode?, b: ListNode?): ListNode? {
        val dummy = ListNode(0)
        var curr = dummy
        var p1 = a
        var p2 = b

        while (p1 != null && p2 != null) {
            if (p1.`val` <= p2.`val`) {
                curr.next = p1; p1 = p1.next
            } else {
                curr.next = p2; p2 = p2.next
            }
            curr = curr.next!!
        }
        curr.next = p1 ?: p2
        return dummy.next
    }

    var remaining = lists.toMutableList()
    while (remaining.size > 1) {
        val merged = mutableListOf<ListNode?>()
        for (i in remaining.indices step 2) {
            val l1 = remaining[i]
            val l2 = if (i + 1 < remaining.size) remaining[i + 1] else null
            merged.add(mergeTwo(l1, l2))
        }
        remaining = merged
    }

    return remaining.firstOrNull()
}
```

Pairs lists up and merges them two at a time, halving the number of lists
each round — `log k` rounds total, each doing O(N) work overall, giving
O(N log k). Same complexity as the heap approach, but no heap needed.

---

## Why the Heap Approach Achieves O(N log k)

Each of the `N` total nodes across all lists is pushed onto the heap
exactly once and popped exactly once. Each heap operation costs O(log k)
since the heap never holds more than `k` elements (one per active list)
at a time.

```kotlin
val heap = PriorityQueue<ListNode>(compareBy { it.`val` })
// heap size is bounded by k, NOT by N — this is what keeps each
// operation cheap even when N is large
```

**Pushing the popped node's `next` immediately maintains the invariant**
that the heap always contains the current "frontier" of every still-active
list:
```kotlin
val smallest = heap.poll()
curr.next = smallest
curr = smallest
smallest.next?.let { heap.add(it) }   // keep that list's next candidate in the race
```

**Why divide-and-conquer also reaches O(N log k):** at each of the
`log k` merge rounds, the *total* work across all pairwise merges in that
round is O(N) — every node is touched once per round, not once per list.
`log k` rounds × O(N) per round = O(N log k), matching the heap approach.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Min-heap (Approach 1) | Most common interview answer, conceptually simplest to explain |
| Divide and conquer (Approach 2) | Want to avoid the heap data structure, or asked for an alternative approach |

---

## Complexity

| | Min-Heap | Divide & Conquer |
|---|---|---|
| **Time** | O(N log k) | O(N log k) |
| **Space** | O(k) — heap size | O(log k) — recursion/merge rounds |

Where `N` = total number of nodes across all lists, `k` = number of lists.

---

## Key Takeaway

> Merging `k` sorted sequences efficiently is a heap's signature use case —
> keep the current "frontier" element of each sequence in the heap, and
> always extract the global minimum next. This generalizes the two-pointer
> merge from LC 21 to any number of lists, at the cost of a log factor for
> heap operations. Divide and conquer reaches the same complexity by
> halving the problem each round instead, useful when you want to avoid
> bringing in a priority queue.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/merge-k-sorted-lists/)

---

{% include kotlin-dsa-nav.html %}
