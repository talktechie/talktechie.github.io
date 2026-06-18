---
title: "Stack: Daily Temperatures — Kotlin Solution"
date: 2026-06-17
categories: [Kotlin DSA, Stack]
tags: [daily-temperatures, stack, monotonic-stack, array, medium, leetcode, leetcode-739]
leetcode_number: 739
leetcode_url: https://leetcode.com/problems/daily-temperatures/
difficulty: Medium
permalink: /posts/kotlin-dsa-stack-daily-temperatures/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [739 — Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) |
| **Difficulty** | Medium |
| **Topic** | Stack, Monotonic Stack, Array |

> Given an array `temperatures`, return an array `answer` such that
> `answer[i]` is the number of days you'd have to wait after day `i` to
> get a warmer temperature. If there's no future day with a warmer
> temperature, set `answer[i] = 0`.

**Example:**
```
Input:  temperatures = [73,74,75,71,69,72,76,73]
Output: [1,1,4,2,1,1,0,0]

Input:  temperatures = [30,40,50,60]
Output: [1,1,1,0]

Input:  temperatures = [30,60,90]
Output: [1,1,0]
```

**Constraints:**
- `1 <= temperatures.length <= 10⁵`
- `30 <= temperatures[i] <= 100`

---

## Approach

The brute force — for each day, scan forward until a warmer day appears —
is O(n²). With a **monotonic stack**, we can do it in O(n).

**Key insight:** Maintain a stack of indices whose temperatures are
**decreasing** (or equal) from bottom to top. When the current day's
temperature is warmer than the temperature at the index on top of the stack,
that's the answer for that index — pop it, record the day difference, and
keep checking against the new top. Push the current index once nothing left
in the stack is beaten.

**Walk through `[73,74,75,71,69,72,76,73]`:**
```
i=0 (73) → stack empty → push → stack: [0]
i=1 (74) → 74 > temps[0]=73 → pop 0, answer[0]=1-0=1 → stack empty → push → stack: [1]
i=2 (75) → 75 > temps[1]=74 → pop 1, answer[1]=2-1=1 → stack empty → push → stack: [2]
i=3 (71) → 71 < temps[2]=75 → push → stack: [2, 3]
i=4 (69) → 69 < temps[3]=71 → push → stack: [2, 3, 4]
i=5 (72) → 72 > temps[4]=69 → pop 4, answer[4]=5-4=1
         → 72 > temps[3]=71 → pop 3, answer[3]=5-3=2
         → 72 < temps[2]=75 → stop popping → push → stack: [2, 5]
i=6 (76) → 76 > temps[5]=72 → pop 5, answer[5]=6-5=1
         → 76 > temps[2]=75 → pop 2, answer[2]=6-2=4
         → stack empty → push → stack: [6]
i=7 (73) → 73 < temps[6]=76 → push → stack: [6, 7]

End → indices remaining in stack (6, 7) never found a warmer day → answer stays 0

Result: [1,1,4,2,1,1,0,0] ✓
```

---

## Kotlin Solution

### Approach 1 — Monotonic stack of indices (optimal)

```kotlin
fun dailyTemperatures(temperatures: IntArray): IntArray {
    val answer = IntArray(temperatures.size)
    val stack = ArrayDeque<Int>()  // stores indices, decreasing temps bottom→top

    for (i in temperatures.indices) {
        while (stack.isNotEmpty() && temperatures[i] > temperatures[stack.last()]) {
            val prevIndex = stack.removeLast()
            answer[prevIndex] = i - prevIndex
        }
        stack.addLast(i)
    }

    return answer
}
```

### Approach 2 — Brute force (O(n²), for comparison only)

```kotlin
fun dailyTemperatures(temperatures: IntArray): IntArray {
    val answer = IntArray(temperatures.size)

    for (i in temperatures.indices) {
        for (j in i + 1..temperatures.lastIndex) {
            if (temperatures[j] > temperatures[i]) {
                answer[i] = j - i
                break
            }
        }
    }

    return answer
}
```

Correct but O(n²) — included only to show the naive approach the monotonic
stack improves on.

---

## Why a Monotonic Stack Gives O(n)

Each index is pushed exactly once and popped at most once — that's the
amortized O(n) guarantee, even though there's a `while` loop inside the `for`.

```
Total push operations: n (one per index)
Total pop operations:  at most n (each index popped at most once)
→ Total work across the entire run: O(n), not O(n²)
```

**Stack stores indices, not temperatures** — this is what lets us compute
the day difference:
```kotlin
while (stack.isNotEmpty() && temperatures[i] > temperatures[stack.last()]) {
    val prevIndex = stack.removeLast()
    answer[prevIndex] = i - prevIndex   // need the index to compute "days waited"
}
```

**The stack is monotonically decreasing** — every element remaining on the
stack has a temperature ≥ the one below it was when pushed. The `while` loop
maintains this invariant by clearing out anything smaller before pushing `i`.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Monotonic stack (Approach 1) | Always — O(n), this is the intended solution |
| Brute force (Approach 2) | Understanding the problem only; TLEs on large inputs |

---

## Complexity

| | Monotonic Stack | Brute Force |
|---|---|---|
| **Time** | O(n) amortized | O(n²) |
| **Space** | O(n) | O(1) extra |

---

## Key Takeaway

> "Next greater element" problems are the classic monotonic stack pattern.
> Keep a stack of indices in decreasing temperature order. Whenever a new
> value beats the top, that's the answer for the popped index — keep
> popping while the new value keeps winning. Every index is pushed once
> and popped at most once, giving O(n) despite the nested loop. This exact
> pattern reappears in Car Fleet and Largest Rectangle in Histogram.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/daily-temperatures/)

---

{% include kotlin-dsa-nav.html %}
