---
title: "Two Pointers: Trapping Rain Water — Kotlin Solution"
date: 2026-06-16
categories: [Kotlin DSA, Two Pointers]
tags: [trapping-rain-water, two-pointers, dynamic-programming, hard, leetcode, leetcode-42]
leetcode_number: 42
leetcode_url: https://leetcode.com/problems/trapping-rain-water/
difficulty: Hard
permalink: /posts/kotlin-dsa-two-pointers-trapping-rain-water/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [42 — Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) |
| **Difficulty** | Hard |
| **Topic** | Two Pointers, Dynamic Programming, Array |

> Given `n` non-negative integers representing an elevation map where
> the width of each bar is 1, compute how much water it can trap after raining.

**Example:**
```
Input:  height = [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6

Input:  height = [4,2,0,3,2,5]
Output: 9
```

**Constraints:**
- `n == height.length`
- `1 <= n <= 2 * 10⁴`
- `0 <= height[i] <= 10⁵`

---

## Approach

Water trapped at any position `i` = `min(maxLeft[i], maxRight[i]) - height[i]`.

It can never be negative (if both walls are shorter than the bar, no water
pools there), so we clamp at 0. The challenge is computing `maxLeft` and
`maxRight` efficiently.

**Prefix arrays approach** does it in O(n) time, O(n) space.
**Two pointers** does it in O(n) time, O(1) space — the optimal solution.

**Key insight for two pointers:** We don't need the exact `maxLeft` and `maxRight`
for every position at once. We only need the side that's currently the bottleneck.
If `maxLeft <= maxRight`, the left side determines how much water can pool at `left`
— we don't care about the exact right maximum, just that it's ≥ maxLeft. Process
left and advance. Otherwise process right and retreat.

**Walk through `[4,2,0,3,2,5]`:**
```
left=0(4) right=5(5) maxL=4 maxR=5  → maxL≤maxR → water at left = 4-4=0 → left++
left=1(2) right=5(5) maxL=4 maxR=5  → maxL≤maxR → water at left = 4-2=2 → left++
left=2(0) right=5(5) maxL=4 maxR=5  → maxL≤maxR → water at left = 4-0=4 → left++
left=3(3) right=5(5) maxL=4 maxR=5  → maxL≤maxR → water at left = 4-3=1 → left++
left=4(2) right=5(5) maxL=4 maxR=5  → maxL≤maxR → water at left = 4-2=2 → left++
→ left >= right, stop

total = 0+2+4+1+2 = 9 ✓
```

---

## Kotlin Solution

### Approach 1 — Two pointers, O(1) space (optimal)

```kotlin
fun trap(height: IntArray): Int {
    var left  = 0
    var right = height.lastIndex
    var maxLeft  = 0
    var maxRight = 0
    var water = 0

    while (left < right) {
        if (height[left] <= height[right]) {
            maxLeft = maxOf(maxLeft, height[left])
            water  += maxLeft - height[left]
            left++
        } else {
            maxRight = maxOf(maxRight, height[right])
            water   += maxRight - height[right]
            right--
        }
    }

    return water
}
```

### Approach 2 — Prefix max arrays, O(n) space (easier to reason about)

```kotlin
fun trap(height: IntArray): Int {
    val n = height.size
    val maxLeft  = IntArray(n)
    val maxRight = IntArray(n)

    maxLeft[0] = height[0]
    for (i in 1 until n) maxLeft[i] = maxOf(maxLeft[i - 1], height[i])

    maxRight[n - 1] = height[n - 1]
    for (i in n - 2 downTo 0) maxRight[i] = maxOf(maxRight[i + 1], height[i])

    var water = 0
    for (i in 0 until n) {
        water += maxOf(0, minOf(maxLeft[i], maxRight[i]) - height[i])
    }

    return water
}
```

Start here if two pointers feels unclear — the prefix arrays make the
`min(maxLeft, maxRight) - height[i]` formula explicit and easy to verify.

---

## Building Intuition: Why min(maxLeft, maxRight)?

```
height = [4,2,0,3,2,5]
           ↑
        Position 2 (height 0)

maxLeft[2]  = max(4,2,0) = 4   (tallest wall to the left)
maxRight[2] = max(0,3,2,5) = 5 (tallest wall to the right)

Water level = min(4, 5) = 4   (limited by the shorter containing wall)
Water trapped = 4 - 0 = 4 ✓
```

The water can only rise as high as the shorter of the two bounding walls —
any higher and it spills over that side.

**Two-pointer correctness** — why we can compute locally:
```kotlin
if (height[left] <= height[right]) {
    // We know maxRight >= height[right] >= height[left] >= maxLeft(so far)
    // So min(maxLeft, maxRight) == maxLeft regardless of maxRight's exact value
    maxLeft = maxOf(maxLeft, height[left])
    water  += maxLeft - height[left]   // safe — maxLeft is the bottleneck
    left++
}
```

**`downTo` in prefix array construction:**
```kotlin
for (i in n - 2 downTo 0) maxRight[i] = maxOf(maxRight[i + 1], height[i])
// Kotlin range from right to left — no need for a manual decrement loop
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Two pointers (Approach 1) | Interview optimal answer — O(1) space |
| Prefix arrays (Approach 2) | First pass, easier to derive and explain |

---

## Complexity

| | Two Pointers | Prefix Arrays |
|---|---|---|
| **Time** | O(n) | O(n) |
| **Space** | O(1) | O(n) |

---

## Key Takeaway

> Water at position `i` = `min(maxLeft, maxRight) - height[i]`.
> Prefix arrays make this explicit in O(n) space. Two pointers get to O(1)
> by observing: if `maxLeft ≤ maxRight`, the left side is the bottleneck —
> process it without knowing the exact right max. The water is determined
> by the shorter wall. Move the shorter pointer inward. Same core insight
> as Container With Most Water, but computing cumulative trapped water.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/trapping-rain-water/)

---

{% include kotlin-dsa-nav.html %}
