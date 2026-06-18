---
title: "Stack: Largest Rectangle In Histogram — Kotlin Solution"
date: 2026-06-17
categories: [Kotlin DSA, Stack]
tags: [largest-rectangle-in-histogram, stack, monotonic-stack, array, hard, leetcode, leetcode-84]
leetcode_number: 84
leetcode_url: https://leetcode.com/problems/largest-rectangle-in-histogram/
difficulty: Hard
permalink: /posts/kotlin-dsa-stack-largest-rectangle-in-histogram/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [84 — Largest Rectangle In Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/) |
| **Difficulty** | Hard |
| **Topic** | Stack, Monotonic Stack, Array |

> Given an array of integers `heights` representing the histogram's bar
> height where the width of each bar is 1, return the area of the largest
> rectangle that can be formed within the histogram.

**Example:**
```
Input:  heights = [2,1,5,6,2,3]
Output: 10
Explanation: The largest rectangle is formed by bars at index 2 and 3
(heights 5 and 6), using height 5 and width 2 → area = 10

Input:  heights = [2,4]
Output: 4
```

**Constraints:**
- `1 <= heights.length <= 10⁵`
- `0 <= heights[i] <= 10⁴`

---

## Approach

Brute force checks every pair of bars as left/right boundaries — O(n²).
We can do O(n) with a **monotonic increasing stack**.

**Key insight:** For each bar, the largest rectangle that uses this bar as
its **shortest** bar extends left and right until it hits a bar shorter than
itself. So: for every bar, find how far left and right it can stretch before
hitting something shorter, then `area = height × width`.

A monotonic increasing stack of `(index, height)` does this in one pass.
When we encounter a bar shorter than the stack's top, we know the top bar's
rectangle is now finalized — its right boundary is the current index, and its
left boundary is whatever's now below it in the stack.

**Walk through `[2,1,5,6,2,3]`** using indices on a monotonic increasing stack
(left boundary = new top of stack **index + 1**, right boundary = current index `i`):
```
i=0 (h=2) → stack empty → push → stack: [0]
i=1 (h=1) → heights[1]=1 < heights[0]=2 → pop 0:
            width = i - (-1) - 1 = 1 - (-1) - 1 = 1  (stack now empty)
            area = 2 * 1 = 2 → max=2
          → push 1 → stack: [1]
i=2 (h=5) → 5 > heights[1]=1 → push → stack: [1, 2]
i=3 (h=6) → 6 > heights[2]=5 → push → stack: [1, 2, 3]
i=4 (h=2) → 2 < heights[3]=6 → pop 3:
            width = i - stack.last() - 1 = 4 - 2 - 1 = 1
            area = 6 * 1 = 6 → max=6
          → 2 < heights[2]=5 → pop 2:
            width = i - stack.last() - 1 = 4 - 1 - 1 = 2
            area = 5 * 2 = 10 → max=10
          → 2 > heights[1]=1 → push 4 → stack: [1, 4]
i=5 (h=3) → 3 > heights[4]=2 → push → stack: [1, 4, 5]

End (i=6, sentinel height=0) → pop everything:
  pop 5: width = 6 - 4 - 1 = 1 → area = 3*1=3  → max stays 10
  pop 4: width = 6 - 1 - 1 = 4 → area = 2*4=8  → max stays 10
  pop 1: width = 6                → area = 1*6=6  → max stays 10

Result: max area = 10 ✓ (the bars at index 2 and 3, heights 5 and 6,
contribute the winning rectangle: height 5 spanning width 2)
```

The left boundary after every pop must use the **new top of the stack**,
not a fixed value — that's exactly why we track the index, not just the height.

---

## Kotlin Solution

### Approach 1 — Monotonic increasing stack of indices (optimal)

```kotlin
fun largestRectangleArea(heights: IntArray): Int {
    val stack = ArrayDeque<Int>()  // indices, strictly increasing heights
    var maxArea = 0

    for (i in heights.indices) {
        while (stack.isNotEmpty() && heights[stack.last()] > heights[i]) {
            val height = heights[stack.removeLast()]
            val width = if (stack.isEmpty()) i else i - stack.last() - 1
            maxArea = maxOf(maxArea, height * width)
        }
        stack.addLast(i)
    }

    // Drain remaining bars — each extends all the way to the end
    while (stack.isNotEmpty()) {
        val height = heights[stack.removeLast()]
        val width = if (stack.isEmpty()) heights.size else heights.size - stack.last() - 1
        maxArea = maxOf(maxArea, height * width)
    }

    return maxArea
}
```

This matches the walk-through above exactly: the stack holds indices in
increasing height order, and the width formula `i - stack.last() - 1`
correctly accounts for the new left boundary after every pop.

### Approach 2 — Stack of indices with explicit boundaries

```kotlin
fun largestRectangleArea(heights: IntArray): Int {
    val stack = ArrayDeque<Int>()  // indices, increasing height
    var maxArea = 0
    val n = heights.size

    for (i in 0..n) {
        val currentHeight = if (i == n) 0 else heights[i]

        while (stack.isNotEmpty() && heights[stack.last()] >= currentHeight) {
            val height = heights[stack.removeLast()]
            val width = if (stack.isEmpty()) i else i - stack.last() - 1
            maxArea = maxOf(maxArea, height * width)
        }

        stack.addLast(i)
    }

    return maxArea
}
```

The classic textbook version — appends a sentinel `0` height at the end
(`i == n`) to force-flush every remaining bar in the stack, avoiding a
separate drain loop.

---

## Why the Width Formula Works

When we pop a bar at index `j` because a shorter bar appeared at `i`:

```
height * width

width = i - stack.last() - 1   (if stack still has elements)
width = i                       (if stack is now empty)
```

The popped bar's rectangle can extend:
- **Right** up to (but not including) `i` — `i` is where it got cut off
- **Left** up to (but not including) the new top of the stack — anything
  below the new top is shorter, so this bar can't extend past it

```
heights = [2,1,5,6,2,3], i=4 (h=2), popping index 3 (h=6):
  stack after pop: [1]   stack.last() = 1
  width = 4 - 1 - 1 = 2   → indices 2,3 → matches bars [5,6] ✓
  area = 6 * 2 = 12

Continue popping index 2 (h=5):
  stack after pop: [1]   stack.last() = 1
  width = 4 - 1 - 1 = 2   → indices 2,3 → bar height 5 spans both → area = 5*2=10 ✓
```

**Sentinel value trick** — appending a height-0 bar conceptually forces
every remaining bar to be popped and evaluated:
```kotlin
for (i in 0..n) {
    val currentHeight = if (i == n) 0 else heights[i]
    // when i == n, currentHeight = 0 guarantees everything left in the
    // stack gets popped and measured before the function returns
}
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Index stack, drain loop (Approach 1) | Clear separation between main pass and cleanup |
| Index stack + sentinel (Approach 2) | Classic textbook form, single unified loop |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) amortized — each bar pushed once, popped at most once |
| **Space** | O(n) — stack can hold all bars in the worst case (strictly increasing heights) |

---

## Key Takeaway

> Maintain a monotonic increasing stack of bars. When a shorter bar arrives,
> it finalizes the rectangle for every taller bar still on the stack — pop
> them, and for each, the width is bounded by the current index (right) and
> the new stack top (left). A sentinel zero-height bar at the very end forces
> a clean flush of anything left over. This is the most advanced application
> of the monotonic stack pattern in this series — same core idea as Daily
> Temperatures and Car Fleet, extended to track both height and width.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/largest-rectangle-in-histogram/)

---

{% include kotlin-dsa-nav.html %}
