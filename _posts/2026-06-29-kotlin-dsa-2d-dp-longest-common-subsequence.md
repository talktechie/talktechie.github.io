---
title: "2-D DP: Longest Common Subsequence — Kotlin Solution"
date: 2026-06-29
categories: [Kotlin DSA, 2-D Dynamic Programming]
tags: [longest-common-subsequence, dynamic-programming, string, medium, leetcode, leetcode-1143]
leetcode_number: 1143
leetcode_url: https://leetcode.com/problems/longest-common-subsequence/
difficulty: Medium
permalink: /posts/kotlin-dsa-2d-dp-longest-common-subsequence/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [1143 — Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/) |
| **Difficulty** | Medium |
| **Topic** | Dynamic Programming, String |

> Given two strings `text1` and `text2`, return the length of their
> **longest common subsequence** (characters in the same relative
> order, not necessarily contiguous), or `0` if none exists.

**Example:**
```
Input:  text1 = "abcde", text2 = "ace"
Output: 3
Explanation: "ace" is a subsequence of both

Input:  text1 = "abc", text2 = "def"
Output: 0
```

**Constraints:**
- `1 <= text1.length, text2.length <= 1000`
- Lowercase English letters only

---

## Approach

This is the **canonical two-string DP** problem — the recurrence shape
that nearly every other two-string DP problem in this phase builds on
or modifies.

**Key insight:** Define `dp[i][j]` as the LCS length using the first `i`
characters of `text1` and the first `j` characters of `text2`. At each
cell:
- If `text1[i-1] == text2[j-1]` (the characters match), extend the
  diagonal: `dp[i][j] = dp[i-1][j-1] + 1`.
- Otherwise, take the best of skipping a character from either string:
  `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`.

**Walk through `text1 = "abcde", text2 = "ace"`:**
```
      ""  a  c  e
  ""   0  0  0  0
  a    0  1  1  1
  b    0  1  1  1
  c    0  1  2  2
  d    0  1  2  2
  e    0  1  2  3

dp[1][1] ('a'=='a'): dp[0][0]+1 = 1
dp[3][2] ('c'=='c', text1[2]='c', text2[1]='c'): dp[2][1]+1 = 1+1 = 2
dp[5][3] ('e'=='e'): dp[4][2]+1 = 2+1 = 3

Answer: dp[5][3] = 3 ✓
```

---

## Kotlin Solution

### Approach 1 — Bottom-up 2-D DP table (clearest)

```kotlin
fun longestCommonSubsequence(text1: String, text2: String): Int {
    val m = text1.length
    val n = text2.length
    val dp = Array(m + 1) { IntArray(n + 1) }

    for (i in 1..m) {
        for (j in 1..n) {
            dp[i][j] = if (text1[i - 1] == text2[j - 1]) {
                dp[i - 1][j - 1] + 1
            } else {
                maxOf(dp[i - 1][j], dp[i][j - 1])
            }
        }
    }

    return dp[m][n]
}
```

### Approach 2 — Space-optimized, two rolling rows

```kotlin
fun longestCommonSubsequence(text1: String, text2: String): Int {
    val m = text1.length
    val n = text2.length
    var prevRow = IntArray(n + 1)

    for (i in 1..m) {
        val currentRow = IntArray(n + 1)
        for (j in 1..n) {
            currentRow[j] = if (text1[i - 1] == text2[j - 1]) {
                prevRow[j - 1] + 1
            } else {
                maxOf(prevRow[j], currentRow[j - 1])
            }
        }
        prevRow = currentRow
    }

    return prevRow[n]
}
```

---

## Why the Extra Row/Column (`dp[m+1][n+1]`, Not `dp[m][n]`) Is Necessary

```kotlin
val dp = Array(m + 1) { IntArray(n + 1) }
```

`dp[0][j]` and `dp[i][0]` represent "the LCS using **zero** characters
from one of the strings" — which is always `0`, since an empty sequence
shares no characters with anything. This extra row and column serve as
clean base cases, avoiding special-case handling for the very first
real character comparisons (`i=1` or `j=1` can safely reference
`dp[i-1][...]` or `dp[...][j-1]` without going out of bounds).

**Why matching characters extend the *diagonal*, not the row or
column:** `dp[i-1][j-1]` represents the LCS of both strings **without**
their current characters at all — if those current characters
(`text1[i-1]` and `text2[j-1]`) match, extending that diagonal value by
1 correctly counts this newly-matched character exactly once, without
double-counting it via the row/column directions.

**Why mismatched characters take the max of row/column neighbors:** if
the current characters don't match, the best LCS up to this point must
have been achieved by ignoring **one** of the two current characters —
either "skip `text1[i-1]`" (`dp[i-1][j]`) or "skip `text2[j-1]`"
(`dp[i][j-1]`) — and we take whichever gives the longer result.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Full 2-D table (Approach 1) | Learning/explaining, or if you need to reconstruct the actual subsequence afterward |
| Rolling rows (Approach 2) | Want O(n) space instead of O(m×n) |

---

## Complexity

| | |
|---|---|
| **Time** | O(m × n) |
| **Space** | O(m × n) full table / O(n) rolling rows |

---

## Key Takeaway

> Longest Common Subsequence is the template every two-string DP problem
> in this phase echoes: a grid where `dp[i][j]` compares the `i`-th
> character of one string against the `j`-th of the other, extending a
> diagonal on a match and taking the best of "skip from string 1" or
> "skip from string 2" otherwise. Internalizing this exact recurrence —
> diagonal-on-match, max-of-neighbors-otherwise — pays off immediately in
> Edit Distance, Interleaving String, and Distinct Subsequences later in
> this phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/longest-common-subsequence/)

---

{% include kotlin-dsa-nav.html %}
