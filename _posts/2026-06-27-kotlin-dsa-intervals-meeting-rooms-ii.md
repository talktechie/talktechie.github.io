---
title: "Intervals: Meeting Rooms II — Kotlin Solution"
date: 2026-06-27
categories: [Kotlin DSA, Intervals]
tags: [meeting-rooms-ii, intervals, heap, two-pointers, sorting, medium, leetcode]
leetcode_number: 253
leetcode_url: https://leetcode.com/problems/meeting-rooms-ii/
difficulty: Medium
permalink: /posts/kotlin-dsa-intervals-meeting-rooms-ii/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [253 — Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/) (Premium — also on NeetCode) |
| **Difficulty** | Medium |
| **Topic** | Intervals, Heap, Two Pointers, Sorting |

> Given an array of meeting time `intervals`, return the **minimum
> number of conference rooms** required to hold all meetings without
> conflicts.

**Example:**
```
Input:  intervals = [[0,30],[5,10],[15,20]]
Output: 2
Explanation: [0,30] needs its own room; [5,10] and [15,20] can share a
             second room (one ends before the other starts)

Input:  intervals = [[7,10],[2,4]]
Output: 1
Explanation: no overlap — one room suffices for both
```

**Constraints:**
- `1 <= intervals.length <= 10⁴`
- `0 <= starti < endi <= 10⁶`

---

## Approach

Meeting Rooms only asked "can one person attend everything" — this asks
"what's the **maximum number of meetings happening simultaneously** at
any point in time," since that peak concurrency is exactly the number of
rooms needed.

**Key insight (Two Pointers on sorted starts/ends) — directly building
on Meeting Rooms' Approach 2:** sort all start times and all end times
**independently**. Walk through start times in order; for each new
meeting starting, check whether an **earlier** meeting has already
ended (by comparing against the sorted end times pointer) — if so, that
room is now free and can be reused; if not, we need an **additional**
room. Track the maximum number of rooms in use at any point.

**Walk through `intervals = [[0,30],[5,10],[15,20]]`:**
```
starts sorted: [0, 5, 15]
ends sorted:   [10, 20, 30]

startPtr=0, endPtr=0, roomsInUse=0, maxRooms=0

process start=0: is starts[0]=0 < ends[0]=10? YES → need a new room
  roomsInUse=1, maxRooms=1, startPtr=1

process start=5: is starts[1]=5 < ends[0]=10? YES → still need a new room
  (the room from meeting [0,30] hasn't freed up yet)
  roomsInUse=2, maxRooms=2, startPtr=2

process start=15: is starts[2]=15 < ends[0]=10? NO (15 >= 10) →
  a room HAS freed up (the one from whichever meeting ended at time 10)
  → reuse it: roomsInUse stays 2 (one freed, one newly occupied — net zero)
  endPtr=1, startPtr=3

All starts processed → maxRooms = 2 ✓
```

---

## Kotlin Solution

### Approach 1 — Two pointers on independently sorted starts/ends (optimal)

```kotlin
fun minMeetingRooms(intervals: Array<IntArray>): Int {
    if (intervals.isEmpty()) return 0

    val starts = intervals.map { it[0] }.sorted()
    val ends = intervals.map { it[1] }.sorted()

    var roomsInUse = 0
    var maxRooms = 0
    var startPtr = 0
    var endPtr = 0

    while (startPtr < intervals.size) {
        if (starts[startPtr] < ends[endPtr]) {
            roomsInUse++          // this meeting needs a new room
            startPtr++
        } else {
            roomsInUse--          // a room has freed up before this meeting starts
            endPtr++
        }
        maxRooms = maxOf(maxRooms, roomsInUse)
    }

    return maxRooms
}
```

### Approach 2 — Min-heap of currently occupied rooms' end times

```kotlin
fun minMeetingRooms(intervals: Array<IntArray>): Int {
    if (intervals.isEmpty()) return 0

    val sorted = intervals.sortedBy { it[0] }
    val heap = PriorityQueue<Int>()   // tracks end times of currently occupied rooms

    for (interval in sorted) {
        // If the earliest-ending room has already freed up, reuse it
        if (heap.isNotEmpty() && heap.peek() <= interval[0]) {
            heap.poll()
        }
        heap.add(interval[1])   // occupy a room (new or reused) until this meeting ends
    }

    return heap.size   // the heap's final size is the peak concurrent room usage
}
```

The min-heap always exposes the **earliest-ending** currently-occupied
room at its top — if that room has already freed up by the time the next
meeting starts, reuse it (pop, then push the new end time); otherwise, a
genuinely new room is needed (just push, growing the heap).

---

## Why Comparing `starts[startPtr] < ends[endPtr]` Determines Room Reuse

This is the heart of the two-pointer technique, and it's worth tracing
through the logic carefully:

```kotlin
if (starts[startPtr] < ends[endPtr]) {
    roomsInUse++   // a meeting is starting BEFORE the earliest currently-tracked end
    startPtr++      // — genuinely need an additional room
} else {
    roomsInUse--   // the earliest currently-tracked end has already passed —
    endPtr++         // that room is free, available for reuse
}
```

`ends[endPtr]` always represents the **earliest end time among meetings
we haven't yet "freed" a room for**. If the next meeting (by start time)
begins before that earliest end, no room has freed up yet — we need a
new one. If it begins at or after that earliest end, that room has
become available, and reusing it means our net room count doesn't
increase.

**Why the heap approach (Approach 2) achieves the same result from a
different angle:** the heap's size at any point directly represents how
many rooms are simultaneously occupied. Popping the heap when its
smallest end time has passed is logically identical to advancing
`endPtr` in the two-pointer approach — both are asking "has the
earliest-ending currently-active meeting finished yet?"

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Two pointers (Approach 1) | Slightly more efficient (no heap overhead), directly extends Meeting Rooms' technique |
| Min-heap (Approach 2) | More intuitive "simulate rooms directly" mental model, generalizes well to richer room-tracking variants |

---

## Complexity

| | Two Pointers | Min-Heap |
|---|---|---|
| **Time** | O(n log n) — dominated by sorting | O(n log n) — sorting plus heap operations |
| **Space** | O(n) — the separate sorted arrays | O(n) — the heap |

---

## Key Takeaway

> "Minimum rooms needed" is really "maximum number of meetings
> overlapping at any single instant" — and tracking that peak doesn't
> require simulating every possible moment in time. The two-pointer
> technique on independently sorted start/end arrays directly extends
> the trick previewed in Meeting Rooms, while the min-heap approach
> offers a more literal "track which rooms are occupied" simulation.
> Both converge on the same core question: at the moment a new meeting
> starts, has an earlier meeting already finished and freed up its room?

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/meeting-rooms-ii/)

---

{% include kotlin-dsa-nav.html %}
