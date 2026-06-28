---
title: "Intervals: Meeting Rooms — Kotlin Solution"
date: 2026-06-27
categories: [Kotlin DSA, Intervals]
tags: [meeting-rooms, intervals, sorting, array, easy, leetcode]
leetcode_number: 252
leetcode_url: https://leetcode.com/problems/meeting-rooms/
difficulty: Easy
permalink: /posts/kotlin-dsa-intervals-meeting-rooms/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [252 — Meeting Rooms](https://leetcode.com/problems/meeting-rooms/) (Premium — also on NeetCode) |
| **Difficulty** | Easy |
| **Topic** | Intervals, Sorting, Array |

> Given an array of meeting time `intervals` where `intervals[i] =
> [starti, endi]`, determine if a person could attend **all** the
> meetings (i.e., no two meetings overlap).

**Example:**
```
Input:  intervals = [[0,30],[5,10],[15,20]]
Output: false
Explanation: [0,30] overlaps with both [5,10] and [15,20]

Input:  intervals = [[7,10],[2,4]]
Output: true
Explanation: no overlap — [2,4] ends before [7,10] starts
```

**Constraints:**
- `0 <= intervals.length <= 10⁴`
- `intervals[i].length == 2`
- `0 <= starti < endi <= 10⁶`

---

## Approach

This is the simplest possible interval-overlap check, and it directly
reuses Merge Intervals' "sort by start, then compare consecutive pairs"
technique — except here we just need a single `true`/`false` answer
rather than constructing a merged result.

**Key insight:** Sort the meetings by start time. If any meeting starts
**before** the previous meeting ends, there's a conflict — return
`false` immediately. If we make it through every consecutive pair
without finding an overlap, every meeting is attendable.

**Walk through `intervals = [[0,30],[5,10],[15,20]]`** (already sorted
by start):
```
compare [0,30] and [5,10]: does [5,10] start (5) before [0,30] ends (30)?
  YES (5 < 30) → CONFLICT → return false ✓
```

---

## Kotlin Solution

### Approach 1 — Sort by start, check consecutive pairs for overlap (optimal)

```kotlin
fun canAttendMeetings(intervals: Array<IntArray>): Boolean {
    if (intervals.size <= 1) return true

    intervals.sortBy { it[0] }

    for (i in 1 until intervals.size) {
        if (intervals[i][0] < intervals[i - 1][1]) {
            return false   // this meeting starts before the previous one ends
        }
    }

    return true
}
```

### Approach 2 — Separate sorted start/end arrays, compare positionally

```kotlin
fun canAttendMeetings(intervals: Array<IntArray>): Boolean {
    val starts = intervals.map { it[0] }.sorted()
    val ends = intervals.map { it[1] }.sorted()

    for (i in 1 until intervals.size) {
        if (starts[i] < ends[i - 1]) return false
    }

    return true
}
```

A clever alternative: independently sort all start times and all end
times. If the `i`-th smallest start time is before the `(i-1)`-th
smallest end time, some overlap must exist somewhere among the
meetings — even though this loses the direct correspondence between a
specific start and its own original end, the **counts** still correctly
detect any conflict.

---

## Why Sorting by Start Time Makes a Single Linear Pass Sufficient

This mirrors exactly the same reasoning as Merge Intervals: once sorted
by start time, the only way two meetings can overlap is if they're
adjacent in this sorted order (or part of a connected overlapping
chain) — there's no need to check all pairs:

```kotlin
intervals.sortBy { it[0] }
for (i in 1 until intervals.size) {
    if (intervals[i][0] < intervals[i - 1][1]) return false
}
```

If meeting `i` doesn't overlap with meeting `i-1` (the one immediately
before it in sorted order), it's guaranteed not to overlap with any
meeting even earlier than `i-1` either — those all end even sooner.

**Why Approach 2's independent sorting still works**, despite seemingly
"losing" which start belongs to which end: this is a classic interval
counting trick. At any point in time, the number of meetings that have
**started** must never exceed the number that have **ended** by *more
than one* concurrently — if the `i`-th earliest start happens before the
`(i-1)`-th earliest end, that means at least two meetings are
simultaneously "in progress" at that moment, which is exactly the
definition of a conflict, regardless of which specific intervals they
originally belonged to.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Sort by start, compare pairs (Approach 1) | Most direct and intuitive — the standard solution |
| Independent sorted starts/ends (Approach 2) | Interesting alternative; foreshadows the technique used in Meeting Rooms II |

---

## Complexity

| | |
|---|---|
| **Time** | O(n log n) — dominated by sorting |
| **Space** | O(1) extra (Approach 1) / O(n) (Approach 2, for the separate arrays) |

---

## Key Takeaway

> The simplest interval-overlap question reduces to: sort by start time,
> then check if any meeting starts before its immediate predecessor
> ends. This Easy-difficulty problem is best understood as a stepping
> stone — Approach 2's trick of separating start times and end times
> into independently sorted arrays previews the **exact** core technique
> used to solve Meeting Rooms II (counting concurrent meetings), making
> this an essential warm-up before tackling that harder follow-up
> problem next in this phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/meeting-rooms/)

---

{% include kotlin-dsa-nav.html %}
