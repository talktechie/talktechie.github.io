---
title: "Intervals: Minimum Interval to Include Each Query — Kotlin Solution"
date: 2026-06-27
categories: [Kotlin DSA, Intervals]
tags: [minimum-interval-to-include-each-query, intervals, heap, sorting, hard, leetcode, leetcode-1851]
leetcode_number: 1851
leetcode_url: https://leetcode.com/problems/minimum-interval-to-include-each-query/
difficulty: Hard
permalink: /posts/kotlin-dsa-intervals-minimum-interval-to-include-each-query/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [1851 — Minimum Interval to Include Each Query](https://leetcode.com/problems/minimum-interval-to-include-each-query/) |
| **Difficulty** | Hard |
| **Topic** | Intervals, Heap, Sorting |

> Given `intervals` and an array of `queries`, for each query find the
> size of the **smallest interval** that contains it (i.e.,
> `start <= query <= end`). Return `-1` for any query with no containing
> interval.

**Example:**
```
Input:  intervals = [[1,4],[2,4],[3,6],[4,4]], queries = [2,3,4,5]
Output: [3,3,1,4]
Explanation:
  query=2: intervals containing it: [1,4](size 4), [2,4](size 3) → smallest=3
  query=3: containing: [2,4](3), [3,6](4) → smallest=3
  query=4: containing: [1,4](4),[2,4](3),[3,6](4),[4,4](1) → smallest=1
  query=5: containing: [3,6](4) → smallest=4
```

**Constraints:**
- `1 <= intervals.length, queries.length <= 10⁵`
- `intervals[i].length == 2`
- `1 <= starti <= endi <= 10⁷`
- `1 <= queries[j] <= 10⁷`

---

## Approach

The naive approach — for every query, scan every interval checking
containment — is O(intervals × queries), far too slow for `10⁵` of each.
This closes out the Intervals phase by combining **two** previously
introduced techniques: sorting (to process things in order) and a
**min-heap** (to always have quick access to the smallest *currently
relevant* interval) — the exact same heap idea as Meeting Rooms II,
applied here to track interval **sizes** instead of end times.

**Key insight:** Sort both `intervals` (by start) and `queries` (keeping
track of original indices, since we must answer in original order).
Process queries in **increasing order**. For each query, first push
every interval whose start is `<=` the query into a min-heap, keyed by
**interval size** (`end - start + 1`). Then, **pop** any interval from
the heap whose `end` is `<` the current query — those intervals can never
satisfy this or any **future** (even larger) query, since queries only
increase from here. Whatever remains at the heap's top, if anything, is
the smallest valid interval for this query.

**Walk through `intervals = [[1,4],[2,4],[3,6],[4,4]]` sorted by start
(already sorted), `queries = [2,3,4,5]` sorted with original indices
`[(2,0),(3,1),(4,2),(5,3)]`:**
```
process query=2: push all intervals with start<=2: [1,4](size4), [2,4](size3)
  heap: {(4,end=4),(3,end=4)}  (size, end) pairs, min-heap by size
  pop any with end < 2? none → answer[0] = heap top size = 3

process query=3: push intervals with start<=3 not yet pushed: [3,6](size4)
  heap: {(3,end=4),(4,end=4),(4,end=6)}
  pop any with end < 3? none → answer[1] = 3

process query=4: push intervals with start<=4 not yet pushed: [4,4](size1)
  heap: {(1,end=4),(3,end=4),(4,end=4),(4,end=6)}
  pop any with end < 4? none → answer[2] = 1

process query=5: no new intervals to push (all already pushed)
  pop any with end < 5? YES — pop (1,end=4),(3,end=4),(4,end=4) all have end=4<5
  heap left: {(4,end=6)} → answer[3] = 4

Final (reordered to original query order): [3,3,1,4] ✓
```

---

## Kotlin Solution

### Approach 1 — Sort both arrays, min-heap keyed by interval size (optimal)

```kotlin
fun minInterval(intervals: Array<IntArray>, queries: IntArray): IntArray {
    val sortedIntervals = intervals.sortedBy { it[0] }
    val sortedQueryIndices = queries.indices.sortedBy { queries[it] }

    val answer = IntArray(queries.size) { -1 }
    val heap = PriorityQueue<IntArray>(compareBy { it[0] })  // [size, end]

    var i = 0   // pointer into sortedIntervals

    for (idx in sortedQueryIndices) {
        val q = queries[idx]

        // Push every interval that has started by now (start <= q)
        while (i < sortedIntervals.size && sortedIntervals[i][0] <= q) {
            val (start, end) = sortedIntervals[i]
            heap.add(intArrayOf(end - start + 1, end))
            i++
        }

        // Remove intervals that have already ended before this query
        while (heap.isNotEmpty() && heap.peek()[1] < q) {
            heap.poll()
        }

        if (heap.isNotEmpty()) {
            answer[idx] = heap.peek()[0]
        }
    }

    return answer
}
```

---

## Why Processing Queries in Sorted Order Lets Us Safely Discard Expired Intervals

This is the critical insight that makes the heap-pruning step correct
and permanent — once we decide an interval is irrelevant, we never need
to reconsider it:

```kotlin
while (heap.isNotEmpty() && heap.peek()[1] < q) {
    heap.poll()   // this interval ends before q — and since FUTURE
                    // queries are even larger (we're processing in
                    // sorted order), it can never satisfy any of them either
}
```

If we processed queries in their original (arbitrary) order instead, an
interval discarded for being "too early" for one query might actually
still be needed for an *earlier*, not-yet-processed query — forcing us
to keep every interval around indefinitely, defeating the entire purpose
of pruning. Sorting queries first guarantees monotonic progress: once an
interval expires, it's gone for good.

**Why the heap is keyed by `[size, end]`, with size as the primary sort
key:** we want O(log n) access to the **smallest currently valid**
interval — that's exactly what a min-heap ordered by size gives us. The
`end` value travels alongside purely so we can check expiration
(`peek()[1] < q`) without needing a separate data structure.

**Why we must track original query indices (`sortedQueryIndices`):** the
problem requires answers in the **original** query order, but we need to
*process* them in sorted order for the pruning logic to work — sorting
indices (not the query values themselves) preserves the ability to write
each answer back to its correct original position.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Sort + min-heap (Approach 1) | The only practical approach at this problem's scale — O((n+q) log(n+q)) total |

---

## Complexity

| | |
|---|---|
| **Time** | O((n + q) log(n + q)) — n = intervals.length, q = queries.length; sorting both, plus heap operations |
| **Space** | O(n + q) — sorted arrays, heap, and the index tracking |

---

## Key Takeaway

> This closing problem of the Intervals phase combines sorting (to
> process both intervals and queries in a meaningful, monotonic order)
> with a min-heap (to maintain fast access to the smallest currently
> "alive" interval) — the same heap-based "track what's currently active"
> idea as Meeting Rooms II, but keyed by size instead of end time, and
> layered with an additional expiration-pruning step enabled specifically
> by processing queries in sorted order. Recognizing that sorting one or
> more inputs can convert what looks like an O(n×q) all-pairs problem
> into a single coordinated O((n+q) log(n+q)) sweep is one of the most
> valuable, broadly applicable lessons from this entire phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/minimum-interval-to-include-each-query/)

---

{% include kotlin-dsa-nav.html %}
