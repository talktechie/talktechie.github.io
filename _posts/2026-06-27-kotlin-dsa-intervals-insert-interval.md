---
title: "Intervals: Insert Interval — Kotlin Solution"
date: 2026-06-27
categories: [Kotlin DSA, Intervals]
tags: [insert-interval, intervals, array, medium, leetcode, leetcode-57]
leetcode_number: 57
leetcode_url: https://leetcode.com/problems/insert-interval/
difficulty: Medium
permalink: /posts/kotlin-dsa-intervals-insert-interval/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [57 — Insert Interval](https://leetcode.com/problems/insert-interval/) |
| **Difficulty** | Medium |
| **Topic** | Intervals, Array |

> Given a list of **non-overlapping** intervals sorted by start time, and
> a new interval, insert it into the list, merging any overlaps as
> needed. Return the resulting list, still sorted and non-overlapping.

**Example:**
```
Input:  intervals = [[1,3],[6,9]], newInterval = [2,5]
Output: [[1,5],[6,9]]
Explanation: [2,5] overlaps with [1,3], merging into [1,5]

Input:  intervals = [[1,2],[3,5],[6,7],[8,10],[12,16]], newInterval = [4,8]
Output: [[1,2],[3,10],[12,16]]
Explanation: [4,8] overlaps with [3,5], [6,7], AND [8,10] — all three
             merge together into one [3,10]
```

**Constraints:**
- `0 <= intervals.length <= 10⁴`
- `intervals[i].length == 2`
- `0 <= start <= end <= 10⁵`
- Existing intervals are sorted by start and non-overlapping
- `newInterval.length == 2`

---

## Approach

This is the entry point to the Intervals phase, establishing the core
**three-phase scan** technique used throughout: process intervals that
come strictly **before** the new one, then merge everything that
**overlaps**, then process intervals strictly **after**.

**Key insight:** Because the input is already sorted, we can walk through
it in a single linear pass, naturally falling into three buckets:
1. Intervals ending **before** the new interval starts — add as-is.
2. Intervals overlapping the new interval — merge them all into one
   expanding interval (`start = min of all starts`, `end = max of all
   ends`).
3. Intervals starting **after** the (possibly expanded) new interval
   ends — add as-is.

**Walk through `intervals = [[1,2],[3,5],[6,7],[8,10],[12,16]],
newInterval = [4,8]`:**
```
[1,2]: ends (2) before newInterval starts (4) → add as-is → result=[[1,2]]

[3,5]: overlaps newInterval (3 <= 8 and 5 >= 4) → merge:
  merged = [min(4,3), max(8,5)] = [3,8]

[6,7]: overlaps merged [3,8] (6<=8, 7>=3) → merge:
  merged = [min(3,6), max(8,7)] = [3,8]  (unchanged, already contains it)

[8,10]: overlaps merged [3,8] (8<=8, 10>=3) → merge:
  merged = [min(3,8), max(8,10)] = [3,10]

[12,16]: starts (12) after merged ends (10) → no more overlap →
  add merged=[3,10] to result, then add [12,16] as-is

Result: [[1,2],[3,10],[12,16]] ✓
```

---

## Kotlin Solution

### Approach 1 — Three-phase linear scan (optimal)

```kotlin
fun insert(intervals: Array<IntArray>, newInterval: IntArray): Array<IntArray> {
    val result = mutableListOf<IntArray>()
    var i = 0
    val n = intervals.size

    // Phase 1 — add all intervals ending before newInterval starts
    while (i < n && intervals[i][1] < newInterval[0]) {
        result.add(intervals[i])
        i++
    }

    // Phase 2 — merge all overlapping intervals into newInterval
    var mergedStart = newInterval[0]
    var mergedEnd = newInterval[1]
    while (i < n && intervals[i][0] <= mergedEnd) {
        mergedStart = minOf(mergedStart, intervals[i][0])
        mergedEnd = maxOf(mergedEnd, intervals[i][1])
        i++
    }
    result.add(intArrayOf(mergedStart, mergedEnd))

    // Phase 3 — add all remaining intervals (starting after the merged one ends)
    while (i < n) {
        result.add(intervals[i])
        i++
    }

    return result.toTypedArray()
}
```

### Approach 2 — Generic overlap check, single pass (slightly more concise)

```kotlin
fun insert(intervals: Array<IntArray>, newInterval: IntArray): Array<IntArray> {
    val result = mutableListOf<IntArray>()
    var start = newInterval[0]
    var end = newInterval[1]
    var inserted = false

    for (interval in intervals) {
        when {
            interval[1] < start -> result.add(interval)   // entirely before — no overlap yet
            interval[0] > end -> {                          // entirely after — overlap phase is done
                if (!inserted) {
                    result.add(intArrayOf(start, end))
                    inserted = true
                }
                result.add(interval)
            }
            else -> {                                        // overlaps — expand the merged range
                start = minOf(start, interval[0])
                end = maxOf(end, interval[1])
            }
        }
    }

    if (!inserted) result.add(intArrayOf(start, end))   // newInterval was never inserted (e.g., goes at the very end)

    return result.toTypedArray()
}
```

---

## Why the Overlap Condition Is `intervals[i][0] <= mergedEnd`

This is the precise condition that determines whether an interval should
be merged versus treated as "starting after" the merged range:

```kotlin
while (i < n && intervals[i][0] <= mergedEnd) {
    mergedStart = minOf(mergedStart, intervals[i][0])
    mergedEnd = maxOf(mergedEnd, intervals[i][1])
    i++
}
```

Two intervals overlap (or touch) if one starts at or before the other
ends. Using `<=` (not `<`) means intervals that are exactly adjacent
(e.g., `[1,5]` and `[5,8]`) are still merged together into `[1,8]` —
whether this is the desired behavior depends on the problem's exact
definition of "overlap," and this problem's examples confirm touching
intervals should merge.

**Why Phase 1's condition checks `intervals[i][1] < newInterval[0]`**
(strict less-than): an interval is safely **before** the new one only if
it ends strictly before the new interval starts — if it ends exactly
when the new interval starts, they touch and should be merged (handled
correctly by Phase 2's `<=` check, not Phase 1's).

**Why `mergedStart`/`mergedEnd` keep expanding across multiple
overlapping intervals (Approach 1):** the `while` loop doesn't stop after
the first overlap — it keeps consuming consecutive overlapping intervals,
continuously widening the merged range, until it finds one that no
longer overlaps (its start exceeds the current merged end).

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Three explicit phases (Approach 1) | Clearest to read and explain — matches the problem's natural three-part structure |
| Single pass with `when` (Approach 2) | More compact, handles the "insert wherever it belongs" logic in one unified loop |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — single linear pass through the sorted intervals |
| **Space** | O(n) — the result list |

---

## Key Takeaway

> Insert Interval establishes the foundational three-phase pattern for
> the whole Intervals phase: process everything strictly before the
> region of interest, merge everything overlapping it, then process
> everything strictly after. Because the input is sorted, this requires
> only a single linear pass — no need to re-sort or use nested loops.
> This exact "before / overlapping / after" decomposition reappears,
> lightly adapted, in Merge Intervals and several other problems in this
> phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/insert-interval/)

---

{% include kotlin-dsa-nav.html %}
