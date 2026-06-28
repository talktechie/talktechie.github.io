---
title: "Greedy: Jump Game — Kotlin Solution"
date: 2026-06-28
categories: [Kotlin DSA, Greedy]
tags: [jump-game, greedy, dynamic-programming, array, medium, leetcode, leetcode-55]
leetcode_number: 55
leetcode_url: https://leetcode.com/problems/jump-game/
difficulty: Medium
permalink: /posts/kotlin-dsa-greedy-jump-game/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [55 — Jump Game](https://leetcode.com/problems/jump-game/) |
| **Difficulty** | Medium |
| **Topic** | Greedy, Dynamic Programming, Array |

> Given an array `nums` where `nums[i]` is the maximum jump length from
> position `i`, return `true` if you can reach the **last index**
> starting from index `0`.

**Example:**
```
Input:  nums = [2,3,1,1,4]
Output: true
Explanation: jump 1 step from index 0 to 1, then 3 steps to the last index

Input:  nums = [3,2,1,0,4]
Output: false
Explanation: you always arrive at index 3 with nums[3]=0, stuck — index 4
             is unreachable
```

**Constraints:**
- `1 <= nums.length <= 10⁴`
- `0 <= nums[i] <= 10⁵`

---

## Approach

**Key insight — track the farthest reachable index seen so far, scanning
left to right:** rather than simulating every possible jump sequence
(which branches combinatorially), greedily track the single best metric
that matters: `maxReach`, the farthest index reachable from everything
processed so far. If at any point our current position `i` **exceeds**
`maxReach`, we're stuck — no jump from any earlier position can get us
this far. Otherwise, update `maxReach` using the current position's jump
range, and keep scanning.

**Walk through `nums = [3,2,1,0,4]`:**
```
maxReach = 0

i=0: is i(0) > maxReach(0)? NO → update maxReach = max(0, 0+3) = 3
i=1: is i(1) > maxReach(3)? NO → update maxReach = max(3, 1+2) = 3
i=2: is i(2) > maxReach(3)? NO → update maxReach = max(3, 2+1) = 3
i=3: is i(3) > maxReach(3)? NO → update maxReach = max(3, 3+0) = 3
i=4: is i(4) > maxReach(3)? YES → STUCK → return false ✓
```

---

## Kotlin Solution

### Approach 1 — Greedy, track farthest reachable index (optimal)

```kotlin
fun canJump(nums: IntArray): Boolean {
    var maxReach = 0

    for (i in nums.indices) {
        if (i > maxReach) return false   // can't even reach this position

        maxReach = maxOf(maxReach, i + nums[i])
    }

    return true
}
```

### Approach 2 — DP, working backward from the end ("good index" tracking)

```kotlin
fun canJump(nums: IntArray): Boolean {
    var lastGoodIndex = nums.lastIndex

    for (i in nums.lastIndex - 1 downTo 0) {
        if (i + nums[i] >= lastGoodIndex) {
            lastGoodIndex = i   // this index can reach a known-good index
        }
    }

    return lastGoodIndex == 0
}
```

Works backward: starting from the assumption that the last index is
trivially "good" (you're already there), check if each earlier index can
jump far enough to reach a position already confirmed good. If index 0
ends up confirmed good, the whole array is traversable.

---

## Why Checking `i > maxReach` Is the Precise Failure Condition

This check is what makes the greedy approach correct without needing to
explore every possible path:

```kotlin
if (i > maxReach) return false
```

`maxReach` represents the absolute farthest position reachable using
**any** combination of jumps from positions `0` through `i-1`. If our
scan reaches position `i` and `i` itself exceeds that farthest reachable
point, it means **no** sequence of earlier jumps could have gotten us
here — the array is fundamentally unreachable beyond `maxReach`, and no
amount of further scanning can fix that.

**Why we don't need to track *which* jumps were taken, only
`maxReach`:** the question is purely "can we reach the end," not "what's
the path" — so the single aggregate value `maxReach` is sufficient
information. This is a hallmark of greedy algorithms: they discard
unnecessary state, keeping only the minimal summary needed to make
future decisions correctly.

**Why Approach 2's backward scan reaches the same conclusion:** instead
of asking "how far can I reach going forward," it asks "can I reach a
position I've already confirmed is good." Both are different lenses on
the same underlying reachability structure — forward-greedy tracks an
expanding frontier, backward-DP tracks a shrinking "known good" boundary.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Forward greedy (Approach 1) | Always — simplest, most direct, O(1) extra space |
| Backward DP (Approach 2) | Interesting alternative framing, useful for building intuition toward Jump Game II next |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — single pass, either direction |
| **Space** | O(1) |

---

## Key Takeaway

> Jump Game is solved by tracking a single greedy quantity — the
> farthest reachable index — rather than exploring the combinatorial
> space of all possible jump sequences. The moment your current position
> exceeds that frontier, the answer is definitively `false`; there's no
> need for backtracking or branching exploration. This "track the
> farthest frontier reached so far" idea, while sufficient for a simple
> yes/no reachability question here, sets up directly for Jump Game II's
> harder follow-up: not just *whether* you can reach the end, but the
> *minimum number of jumps* required to do so.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/jump-game/)

---

{% include kotlin-dsa-nav.html %}
