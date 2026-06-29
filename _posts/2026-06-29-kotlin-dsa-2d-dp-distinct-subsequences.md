---
title: "2-D DP: Distinct Subsequences — Kotlin Solution"
date: 2026-06-29
categories: [Kotlin DSA, 2-D Dynamic Programming]
tags: [distinct-subsequences, dynamic-programming, string, hard, leetcode, leetcode-115]
leetcode_number: 115
leetcode_url: https://leetcode.com/problems/distinct-subsequences/
difficulty: Hard
permalink: /posts/kotlin-dsa-2d-dp-distinct-subsequences/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [115 — Distinct Subsequences](https://leetcode.com/problems/distinct-subsequences/) |
| **Difficulty** | Hard |
| **Topic** | Dynamic Programming, String |

> Given strings `s` and `t`, return the **number of distinct
> subsequences** of `s` that equal `t`.

**Example:**
```
Input:  s = "rabbbit", t = "rabbit"
Output: 3
Explanation: 3 different ways to pick characters from s (choosing
             different combinations of the three 'b's) that spell "rabbit"

Input:  s = "babgbag", t = "bag"
Output: 5
```

**Constraints:**
- `1 <= s.length, t.length <= 1000`
- Both strings consist of English letters

---

## Approach

This is the **counting** cousin of Longest Common Subsequence's
recurrence shape, but with an important twist: instead of "do these
characters match, extend the diagonal," we need to count **every way**
characters could be selected from `s` (in order) to spell out `t`.

**Key insight:** Define `dp[i][j]` as the number of ways the first `i`
characters of `s` can form the first `j` characters of `t`. At each
cell:
- We can always **skip** the current character of `s` (don't use it):
  contributes `dp[i-1][j]`.
- If `s[i-1] == t[j-1]`, we can **also** use this character of `s` to
  match this character of `t`: contributes `dp[i-1][j-1]`.
- Sum both contributions (not max — since both represent genuinely
  different, valid ways).

**Walk through `s = "bag", t = "bag"`** (a tiny illustrative trace):
```
      ""  b  a  g
  ""   1  0  0  0
  b    1  1  0  0
  a    1  1  1  0
  g    1  1  1  1

dp[1][1] (s[0]='b'==t[0]='b'): dp[0][0] + dp[0][1] = 1+0 = 1
dp[2][2] (s[1]='a'==t[1]='a'): dp[1][1] + dp[1][2] = 1+0 = 1
dp[3][3] (s[2]='g'==t[2]='g'): dp[2][2] + dp[2][3] = 1+0 = 1

Answer: dp[3][3] = 1 ✓ (only one way — using every character exactly once)
```

---

## Kotlin Solution

### Approach 1 — Bottom-up 2-D DP table (clearest)

```kotlin
fun numDistinct(s: String, t: String): Int {
    val m = s.length
    val n = t.length
    val dp = Array(m + 1) { LongArray(n + 1) }   // counts can exceed Int range for large inputs

    for (i in 0..m) dp[i][0] = 1L   // empty t is trivially formed exactly 1 way (use nothing)

    for (i in 1..m) {
        for (j in 1..n) {
            dp[i][j] = dp[i - 1][j]   // always allowed: skip s[i-1]

            if (s[i - 1] == t[j - 1]) {
                dp[i][j] += dp[i - 1][j - 1]   // also allowed: match s[i-1] with t[j-1]
            }
        }
    }

    return dp[m][n].toInt()
}
```

### Approach 2 — Space-optimized, single rolling row (iterate j backward)

```kotlin
fun numDistinct(s: String, t: String): Int {
    val m = s.length
    val n = t.length
    val dp = LongArray(n + 1)
    dp[0] = 1L

    for (i in 1..m) {
        for (j in n downTo 1) {
            if (s[i - 1] == t[j - 1]) {
                dp[j] += dp[j - 1]
            }
            // when characters don't match, dp[j] simply stays as-is
            // (equivalent to "always inherit dp[i-1][j]" in the 2-D version)
        }
    }

    return dp[n].toInt()
}
```

Iterates `j` **backward** within each row — this ensures `dp[j-1]` still
refers to the **previous** row's value (not yet overwritten by this
row's own update), exactly mirroring the 0/1 knapsack downward-iteration
trick from Partition Equal Subset Sum.

---

## Why We ADD the Two Contributions, Not Take the Max

This is the critical distinction from Longest Common Subsequence's
recurrence, and it's the source of most bugs when first attempting this
problem:

```kotlin
dp[i][j] = dp[i - 1][j]                              // skip s[i-1]: a valid count
if (s[i - 1] == t[j - 1]) {
    dp[i][j] += dp[i - 1][j - 1]                       // ALSO use s[i-1]: a DIFFERENT valid count
}
```

LCS only cares about **existence** of *a* longest match — taking the max
of two options makes sense there, since we just want the best single
answer. Here, we're **counting distinct ways**, and "skip this character
of `s`" and "use this character of `s` to match `t`" are two genuinely
**different, non-overlapping** sets of subsequence-selections — both
contribute separately to the total count, so they must be **summed**,
never maxed.

**Why `dp[i][0] = 1` for all `i`:** there's exactly one way to form an
**empty** target string from any prefix of `s` — simply select nothing.
This seed value correctly anchors the "skip everything" baseline that
every other count builds from.

**Why `Long` instead of `Int` for the DP values:** with strings up to
1000 characters, the number of distinct subsequence matches can grow
combinatorially large, potentially exceeding `Int`'s range — using
`Long` internally avoids silent overflow, even though the final returned
answer (per the problem's constraints) fits back into an `Int`.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Full 2-D table (Approach 1) | Learning/explaining — visualizes the sum-of-two-contributions clearly |
| Rolling row, backward iteration (Approach 2) | Want O(n) space; mirrors the same downward-iteration discipline as 0/1 knapsack problems |

---

## Complexity

| | |
|---|---|
| **Time** | O(m × n) |
| **Space** | O(m × n) table / O(n) rolling row |

---

## Key Takeaway

> Distinct Subsequences modifies Longest Common Subsequence's grid
> template with one crucial change: sum instead of max, because "skip
> this character" and "match this character" represent genuinely
> distinct counting paths, not competing options where only the better
> one matters. This sum-vs-max distinction is the dividing line between
> "count the number of ways" DP problems and "find the best single
> outcome" DP problems — recognizing which category a problem falls into
> is often the very first decision that determines the entire
> recurrence's shape.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/distinct-subsequences/)

---

{% include kotlin-dsa-nav.html %}
