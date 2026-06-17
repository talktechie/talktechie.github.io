---
title: "Two Pointers: Container With Most Water — Kotlin Solution"
date: 2026-06-16
categories: [Kotlin DSA, Two Pointers]
tags: [container-with-most-water, two-pointers, greedy, medium, leetcode, leetcode-11]
leetcode_number: 11
leetcode_url: https://leetcode.com/problems/container-with-most-water/
difficulty: Medium
permalink: /posts/kotlin-dsa-two-pointers-container-with-most-water/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [11 — Container With Most Water](https://leetcode.com/problems/container-with-most-water/) |
| **Difficulty** | Medium |
| **Topic** | Two Pointers, Greedy, Array |

> You are given an integer array `height` of length `n`. There are `n`
> vertical lines drawn at positions `i` and `i+1`. Find two lines that,
> together with the x-axis, form a container that holds the most water.
>
> Return the maximum amount of water the container can store.
> You may not slant the container.

**Example:**
```
Input:  height = [1,8,6,2,5,4,8,3,7]
Output: 49
// Lines at index 1 (height 8) and index 8 (height 7)
// width = 8 - 1 = 7, height = min(8,7) = 7, area = 49

Input:  height = [1,1]
Output: 1
```

**Constraints:**
- `n == height.length`
- `2 <= n <= 10⁵`
- `0 <= height[i] <= 10⁴`

---

## Approach

Area = `width × min(height[left], height[right])`.

Width shrinks as the pointers converge. The only way to potentially increase area
is to find a taller wall. So when the two pointers point at walls, we should move
the **shorter** one inward — keeping the taller one gives us the best possible
height for the next narrower container.

**Key insight:** Moving the taller wall inward can only decrease or maintain the
height (it's bounded by the shorter wall anyway), while the width also decreases.
That's guaranteed worse. So always move the shorter pointer.

**Walk through `[1,8,6,2,5,4,8,3,7]`:**
```
left=0(1)  right=8(7) → area = min(1,7)*8 = 8   → move left (shorter)
left=1(8)  right=8(7) → area = min(8,7)*7 = 49  → move right (shorter)
left=1(8)  right=7(3) → area = min(8,3)*6 = 18  → move right
left=1(8)  right=6(8) → area = min(8,8)*5 = 40  → move either (equal, move right)
left=1(8)  right=5(4) → area = min(8,4)*4 = 16  → move right
left=1(8)  right=4(5) → area = min(8,5)*3 = 15  → move right
left=1(8)  right=3(2) → area = min(8,2)*2 = 4   → move right
left=1(8)  right=2(6) → area = min(8,6)*1 = 6   → move right
→ left >= right, stop

max = 49 ✓
```

---

## Kotlin Solution

### Approach 1 — Two pointers, greedy (optimal)

```kotlin
fun maxArea(height: IntArray): Int {
    var left  = 0
    var right = height.lastIndex
    var maxWater = 0

    while (left < right) {
        val water = minOf(height[left], height[right]) * (right - left)
        maxWater = maxOf(maxWater, water)

        if (height[left] < height[right]) left++
        else right--
    }

    return maxWater
}
```

### Approach 2 — Brute force (O(n²), for comparison only)

```kotlin
fun maxArea(height: IntArray): Int {
    var maxWater = 0
    for (i in height.indices) {
        for (j in i + 1..height.lastIndex) {
            maxWater = maxOf(maxWater, minOf(height[i], height[j]) * (j - i))
        }
    }
    return maxWater
}
```

TLEs on large inputs — only shown to contrast. Two pointers eliminates the need
to check every pair.

---

## Why Moving the Shorter Pointer Is Correct

Suppose `height[left] <= height[right]`. All pairs `(left, x)` where `x < right`
have been implicitly ruled out:
- Width is smaller than `right - left`
- Height is at most `height[left]` (same or smaller constraint)
- Area can only be ≤ current area

So none of those pairs can beat what we've seen. We safely discard `left` and
move it inward. The same logic applies symmetrically when `height[right] < height[left]`.

**`minOf` and `maxOf`** — prefer over `Math.min`/`Math.max` in Kotlin:
```kotlin
val water = minOf(height[left], height[right]) * (right - left)
maxWater  = maxOf(maxWater, water)
// Kotlin top-level functions — no need to qualify with Math.
```

**When heights are equal**, moving either pointer is fine:
```kotlin
if (height[left] < height[right]) left++
else right--   // covers equal case — move right (or left, both correct)
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Two pointers (Approach 1) | Always — O(n) optimal |
| Brute force (Approach 2) | Understanding the problem only; never submit |

---

## Complexity

| | Two Pointers | Brute Force |
|---|---|---|
| **Time** | O(n) | O(n²) |
| **Space** | O(1) | O(1) |

---

## Key Takeaway

> Area = width × min height. Width only shrinks as pointers converge.
> Always move the shorter wall — keeping the taller one maximises the
> height component of every future candidate. Moving the taller wall
> can never help: height is still bounded by the shorter side and width
> just got smaller. This greedy move is the core insight.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/container-with-most-water/)

---

{% include kotlin-dsa-nav.html %}
