---
title: "2-D DP: Unique Paths — Kotlin Solution"
date: 2026-06-29
categories: [Kotlin DSA, 2-D Dynamic Programming]
tags: [unique-paths, dynamic-programming, matrix, medium, leetcode, leetcode-62]
leetcode_number: 62
leetcode_url: https://leetcode.com/problems/unique-paths/
difficulty: Medium
permalink: /posts/kotlin-dsa-2d-dp-unique-paths/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [62 — Unique Paths](https://leetcode.com/problems/unique-paths/) |
| **Difficulty** | Medium |
| **Topic** | Dynamic Programming, Matrix |

> A robot starts at the top-left corner of an `m x n` grid and can only
> move **right** or **down**. Return the number of unique paths to the
> bottom-right corner.

**Example:**
```
Input:  m = 3, n = 7
Output: 28

Input:  m = 3, n = 2
Output: 3
Explanation: Right→Down→Down, Down→Right→Down, Down→Down→Right
```

**Constraints:**
- `1 <= m, n <= 100`

---

## Approach

This is the entry point to the 2-D DP phase, establishing the core
template: a 2-D table where each cell's value depends on the cells
**above** and **to the left** of it.

**Key insight:** The number of ways to reach cell `(r, c)` equals the
sum of ways to reach the cell directly above it plus the cell directly
to its left — since those are the only two places you could have moved
from. The entire first row and first column have exactly **1** way to
reach them (you can only ever move in a single straight line to get
there).

**Walk through `m = 3, n = 3`:**
```
dp table (rows × cols):
1  1  1
1  2  3
1  3  6

dp[1][1] = dp[0][1] + dp[1][0] = 1 + 1 = 2
dp[1][2] = dp[0][2] + dp[1][1] = 1 + 2 = 3
dp[2][1] = dp[1][1] + dp[2][0] = 2 + 1 = 3
dp[2][2] = dp[1][2] + dp[2][1] = 3 + 3 = 6

Answer: dp[2][2] = 6 ✓
```

---

## Kotlin Solution

### Approach 1 — Bottom-up 2-D DP table (clearest, most instructive)

```kotlin
fun uniquePaths(m: Int, n: Int): Int {
    val dp = Array(m) { IntArray(n) { 1 } }   // first row/col initialized to 1

    for (r in 1 until m) {
        for (c in 1 until n) {
            dp[r][c] = dp[r - 1][c] + dp[r][c - 1]
        }
    }

    return dp[m - 1][n - 1]
}
```

### Approach 2 — Space-optimized 1-D rolling array

```kotlin
fun uniquePaths(m: Int, n: Int): Int {
    var prevRow = IntArray(n) { 1 }

    for (r in 1 until m) {
        val currentRow = IntArray(n) { 1 }
        for (c in 1 until n) {
            currentRow[c] = currentRow[c - 1] + prevRow[c]
        }
        prevRow = currentRow
    }

    return prevRow[n - 1]
}
```

Since each row only depends on the row directly above it, we never need
to keep the full 2-D table around — just the single previous row.

---

## Why Only the Row Above and the Cell to the Left Matter

```kotlin
dp[r][c] = dp[r - 1][c] + dp[r][c - 1]
```

This is the entire recurrence, and it's worth being explicit about why
it's complete: since movement is restricted to **right** and **down**
only, the *only* two cells that could have been the robot's immediately
previous position are the one directly above (arriving via a "down"
move) and the one directly to the left (arriving via a "right" move).
Summing both counts every distinct path exactly once, since any path
reaching `(r,c)` must have taken its final step from exactly one of
those two cells.

**Why the space-optimized version only needs one previous row:**
`dp[r][c]` depends on `dp[r-1][c]` (directly above, from the *previous*
row) and `dp[r][c-1]` (to the left, from the **current** row being
built). This is why `currentRow` is built left-to-right within the same
row, while only ever referencing `prevRow` for the "row above" term —
no need to retain rows further back than that.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Full 2-D table (Approach 1) | Learning/explaining — makes the grid structure directly visible |
| Rolling 1-D array (Approach 2) | Want O(n) space instead of O(m×n) — the standard optimization for this phase |

---

## Complexity

| | 2-D Table | Rolling Array |
|---|---|---|
| **Time** | O(m × n) | O(m × n) |
| **Space** | O(m × n) | O(n) |

---

## Key Takeaway

> Unique Paths establishes the foundational 2-D DP shape: a grid where
> each cell's value is built from a small, fixed set of neighboring
> cells — here, directly above and directly to the left. This "row
> depends only on the row above it" property is what enables the
> space optimization down to a single rolling array, a technique that
> reappears throughout this entire phase whenever a 2-D recurrence only
> reaches back one row (or one column).

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/unique-paths/)

---

{% include kotlin-dsa-nav.html %}
