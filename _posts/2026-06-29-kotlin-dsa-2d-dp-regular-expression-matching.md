---
title: "2-D DP: Regular Expression Matching — Kotlin Solution"
date: 2026-06-29
categories: [Kotlin DSA, 2-D Dynamic Programming]
tags: [regular-expression-matching, dynamic-programming, string, recursion, hard, leetcode, leetcode-10]
leetcode_number: 10
leetcode_url: https://leetcode.com/problems/regular-expression-matching/
difficulty: Hard
permalink: /posts/kotlin-dsa-2d-dp-regular-expression-matching/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [10 — Regular Expression Matching](https://leetcode.com/problems/regular-expression-matching/) |
| **Difficulty** | Hard |
| **Topic** | Dynamic Programming, String, Recursion |

> Given a string `s` and a pattern `p` containing `.` (matches any
> single character) and `*` (matches **zero or more** of the preceding
> element), return `true` if `p` matches the **entire** `s`.

**Example:**
```
Input:  s = "aa", p = "a*"
Output: true
Explanation: 'a*' means zero or more of 'a' — matches "aa" exactly

Input:  s = "ab", p = ".*"
Output: true
Explanation: '.*' means zero or more of any character — matches anything

Input:  s = "mississippi", p = "mis*is*p*."
Output: false
```

**Constraints:**
- `0 <= s.length <= 20`
- `1 <= p.length <= 30`
- `p` contains only lowercase letters, `.`, and `*`
- `*` never appears as the first character, and never follows another `*`

---

## Approach

This closes out the 2-D DP phase with its most intricate transition
logic — the **two-string grid** template returns one final time, but
`*`'s "zero or more" semantics require careful, explicit case handling
unlike anything seen earlier in this phase.

**Key insight:** Define `dp[i][j]` as "does `s[0..i-1]` match
`p[0..j-1]`?" At each cell, the behavior branches heavily based on
`p[j-1]`:
- If `p[j-1]` is a plain character or `.`: it must match `s[i-1]`
  (directly or via wildcard), and the rest must already match —
  `dp[i][j] = dp[i-1][j-1]` (when characters align).
- If `p[j-1]` is `*`: it refers to `p[j-2]` (the character **before**
  it), and represents two possibilities **OR**ed together: (a) treat
  `p[j-2]*` as matching **zero** occurrences — skip both, check
  `dp[i][j-2]`; or (b) if `p[j-2]` matches `s[i-1]`, treat it as matching
  **one more** occurrence and check `dp[i-1][j]` (the `*` might still
  match additional repeats of the same character going forward).

**Walk through `s = "aa", p = "a*"`:**
```
      ""    a    *
  ""   T    F    T (p[1]='*' refers to p[0]='a' matching ZERO times)
  a    F    T    T (s[0]='a' matches p[0]='a' → dp[0][0]=T; ALSO
                     p[1]='*': zero-match dp[1][0]=F, OR one-more-match
                     since p[0]='a'==s[0]='a': dp[0][2]=T → overall T)
  a    F    F    T (similar one-more-match logic extends the match)

Answer: dp[2][2] = true ✓
```

---

## Kotlin Solution

### Approach 1 — Bottom-up 2-D DP table (handles all cases explicitly)

```kotlin
fun isMatch(s: String, p: String): Boolean {
    val m = s.length
    val n = p.length
    val dp = Array(m + 1) { BooleanArray(n + 1) }
    dp[0][0] = true   // empty string matches empty pattern

    // Handle patterns like "a*", "a*b*", etc. matching an empty s
    for (j in 1..n) {
        if (p[j - 1] == '*') {
            dp[0][j] = dp[0][j - 2]   // zero occurrences of p[j-2]
        }
    }

    for (i in 1..m) {
        for (j in 1..n) {
            when {
                p[j - 1] == '*' -> {
                    val zeroOccurrence = dp[i][j - 2]
                    val oneOrMoreOccurrence = (p[j - 2] == '.' || p[j - 2] == s[i - 1]) && dp[i - 1][j]
                    dp[i][j] = zeroOccurrence || oneOrMoreOccurrence
                }
                p[j - 1] == '.' || p[j - 1] == s[i - 1] -> {
                    dp[i][j] = dp[i - 1][j - 1]
                }
                else -> {
                    dp[i][j] = false
                }
            }
        }
    }

    return dp[m][n]
}
```

### Approach 2 — Top-down memoization (often easier to derive correctly first)

```kotlin
fun isMatch(s: String, p: String): Boolean {
    val memo = HashMap<Pair<Int, Int>, Boolean>()

    fun dp(i: Int, j: Int): Boolean {
        if (j == p.length) return i == s.length

        val key = i to j
        memo[key]?.let { return it }

        val firstMatch = i < s.length && (p[j] == '.' || p[j] == s[i])

        val result = if (j + 1 < p.length && p[j + 1] == '*') {
            dp(i, j + 2) || (firstMatch && dp(i + 1, j))
        } else {
            firstMatch && dp(i + 1, j + 1)
        }

        memo[key] = result
        return result
    }

    return dp(0, 0)
}
```

Often easier to get right on a first attempt, since the recursive
structure mirrors the problem statement's own phrasing more directly:
"does the rest match after skipping a `*` pair, OR does the current
character match AND does the rest match after consuming it."

---

## Why `*` Needs TWO Cases Combined with OR, Never Just One

This is the trickiest transition in the entire phase, worth dissecting
fully:

```kotlin
val zeroOccurrence = dp[i][j - 2]
val oneOrMoreOccurrence = (p[j - 2] == '.' || p[j - 2] == s[i - 1]) && dp[i - 1][j]
dp[i][j] = zeroOccurrence || oneOrMoreOccurrence
```

`p[j-2]*` is fundamentally ambiguous about **how many** times it
matches — possibly zero, possibly many. We can't commit to one
interpretation upfront, so both must be checked:

- **Zero occurrences:** simply skip the `p[j-2]*` pair entirely and
  check if the rest of the pattern (`p[0..j-3]`) already matches what
  we have so far in `s` — this is `dp[i][j-2]`.
- **One more occurrence:** if `p[j-2]` matches the **current** character
  of `s` (`s[i-1]`), we can "consume" this one character of `s` using
  this repetition of `p[j-2]*`, then check if the **remaining** string
  `s[0..i-2]` still matches against the **same** pattern position `j`
  (since `*` might match even more repeats afterward) — this is
  `dp[i-1][j]`.

**Why the empty-string row (`dp[0][j]`) needs special handling:** a
pattern like `"a*b*"` can validly match an **empty** string (zero `a`s,
zero `b`s) — but this can only be discovered by checking `*` constructs
against an empty `s`, which the general `i >= 1` loop never reaches.
The dedicated pre-loop handles this base case explicitly.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Bottom-up table (Approach 1) | Want the iterative, all-cases-explicit version |
| Top-down memoization (Approach 2) | Often more natural to derive correctly the first time, since it mirrors the problem's own "rest of the match" phrasing |

---

## Complexity

| | |
|---|---|
| **Time** | O(m × n) |
| **Space** | O(m × n) |

---

## Key Takeaway

> Regular Expression Matching closes the 2-D DP phase by combining
> everything: the two-string grid template (from LCS onward), an OR of
> multiple valid transition sources (echoing Interleaving String), and
> genuinely ambiguous repetition semantics requiring careful explicit
> case-by-case reasoning. There's no single elegant one-line recurrence
> here — getting `*` right requires methodically separating "zero
> occurrences" from "one more occurrence" and combining them with `||`.
> This problem is a fitting capstone: by this point in the phase, the
> *grid* structure itself should feel completely familiar, even as the
> *transition logic* within each cell grows as intricate as the problem
> demands.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/regular-expression-matching/)

---

{% include kotlin-dsa-nav.html %}
