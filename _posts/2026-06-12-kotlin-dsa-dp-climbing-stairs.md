---
title: "Dynamic Programming: Climbing Stairs — Kotlin Solution"
date: 2026-06-12
categories: [Kotlin DSA, Dynamic Programming]
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

> You are climbing a staircase with `n` steps. Each time you can climb 1 or 2 steps.
> How many distinct ways can you climb to the top?

**Example:**
```
Input:  n = 3
Output: 3

Ways: [1,1,1], [1,2], [2,1]
```

---

## Approach

To reach step `n`, you either:
- Came from step `n-1` (took 1 step), or
- Came from step `n-2` (took 2 steps)

So: `ways(n) = ways(n-1) + ways(n-2)`

That's the Fibonacci sequence.

```
n=1: 1 way   → [1]
n=2: 2 ways  → [1,1], [2]
n=3: 3 ways  → [1,1,1], [1,2], [2,1]
n=4: 5 ways  → ways(3) + ways(2) = 3 + 2 = 5
n=5: 8 ways  → ways(4) + ways(3) = 5 + 3 = 8
```

---

## Kotlin Solution

### Approach 1 — Bottom-up DP (array)

```kotlin
fun climbStairs(n: Int): Int {
    if (n <= 2) return n

    val dp = IntArray(n + 1)
    dp[1] = 1
    dp[2] = 2

    for (i in 3..n) {
        dp[i] = dp[i - 1] + dp[i - 2]
    }

    return dp[n]
}
```

### Approach 2 — O(1) Space (two variables)

```kotlin
fun climbStairs(n: Int): Int {
    if (n <= 2) return n

    var prev2 = 1
    var prev1 = 2

    for (i in 3..n) {
        val current = prev1 + prev2
        prev2 = prev1
        prev1 = current
    }

    return prev1
}
```

---

## Why Kotlin Shines Here

**`IntArray(n + 1)`** — zero-initialised primitive array, no boxing:
```kotlin
val dp = IntArray(n + 1)
// vs Java: int[] dp = new int[n + 1];
// Same length but Kotlin's version is idiomatic
```

**`3..n` range** — clean and readable loop:
```kotlin
for (i in 3..n)
// vs Java: for (int i = 3; i <= n; i++)
```

**Destructuring assignment pattern** — updating two variables cleanly:
```kotlin
val current = prev1 + prev2
prev2 = prev1
prev1 = current
// Clear intent — no off-by-one confusion
```

---

## Two Approaches Compared

| Approach | Time | Space | Notes |
|---|---|---|---|
| DP array | O(n) | O(n) | Easy to understand |
| Two variables | O(n) | O(1) | Optimal — interview preferred |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) |
| **Space** | O(1) with two-variable approach |

---

## Key Takeaway

> Ways to reach step n = ways to reach n-1 + ways to reach n-2.
> It's Fibonacci. You only need the last two values — no array needed.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/climbing-stairs/)

---

{% include kotlin-dsa-nav.html %}
