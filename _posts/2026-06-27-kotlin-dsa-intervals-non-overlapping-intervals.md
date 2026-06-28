---
title: "Intervals: Non Overlapping Intervals — Kotlin Solution"
date: 2026-06-27
categories: [Kotlin DSA, Intervals]
tags: [non-overlapping-intervals, intervals, greedy, array, sorting, medium, leetcode, leetcode-435]
leetcode_number: 435
leetcode_url: https://leetcode.com/problems/non-overlapping-intervals/
difficulty: Medium
permalink: /posts/kotlin-dsa-intervals-non-overlapping-intervals/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [435 — Non Overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) |
| **Difficulty** | Medium |
| **Topic** | Intervals, Greedy, Array, Sorting |

> Given an array of `intervals`, return the **minimum number of
> intervals** you need to remove so that the rest are non-overlapping.

**Example:**
```
Input:  intervals = [[1,2],[2,3],[3,4],[1,3]]
Output: 1
Explanation: remove [1,3] — the rest ([1,2],[2,3],[3,4]) don't overlap
             (touching endpoints are fine)

Input:  intervals = [[1,2],[1,2],[1,2]]
Output: 2
Explanation: keep just one [1,2], remove the other two duplicates
```

**Constraints:**
- `1 <= intervals.length <= 10⁵`
- `intervals[i].length == 2`
- `-5 * 10⁴ <= starti < endi <= 5 * 10⁴`

---

## Approach

This is a classic **greedy interval scheduling** problem — the same
family as the famous "activity selection" problem. The key realization:
minimizing *removals* is equivalent to **maximizing** the number of
non-overlapping intervals we *keep*.

**Key insight — sort by END time, then greedily keep whichever interval
ends earliest:** sorting by end time (not start time, unlike Merge
Intervals) ensures that at every step, the interval we're considering
keeping leaves as much "room" as possible for future intervals. Whenever
the next interval's start is `<` the last **kept** interval's end, it
overlaps — we must remove one of them, and since we've already committed
to keeping the one that ends earlier (greedy choice), we remove the new
one instead.

**Walk through `intervals = [[1,2],[2,3],[3,4],[1,3]]`, sorted by end
time → `[[1,2],[2,3],[1,3],[3,4]]`:**
```
keep [1,2]: lastEnd = 2

[2,3]: start(2) < lastEnd(2)? NO (2 is not < 2) → doesn't overlap → KEEP
  lastEnd = 3

[1,3]: start(1) < lastEnd(3)? YES → overlaps → REMOVE this one (count++)
  lastEnd stays 3 (we keep the previously-kept [2,3], which ends earlier)

[3,4]: start(3) < lastEnd(3)? NO → doesn't overlap → KEEP
  lastEnd = 4

Total removed: 1 ✓
```

---

## Kotlin Solution

### Approach 1 — Sort by end time, greedy keep-or-remove (optimal)

```kotlin
fun eraseOverlapIntervals(intervals: Array<IntArray>): Int {
    if (intervals.isEmpty()) return 0

    intervals.sortBy { it[1] }   // sort by END time — the key greedy choice

    var removals = 0
    var lastEnd = intervals[0][1]

    for (i in 1 until intervals.size) {
        if (intervals[i][0] < lastEnd) {
            removals++   // this interval overlaps — it must be removed
            // lastEnd stays the same — we keep the previously-kept interval,
            // since it ends earlier (and thus leaves more room for the future)
        } else {
            lastEnd = intervals[i][1]   // no overlap — keep this one, advance lastEnd
        }
    }

    return removals
}
```

### Approach 2 — Equivalent framing: count the maximum non-overlapping set, subtract from total

```kotlin
fun eraseOverlapIntervals(intervals: Array<IntArray>): Int {
    if (intervals.isEmpty()) return 0

    intervals.sortBy { it[1] }

    var keptCount = 1
    var lastEnd = intervals[0][1]

    for (i in 1 until intervals.size) {
        if (intervals[i][0] >= lastEnd) {
            keptCount++
            lastEnd = intervals[i][1]
        }
    }

    return intervals.size - keptCount
}
```

Computes the answer from the opposite direction — directly counting how
many intervals **can** be kept (the maximum non-overlapping subset), then
subtracting from the total to get the minimum number that must be
removed. Mathematically identical to Approach 1, just framed as "maximize
keeps" instead of "minimize removals."

---

## Why Sorting by END Time (Not Start Time) Is the Critical Greedy Choice

This is the detail that distinguishes this problem from Merge Intervals,
and it's worth understanding the greedy proof intuition:

```kotlin
intervals.sortBy { it[1] }   // NOT it[0] — this is the key difference
```

If we always prefer to keep the interval that **ends earliest** among any
group of mutually overlapping intervals, we leave the maximum possible
remaining time available for intervals that come after. Sorting by start
time wouldn't give us this guarantee — an early-starting interval could
still have a very late end time, "blocking" far more of the timeline than
necessary.

**Why `intervals[i][0] < lastEnd` (strict less-than) is the overlap
check, allowing touching intervals to coexist:**
```kotlin
if (intervals[i][0] < lastEnd) {
    removals++
}
```
If the next interval's start exactly **equals** `lastEnd` (they touch,
like `[1,2]` and `[2,3]`), that's explicitly **not** considered
overlapping per this problem's definition — confirmed by the first
example, where `[1,2]`, `[2,3]`, and `[3,4]` all coexist without removal.

**Why we never need to "undo" a greedy choice:** once we decide to keep
an interval (because it ends earliest among its overlapping group), no
future interval can retroactively make that choice wrong — any interval
ending later than the one we kept can only be *more* restrictive, never
less, so the greedy choice remains optimal at every step (this is the
core of why a greedy approach works correctly for this specific problem
structure).

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Count removals directly (Approach 1) | Most direct mapping to the question being asked |
| Count keeps, subtract (Approach 2) | Prefer thinking in terms of "maximum non-overlapping subset," a more general framing |

---

## Complexity

| | |
|---|---|
| **Time** | O(n log n) — dominated by the sort |
| **Space** | O(1) extra (excluding the sort's own space) |

---

## Key Takeaway

> This is the classic **interval scheduling maximization** problem in
> disguise: sort by end time, then greedily keep whichever interval ends
> earliest whenever a conflict arises. This greedy strategy is provably
> optimal — preferring the earliest-ending interval always leaves the
> most room for whatever comes next. The "sort by end, not start" choice
> is the single most important detail distinguishing this problem from
> earlier ones in this phase, and it's a pattern that reappears directly
> in Meeting Rooms II's greedy reasoning later in this same phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/non-overlapping-intervals/)

---

{% include kotlin-dsa-nav.html %}
