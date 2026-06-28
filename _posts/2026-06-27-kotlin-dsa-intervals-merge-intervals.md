---
title: "Intervals: Merge Intervals — Kotlin Solution"
date: 2026-06-27
categories: [Kotlin DSA, Intervals]
tags: [merge-intervals, intervals, array, sorting, medium, leetcode, leetcode-56]
leetcode_number: 56
leetcode_url: https://leetcode.com/problems/merge-intervals/
difficulty: Medium
permalink: /posts/kotlin-dsa-intervals-merge-intervals/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [56 — Merge Intervals](https://leetcode.com/problems/merge-intervals/) |
| **Difficulty** | Medium |
| **Topic** | Intervals, Array, Sorting |

> Given an array of `intervals` (**not necessarily sorted**), merge all
> overlapping intervals and return an array of the non-overlapping
> intervals that cover all the input ranges.

**Example:**
```
Input:  intervals = [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
Explanation: [1,3] and [2,6] overlap, merging into [1,6]

Input:  intervals = [[1,4],[4,5]]
Output: [[1,5]]
Explanation: touching intervals (one ends exactly where the next starts)
             still count as overlapping and merge
```

**Constraints:**
- `1 <= intervals.length <= 10⁴`
- `intervals[i].length == 2`
- `0 <= starti <= endi <= 10⁴`

---

## Approach

Unlike Insert Interval (where the existing list was already sorted),
here the input arrives in **arbitrary order** — so the first and most
important step is to **sort by start time**. Once sorted, merging becomes
a simple linear scan: compare each interval to the most recently merged
one, and either extend it or start a new one.

**Key insight:** After sorting, if the current interval's start is `<=`
the **end of the last interval added to the result**, they overlap (or
touch) — extend the last result interval's end if needed. Otherwise, the
current interval starts a brand new, non-overlapping group.

**Walk through `intervals = [[1,3],[2,6],[8,10],[15,18]]`** (already
sorted by start in this example):
```
result = [[1,3]]   (seed with the first interval)

[2,6]: 2 <= result.last().end(3) → overlaps → extend:
  result.last() becomes [1, max(3,6)] = [1,6]
  result = [[1,6]]

[8,10]: 8 <= result.last().end(6)? NO → doesn't overlap → start new group
  result = [[1,6],[8,10]]

[15,18]: 15 <= result.last().end(10)? NO → start new group
  result = [[1,6],[8,10],[15,18]]

Final result: [[1,6],[8,10],[15,18]] ✓
```

---

## Kotlin Solution

### Approach 1 — Sort by start, then linear merge scan (optimal)

```kotlin
fun merge(intervals: Array<IntArray>): Array<IntArray> {
    if (intervals.isEmpty()) return intervals

    intervals.sortBy { it[0] }

    val result = mutableListOf<IntArray>()
    result.add(intervals[0])

    for (i in 1 until intervals.size) {
        val current = intervals[i]
        val lastMerged = result.last()

        if (current[0] <= lastMerged[1]) {
            // Overlaps (or touches) — extend the last merged interval's end
            lastMerged[1] = maxOf(lastMerged[1], current[1])
        } else {
            // No overlap — this starts a new group
            result.add(current)
        }
    }

    return result.toTypedArray()
}
```

### Approach 2 — Same idea, building a new IntArray instead of mutating in place

```kotlin
fun merge(intervals: Array<IntArray>): Array<IntArray> {
    val sorted = intervals.sortedBy { it[0] }
    val result = mutableListOf<IntArray>()

    for (interval in sorted) {
        if (result.isNotEmpty() && interval[0] <= result.last()[1]) {
            val last = result.removeAt(result.lastIndex)
            result.add(intArrayOf(last[0], maxOf(last[1], interval[1])))
        } else {
            result.add(interval)
        }
    }

    return result.toTypedArray()
}
```

Avoids mutating the original `intervals` array's inner `IntArray`
objects directly — instead creates fresh arrays for any merged result,
which can be a safer default if the caller expects the original input
arrays to remain untouched.

---

## Why Sorting by Start Time Is the Essential First Step

This is the single most important realization in the problem. Without
sorting, intervals that should merge could be scattered anywhere in the
input, making it impossible to merge correctly with a single linear
pass — you'd need to check every interval against every other one,
giving O(n²).

```kotlin
intervals.sortBy { it[0] }
// Once sorted by start, any two intervals that COULD overlap are
// guaranteed to be adjacent (or nearly so) in the sorted order —
// a single linear scan comparing consecutive intervals is now sufficient
```

After sorting, the **only** thing that can prevent merging consecutive
intervals is a genuine gap — if `current[0] > lastMerged[1]`, there's no
possible overlap, because every interval after `current` in sorted order
starts even later, so none of them could connect back to `lastMerged`
either.

**Why `lastMerged[1] = maxOf(lastMerged[1], current[1])` instead of just
`current[1]`:** the current interval might be entirely **contained
within** the already-merged range (e.g., merging `[1,10]` with `[3,5]`)
— taking the max ensures we never accidentally shrink the merged
interval's end.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| In-place mutation (Approach 1) | Slightly more efficient, fine if mutating input arrays is acceptable |
| Fresh arrays (Approach 2) | Want to avoid side effects on the original input |

---

## Complexity

| | |
|---|---|
| **Time** | O(n log n) — dominated by the sort; the merge scan itself is O(n) |
| **Space** | O(n) — the result list (and O(log n) or O(n) for the sort, depending on implementation) |

---

## Key Takeaway

> When intervals arrive unsorted, **sort by start time first** — this
> single step transforms an otherwise all-pairs comparison problem into
> a simple linear scan, since overlapping intervals are guaranteed to be
> adjacent after sorting. The merge condition itself
> (`current.start <= lastMerged.end`) and the extension rule
> (`max` of both ends) are exactly the same overlap-detection logic
> introduced in Insert Interval — Merge Intervals is really just "Insert
> Interval's merging step, applied repeatedly across an entire unsorted
> list instead of inserting a single new interval into an already-sorted
> one."

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/merge-intervals/)

---

{% include kotlin-dsa-nav.html %}
