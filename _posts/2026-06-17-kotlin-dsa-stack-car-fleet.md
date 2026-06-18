---
title: "Stack: Car Fleet — Kotlin Solution"
date: 2026-06-17
categories: [Kotlin DSA, Stack]
tags: [car-fleet, stack, monotonic-stack, sorting, array, medium, leetcode, leetcode-853]
leetcode_number: 853
leetcode_url: https://leetcode.com/problems/car-fleet/
difficulty: Medium
permalink: /posts/kotlin-dsa-stack-car-fleet/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [853 — Car Fleet](https://leetcode.com/problems/car-fleet/) |
| **Difficulty** | Medium |
| **Topic** | Stack, Monotonic Stack, Sorting, Array |

> `n` cars are heading to the same destination on a one-lane road of
> length `target`. Car `i` starts at `position[i]` with `speed[i]`.
>
> A car can never pass another car ahead of it, but it can catch up and
> drive at the same speed, forming a **fleet**. A fleet is defined as one
> or more cars driving at the same speed and same position.
>
> Return the number of fleets that will arrive at the destination.

**Example:**
```
Input:  target = 12, position = [10,8,0,5,3], speed = [2,4,1,1,3]
Output: 3
Explanation:
The cars starting at 10 (speed 2) and 8 (speed 4) become a fleet,
meeting each other at 12.
The car starting at 0 (speed 1) doesn't catch up to any other car,
so it's a fleet by itself.
The cars starting at 5 (speed 1) and 3 (speed 3) become a fleet,
meeting each other at 6.

Input:  target = 10, position = [3], speed = [3]
Output: 1

Input:  target = 100, position = [0,2,4], speed = [4,2,1]
Output: 1
```

**Constraints:**
- `n == position.length == speed.length`
- `1 <= n <= 10⁵`
- `0 <= position[i] < target <= 10⁶`
- `0 <= speed[i] <= 10⁶`
- All values of `position` are unique

---

## Approach

Convert each car's state into a single number: **the time it would take to
reach the destination if nothing blocked it**.

```
time = (target - position) / speed
```

**Key insight:** Process cars from **closest to target → farthest**. A car
catches up to and merges with the fleet ahead of it if and only if its own
arrival time is **less than or equal to** the time of the car (or fleet)
directly ahead. If it would arrive later, it forms a new, separate fleet.

This is the same "compare against what's ahead, merge or start new" logic as
a monotonic stack — except here we track time instead of height.

**Walk through `target=12, position=[10,8,0,5,3], speed=[2,4,1,1,3]`:**
```
Compute time = (target - position) / speed for each car:
  pos=10 speed=2 → time=(12-10)/2=1.0
  pos=8  speed=4 → time=(12-8)/4=1.0
  pos=0  speed=1 → time=(12-0)/1=12.0
  pos=5  speed=1 → time=(12-5)/1=7.0
  pos=3  speed=3 → time=(12-3)/3=3.0

Sort by position descending (closest to target first):
  pos=10 time=1.0
  pos=8  time=1.0
  pos=5  time=7.0
  pos=3  time=3.0
  pos=0  time=12.0

Process in this order, stack holds fleet times:
  pos=10, t=1.0 → stack empty → push → stack: [1.0]
  pos=8,  t=1.0 → 1.0 <= stack.top=1.0 → merges, don't push new fleet
  pos=5,  t=7.0 → 7.0 >  stack.top=1.0 → new fleet → push → stack: [1.0, 7.0]
  pos=3,  t=3.0 → 3.0 <= stack.top=7.0 → merges, don't push new fleet
  pos=0,  t=12.0 → 12.0 > stack.top=7.0 → new fleet → push → stack: [1.0, 7.0, 12.0]

Fleets = stack.size = 3 ✓
```

---

## Kotlin Solution

### Approach 1 — Sort by position + monotonic stack (optimal)

```kotlin
fun carFleet(target: Int, position: IntArray, speed: IntArray): Int {
    val n = position.size
    // Pair each car's position with its time-to-target, sorted by position descending
    val cars = position.indices
        .map { i -> position[i] to (target - position[i]).toDouble() / speed[i] }
        .sortedByDescending { it.first }

    val stack = ArrayDeque<Double>()

    for ((_, time) in cars) {
        // A new fleet forms only if this car is slower (arrives later)
        // than the fleet immediately ahead of it
        if (stack.isEmpty() || time > stack.last()) {
            stack.addLast(time)
        }
        // else: this car catches up to the fleet ahead — merges, no push
    }

    return stack.size
}
```

### Approach 2 — Sort indices, no Pair allocation

```kotlin
fun carFleet(target: Int, position: IntArray, speed: IntArray): Int {
    val n = position.size
    val indices = (0 until n).sortedByDescending { position[it] }

    var fleets = 0
    var lastTime = 0.0

    for (i in indices) {
        val time = (target - position[i]).toDouble() / speed[i]
        if (time > lastTime) {
            fleets++
            lastTime = time
        }
    }

    return fleets
}
```

Since we only ever compare against the **most recent** fleet's time (the
stack's top), we don't need an actual stack — just one variable tracking
the last fleet-forming time. This works because cars are processed in
position order, so "top of stack" is always "the previous fleet we created."

---

## Why We Don't Need a Real Stack Here

This is a subtle but important realization. In Daily Temperatures, the stack
needed to hold multiple unresolved indices because future days could pop
several of them at once. Here, once a car merges into a fleet, it's done —
we never need to "look back" at fleets older than the most recent one,
because position order guarantees monotonic processing.

```kotlin
if (time > lastTime) {
    fleets++
    lastTime = time   // this car becomes the new "leader" to compare against
}
// A car either merges into lastTime's fleet (time <= lastTime)
// or starts a new fleet and becomes the new lastTime
```

**Sorting by position descending** is essential — we must evaluate cars
closest to the destination first, since they can never be blocked by a car
behind them:
```kotlin
val indices = (0 until n).sortedByDescending { position[it] }
```

**Using `Double` for time** avoids the truncation Integer division would
introduce — fractional time differences matter for correctness:
```kotlin
val time = (target - position[i]).toDouble() / speed[i]
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Explicit stack (Approach 1) | Want to mirror the monotonic-stack mental model exactly |
| Single variable (Approach 2) | Cleaner code once you see only the top ever matters |

---

## Complexity

| | |
|---|---|
| **Time** | O(n log n) — dominated by the sort |
| **Space** | O(n) — storing sorted indices/times |

---

## Key Takeaway

> Convert physical positions and speeds into a single comparable quantity —
> time to reach the destination. Sort by starting position (closest to
> target first), then walk through: a car merges into the fleet ahead if it
> would arrive no later than that fleet; otherwise it starts a new one.
> Because of position-ordered processing, we only ever need to remember the
> most recent fleet's time — no need for a full stack data structure, even
> though the reasoning is identical to a monotonic stack.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/car-fleet/)

---

{% include kotlin-dsa-nav.html %}
