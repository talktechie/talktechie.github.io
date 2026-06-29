---
title: "2-D DP: Edit Distance — Kotlin Solution"
date: 2026-06-29
categories: [Kotlin DSA, 2-D Dynamic Programming]
tags: [edit-distance, dynamic-programming, string, medium, leetcode, leetcode-72]
leetcode_number: 72
leetcode_url: https://leetcode.com/problems/edit-distance/
difficulty: Medium
permalink: /posts/kotlin-dsa-2d-dp-edit-distance/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [72 — Edit Distance](https://leetcode.com/problems/edit-distance/) |
| **Difficulty** | Medium |
| **Topic** | Dynamic Programming, String |

> Given two strings `word1` and `word2`, return the **minimum number of
> operations** (insert, delete, or replace a character) to convert
> `word1` into `word2`.

**Example:**
```
Input:  word1 = "horse", word2 = "ros"
Output: 3
Explanation: horse → rorse (replace h→r) → rose (delete r) → ros
             (delete e) — 3 operations

Input:  word1 = "intention", word2 = "execution"
Output: 5
```

**Constraints:**
- `0 <= word1.length, word2.length <= 500`
- Lowercase English letters only

---

## Approach

This is the **minimization** sibling of Longest Common Subsequence's
grid — same two-string template, but instead of finding a match length,
we're finding the cheapest sequence of edits.

**Key insight:** Define `dp[i][j]` as the minimum edits to convert the
first `i` characters of `word1` into the first `j` characters of
`word2`. At each cell:
- If `word1[i-1] == word2[j-1]`, no edit needed for this character pair
  — inherit `dp[i-1][j-1]` directly.
- Otherwise, take the best (minimum) of three possible operations, each
  costing **1** plus whatever the resulting subproblem costs:
  **replace** (`dp[i-1][j-1] + 1`), **delete** from word1
  (`dp[i-1][j] + 1`), or **insert** into word1 (`dp[i][j-1] + 1`).

**Walk through `word1 = "ros", word2 = "ro"`** (a small illustrative
slice):
```
      ""  r  o
  ""   0  1  2
  r    1  0  1
  o    2  1  0
  s    3  2  1

dp[1][1] ('r'=='r'): dp[0][0] = 0
dp[2][2] ('o'=='o'): dp[1][1] = 0
dp[3][2] ('s' vs nothing left in word2, j=2 is the last valid index):
  word1[2]='s', and j=2 means we've matched "ro" already
  delete 's' from word1: dp[2][2]+1 = 0+1 = 1

Answer: dp[3][2] = 1 ✓ (just delete the trailing 's')
```

---

## Kotlin Solution

### Approach 1 — Bottom-up 2-D DP table (clearest)

```kotlin
fun minDistance(word1: String, word2: String): Int {
    val m = word1.length
    val n = word2.length
    val dp = Array(m + 1) { IntArray(n + 1) }

    for (i in 0..m) dp[i][0] = i   // delete all i characters of word1 to reach ""
    for (j in 0..n) dp[0][j] = j   // insert all j characters to build word2 from ""

    for (i in 1..m) {
        for (j in 1..n) {
            dp[i][j] = if (word1[i - 1] == word2[j - 1]) {
                dp[i - 1][j - 1]   // characters match — no edit needed here
            } else {
                1 + minOf(
                    dp[i - 1][j - 1],   // replace
                    dp[i - 1][j],       // delete from word1
                    dp[i][j - 1]        // insert into word1
                )
            }
        }
    }

    return dp[m][n]
}
```

### Approach 2 — Space-optimized, two rolling rows

```kotlin
fun minDistance(word1: String, word2: String): Int {
    val m = word1.length
    val n = word2.length
    var prevRow = IntArray(n + 1) { it }

    for (i in 1..m) {
        val currentRow = IntArray(n + 1)
        currentRow[0] = i

        for (j in 1..n) {
            currentRow[j] = if (word1[i - 1] == word2[j - 1]) {
                prevRow[j - 1]
            } else {
                1 + minOf(prevRow[j - 1], prevRow[j], currentRow[j - 1])
            }
        }

        prevRow = currentRow
    }

    return prevRow[n]
}
```

---

## Why the Base Cases Are `dp[i][0] = i` and `dp[0][j] = j`

This is worth being precise about, since it's a common off-by-one
source of bugs:

```kotlin
for (i in 0..m) dp[i][0] = i   // converting word1's first i chars to "" needs i DELETIONS
for (j in 0..n) dp[0][j] = j   // converting "" to word2's first j chars needs j INSERTIONS
```

Converting any prefix of `word1` of length `i` into an **empty** string
requires deleting all `i` characters — no other operation could be
cheaper. Symmetrically, building any prefix of `word2` of length `j`
from an **empty** starting string requires exactly `j` insertions. These
serve as the DP table's "edges," anchoring every interior cell's
recursive computation.

**Why three options are minimized (not summed, unlike Distinct
Subsequences) when characters don't match:**
```kotlin
1 + minOf(dp[i - 1][j - 1], dp[i - 1][j], dp[i][j - 1])
```
We want the **single cheapest** sequence of edits — these three options
represent mutually exclusive strategies for handling the current
character mismatch (replace one character, delete one extra character
from `word1`, or insert one needed character into `word1`), and we only
ever need to actually perform **one** of them, then continue optimally
from there. Since we're minimizing total operation count, `min` (not
`sum`) is the correct combinator — directly mirroring Min Cost Climbing
Stairs' min-based recurrence from the 1-D DP phase, just extended to
three options instead of two.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Full 2-D table (Approach 1) | Learning/explaining, or if reconstructing the actual edit sequence is needed |
| Rolling rows (Approach 2) | Want O(n) space instead of O(m×n) |

---

## Complexity

| | |
|---|---|
| **Time** | O(m × n) |
| **Space** | O(m × n) table / O(n) rolling rows |

---

## Key Takeaway

> Edit Distance is the "minimize total cost" counterpart to Longest
> Common Subsequence's "maximize match length" — same two-string grid,
> but combining three possible operations (replace, delete, insert) via
> `min` instead of extending a single diagonal via `max`. This problem
> is one of the most celebrated classical DP results (it underlies
> spell-checkers, DNA sequence alignment, and diff tools), and
> internalizing its three-way minimum recurrence rounds out the full
> toolkit of two-string DP patterns — match-and-extend (LCS), sum-of-
> sources (Distinct Subsequences), and now min-of-operations (Edit
> Distance) — covered across this phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/edit-distance/)

---

{% include kotlin-dsa-nav.html %}
