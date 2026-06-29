---
title: "2-D DP: Interleaving String — Kotlin Solution"
date: 2026-06-29
categories: [Kotlin DSA, 2-D Dynamic Programming]
tags: [interleaving-string, dynamic-programming, string, medium, leetcode, leetcode-97]
leetcode_number: 97
leetcode_url: https://leetcode.com/problems/interleaving-string/
difficulty: Medium
permalink: /posts/kotlin-dsa-2d-dp-interleaving-string/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [97 — Interleaving String](https://leetcode.com/problems/interleaving-string/) |
| **Difficulty** | Medium |
| **Topic** | Dynamic Programming, String |

> Given strings `s1`, `s2`, and `s3`, return `true` if `s3` can be
> formed by **interleaving** `s1` and `s2` (preserving each string's
> own internal character order).

**Example:**
```
Input:  s1 = "aabcc", s2 = "dbbca", s3 = "aadbbcbcac"
Output: true

Input:  s1 = "aabcc", s2 = "dbbca", s3 = "aadbbbaccc"
Output: false
```

**Constraints:**
- `0 <= s1.length, s2.length <= 100`
- `s1.length + s2.length == s3.length`
- All strings consist of lowercase English letters

---

## Approach

This directly extends Longest Common Subsequence's two-string grid
template — `dp[i][j]` again represents progress through `i` characters
of `s1` and `j` characters of `s2`, but now answers a yes/no question
about whether their **combined first `i+j` characters** can match
`s3`'s first `i+j` characters in *some* valid interleaved order.

**Key insight:** `dp[i][j]` is `true` if `s3[0..i+j-1]` can be formed by
interleaving `s1[0..i-1]` and `s2[0..j-1]`. At each cell, check **two**
possibilities: did the most recent character of `s3` come from `s1`
(requiring `s1[i-1] == s3[i+j-1]` AND `dp[i-1][j]` already true), or did
it come from `s2` (requiring `s2[j-1] == s3[i+j-1]` AND `dp[i][j-1]`
already true)?

**Walk through `s1 = "ab", s2 = "c", s3 = "abc"`:**
```
      ""    c
  ""  T     T (s2[0]='c'==s3[0]='c', dp[0][0]=T)
  a   T     F (s1[0]='a'==s3[1]='b'? NO; s2[0]='c'==s3[1]='b'? NO)

Wait, let's recheck indices carefully:
dp[0][0]=true (both empty, s3 empty too — trivial)
dp[1][0]: s1[0]='a'==s3[0]='a' AND dp[0][0]=T → dp[1][0]=T
dp[0][1]: s2[0]='c'==s3[0]='a'? NO → dp[0][1]=F
dp[1][1]: from s1: s1[0]='a'==s3[1]='b'? NO
          from s2: s2[0]='c'==s3[1]='b'? NO → dp[1][1]=F
dp[2][1]: from s1: s1[1]='b'==s3[2]='c'? NO
          from s2: s2[0]='c'==s3[2]='c' AND dp[2][0]=? 
  dp[2][0]: s1[1]='b'==s3[1]='b' AND dp[1][0]=T → dp[2][0]=T
  so dp[2][1]: s2[0]='c'==s3[2]='c' AND dp[2][0]=T → dp[2][1]=T

Answer: dp[2][1] = true ✓ ("ab"+"c" interleaved as "abc")
```

---

## Kotlin Solution

### Approach 1 — Bottom-up 2-D DP table (clearest)

```kotlin
fun isInterleave(s1: String, s2: String, s3: String): Boolean {
    val m = s1.length
    val n = s2.length
    if (m + n != s3.length) return false

    val dp = Array(m + 1) { BooleanArray(n + 1) }
    dp[0][0] = true

    for (i in 0..m) {
        for (j in 0..n) {
            if (i == 0 && j == 0) continue

            val fromS1 = i > 0 && dp[i - 1][j] && s1[i - 1] == s3[i + j - 1]
            val fromS2 = j > 0 && dp[i][j - 1] && s2[j - 1] == s3[i + j - 1]

            dp[i][j] = fromS1 || fromS2
        }
    }

    return dp[m][n]
}
```

### Approach 2 — Space-optimized, single rolling row

```kotlin
fun isInterleave(s1: String, s2: String, s3: String): Boolean {
    val m = s1.length
    val n = s2.length
    if (m + n != s3.length) return false

    var dp = BooleanArray(n + 1)
    dp[0] = true

    for (j in 1..n) {
        dp[j] = dp[j - 1] && s2[j - 1] == s3[j - 1]
    }

    for (i in 1..m) {
        dp[0] = dp[0] && s1[i - 1] == s3[i - 1]

        for (j in 1..n) {
            val fromS1 = dp[j] && s1[i - 1] == s3[i + j - 1]
            val fromS2 = dp[j - 1] && s2[j - 1] == s3[i + j - 1]
            dp[j] = fromS1 || fromS2
        }
    }

    return dp[n]
}
```

Collapses the 2-D table to a single row, since `dp[i][j]` only depends
on `dp[i-1][j]` (directly above, read **before** this row overwrites
it — hence `dp[j]` on the right side of `fromS1` still holds the
previous row's value at the start of each inner iteration) and
`dp[i][j-1]` (to the left, already updated earlier in this same row's
pass).

---

## Why TWO Conditions Are Checked at Every Cell (Unlike LCS's Single Diagonal Check)

This is the key structural difference from Longest Common Subsequence:
LCS only ever extends a **diagonal** on a match. Here, a match against
`s3` could have come from **either** string, so both possibilities must
be checked independently:

```kotlin
val fromS1 = i > 0 && dp[i - 1][j] && s1[i - 1] == s3[i + j - 1]
val fromS2 = j > 0 && dp[i][j - 1] && s2[j - 1] == s3[i + j - 1]
dp[i][j] = fromS1 || fromS2
```

`dp[i][j]` is true if **either** path leads to a valid interleaving —
we don't need to track *which* one, only that *some* valid arrangement
exists up to this point.

**Why `s3[i + j - 1]` is the correct index to compare against:** by the
time we've used `i` characters of `s1` and `j` characters of `s2`,
exactly `i + j` characters of `s3` must have been consumed — so the
**most recent** character consumed is at index `i + j - 1` (0-indexed).

**Why the space-optimized version's `dp[0]` update happens at the top of
the outer loop, before the inner loop begins:** `dp[0][i]`'s row
(conceptually, "zero characters of `s2` used") only ever depends on
`s1` matching `s3` directly, with no contribution from `s2` at all —
this single-variable update handles that edge column correctly before
the general two-source logic takes over for `j >= 1`.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Full 2-D table (Approach 1) | Learning/explaining — visualizes the two-source check clearly |
| Rolling row (Approach 2) | Want O(n) space instead of O(m×n) |

---

## Complexity

| | |
|---|---|
| **Time** | O(m × n) |
| **Space** | O(m × n) table / O(n) rolling row |

---

## Key Takeaway

> Interleaving String extends Longest Common Subsequence's two-string
> grid template by checking **two independent sources** at every cell
> instead of a single diagonal match — reflecting that each character
> of `s3` could have come from either `s1` or `s2`. This "OR together
> multiple valid transition sources" pattern, layered on top of the
> familiar two-string DP grid, is a recurring theme as problems in this
> phase grow more intricate without abandoning the core grid-based
> mental model established by LCS.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/interleaving-string/)

---

{% include kotlin-dsa-nav.html %}
