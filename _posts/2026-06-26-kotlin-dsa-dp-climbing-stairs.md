---
title: "1-D DP: Climbing Stairs — Kotlin Solution"
date: 2026-06-26
categories: [Kotlin DSA, 1-D Dynamic Programming]
tags: [climbing-stairs, dynamic-programming, fibonacci, easy, leetcode, leetcode-70]
leetcode_number: 70
leetcode_url: https://leetcode.com/problems/climbing-stairs/
difficulty: Easy
permalink: /posts/kotlin-dsa-dp-climbing-stairs/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [70 — Climbing Stairs](https://leetcode.com/problems/climbing-stairs/) |
| **Difficulty** | Easy |
| **Topic** | Dynamic Programming, Fibonacci |

> You are climbing a staircase with `n` steps. Each move, you can climb
> either `1` or `2` steps. Return the number of **distinct ways** to
> reach the top.

**Example:**
```
Input:  n = 2
Output: 2
Explanation: 1+1, or 2

Input:  n = 3
Output: 3
Explanation: 1+1+1, 1+2, or 2+1
```

**Constraints:**
- `1 <= n <= 45`

---

## Approach

This is the entry point to the 1-D DP phase, and it's secretly the
Fibonacci sequence wearing a different costume.

**Key insight:** To reach step `n`, your **last move** was either a
single step from `n-1`, or a double step from `n-2`. So the number of
ways to reach step `n` is simply the **sum** of the ways to reach `n-1`
and `n-2` — exactly the Fibonacci recurrence: `ways(n) = ways(n-1) +
ways(n-2)`.

**Walk through `n = 5`:**
```
ways(1) = 1   (just one way: a single step)
ways(2) = 2   (1+1, or 2)
ways(3) = ways(2) + ways(1) = 2 + 1 = 3
ways(4) = ways(3) + ways(2) = 3 + 2 = 5
ways(5) = ways(4) + ways(3) = 5 + 3 = 8

Answer: 8 ✓
```

---

## Kotlin Solution

### Approach 1 — Bottom-up DP, O(1) space (optimal)

```kotlin
fun climbStairs(n: Int): Int {
    if (n <= 2) return n

    var prev2 = 1   // ways(1)
    var prev1 = 2   // ways(2)

    for (i in 3..n) {
        val current = prev1 + prev2
        prev2 = prev1
        prev1 = current
    }

    return prev1
}
```

### Approach 2 — Top-down memoization

```kotlin
fun climbStairs(n: Int): Int {
    val memo = HashMap<Int, Int>()

    fun dp(i: Int): Int {
        if (i <= 2) return i
        return memo.getOrPut(i) { dp(i - 1) + dp(i - 2) }
    }

    return dp(n)
}
```

### Approach 3 — Naive recursion (no memoization, exponential — for contrast)

```kotlin
fun climbStairs(n: Int): Int {
    if (n <= 2) return n
    return climbStairs(n - 1) + climbStairs(n - 2)
}
```

Correct, but recomputes the same subproblems exponentially many times —
`climbStairs(3)` gets called separately within both `climbStairs(4)` and
`climbStairs(5)`'s call trees, and this redundancy compounds rapidly.
Shown only to illustrate exactly what memoization (Approach 2) fixes.

---

## Why Only Two Variables Are Needed (Not a Full Array)

This is the key space optimization that turns O(n) array-based DP into
O(1):

```kotlin
var prev2 = 1   // ways(1)
var prev1 = 2   // ways(2)

for (i in 3..n) {
    val current = prev1 + prev2
    prev2 = prev1   // slide the window forward
    prev1 = current
}
```

Since `ways(i)` only ever depends on the **two** most recent values
(`ways(i-1)` and `ways(i-2)`), there's no need to store the entire
history in an array — just keep sliding two variables forward. This
"only the last k values matter" observation is one of the most common
space optimizations across 1-D DP problems.

**Why naive recursion (Approach 3) is exponential:** without
memoization, computing `climbStairs(n)` re-derives `climbStairs(n-2)`
from scratch via **two** separate branches (once directly, once through
`climbStairs(n-1)`'s own recursion) — this overlapping recomputation
doubles at every level, giving O(2ⁿ) time overall.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Bottom-up, O(1) space (Approach 1) | Always — fastest and most memory-efficient |
| Top-down memoization (Approach 2) | Want the recursive structure to read more naturally, fine with O(n) space |
| Naive recursion (Approach 3) | Never for submission — included to show the problem memoization solves |

---

## Complexity

| | Bottom-Up | Memoized | Naive |
|---|---|---|---|
| **Time** | O(n) | O(n) | O(2ⁿ) |
| **Space** | O(1) | O(n) | O(n) — recursion stack |

---

## Key Takeaway

> Climbing Stairs is Fibonacci in disguise: the answer for `n` depends
> only on the two previous answers, because the last move is always
> either a 1-step or a 2-step. Recognizing this "current state depends
> only on a small, fixed number of previous states" pattern — and that
> you only need to keep those few previous values around, not a full
> array — is the foundational skill for the entire 1-D DP phase. Nearly
> every other problem here is a variation on "what's the recurrence
> relating this state to earlier ones?"

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/climbing-stairs/)

---

{% include kotlin-dsa-nav.html %}
