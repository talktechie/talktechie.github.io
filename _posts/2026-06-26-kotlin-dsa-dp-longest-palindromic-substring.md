---
title: "1-D DP: Longest Palindromic Substring — Kotlin Solution"
date: 2026-06-26
categories: [Kotlin DSA, 1-D Dynamic Programming]
tags: [longest-palindromic-substring, dynamic-programming, string, two-pointers, medium, leetcode, leetcode-5]
leetcode_number: 5
leetcode_url: https://leetcode.com/problems/longest-palindromic-substring/
difficulty: Medium
permalink: /posts/kotlin-dsa-dp-longest-palindromic-substring/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [5 — Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/) |
| **Difficulty** | Medium |
| **Topic** | Dynamic Programming, String, Two Pointers |

> Given a string `s`, return the **longest palindromic substring** (a
> contiguous substring that reads the same forwards and backwards).

**Example:**
```
Input:  s = "babad"
Output: "bab" (or "aba" — both are valid length-3 answers)

Input:  s = "cbbd"
Output: "bb"
```

**Constraints:**
- `1 <= s.length <= 1000`
- `s` consists of lowercase/uppercase English letters, digits

---

## Approach

The brute force — check every substring for being a palindrome — is
O(n³). We'll cover two faster techniques: **Expand Around Center**
(O(n²) time, O(1) space) and DP with a precomputed table (O(n²) time
and space, but conceptually connects directly to Palindromic Substrings,
the next problem in this phase).

**Key insight (Expand Around Center):** Every palindrome has a center —
either a single character (odd-length palindrome) or a gap between two
characters (even-length palindrome). There are exactly `2n - 1` possible
centers. For each, expand outward as far as both sides keep matching,
tracking the longest match found.

**Walk through `s = "babad"`, center at index 1 ('a'):**
```
center=1 (odd-length check): left=1, right=1 → s[1]='a' → match itself trivially
  expand: left=0, right=2 → s[0]='b', s[2]='b' → match! → "bab" (length 3)
  expand: left=-1 → out of bounds → stop

Found "bab" — compare against best-so-far, update if longer.

(Repeating this for every center eventually also finds "aba" at center=2,
also length 3 — either is an acceptable answer.)
```

---

## Kotlin Solution

### Approach 1 — Expand Around Center (optimal, O(1) space)

```kotlin
fun longestPalindrome(s: String): String {
    if (s.isEmpty()) return ""

    var start = 0
    var maxLen = 1

    fun expandFromCenter(left: Int, right: Int) {
        var l = left
        var r = right
        while (l >= 0 && r < s.length && s[l] == s[r]) {
            l--
            r++
        }
        // l, r have gone one step too far — the valid palindrome is (l+1, r-1)
        val length = r - l - 1
        if (length > maxLen) {
            maxLen = length
            start = l + 1
        }
    }

    for (i in s.indices) {
        expandFromCenter(i, i)       // odd-length palindromes, centered on i
        expandFromCenter(i, i + 1)   // even-length palindromes, centered between i and i+1
    }

    return s.substring(start, start + maxLen)
}
```

### Approach 2 — DP table, isPalin[i][j] = is s[i..j] a palindrome

```kotlin
fun longestPalindrome(s: String): String {
    val n = s.length
    val isPalin = Array(n) { BooleanArray(n) }
    var start = 0
    var maxLen = 1

    for (i in 0 until n) isPalin[i][i] = true   // single characters are always palindromes

    for (end in 0 until n) {
        for (begin in end downTo 0) {
            if (s[begin] == s[end]) {
                isPalin[begin][end] = (end - begin < 2) || isPalin[begin + 1][end - 1]

                if (isPalin[begin][end] && end - begin + 1 > maxLen) {
                    maxLen = end - begin + 1
                    start = begin
                }
            }
        }
    }

    return s.substring(start, start + maxLen)
}
```

---

## Why Both Odd AND Even Center Checks Are Necessary

```kotlin
expandFromCenter(i, i)       // odd-length: "aba" — single character center
expandFromCenter(i, i + 1)   // even-length: "abba" — center is a GAP between two characters
```

A palindrome like `"aba"` has a true middle character (index 1), while
`"abba"` has no single middle character — its center is the gap between
index 1 and 2. Calling `expandFromCenter` with `(i, i)` handles the
first case; calling it with `(i, i+1)` (starting from two *already
adjacent* positions, even before checking if they match) correctly
handles the second case. Without checking both, even-length palindromes
would be systematically missed.

**Why the DP recurrence checks `end - begin < 2` as a shortcut:**
```kotlin
isPalin[begin][end] = (end - begin < 2) || isPalin[begin + 1][end - 1]
// end - begin < 2 means the substring has length 1 or 2:
//   length 1 (begin == end): trivially a palindrome (already true from initialization)
//   length 2 (end == begin + 1): a palindrome iff the two characters match,
//     which we've ALREADY confirmed via s[begin] == s[end] in the outer if
// For length >= 3, we additionally need the INNER substring to also be a palindrome
```

**Why the DP table fills in order of increasing substring length** (`end`
increasing, `begin` decreasing from `end`): `isPalin[begin+1][end-1]`
must already be computed by the time we need it, and this fill order
guarantees that — shorter substrings are always resolved before longer
ones that depend on them.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Expand Around Center (Approach 1) | Always — same O(n²) time, but O(1) space instead of O(n²) |
| DP table (Approach 2) | Want explicit reuse across multiple queries, or building toward Palindromic Substrings next |

---

## Complexity

| | Expand Around Center | DP Table |
|---|---|---|
| **Time** | O(n²) | O(n²) |
| **Space** | O(1) | O(n²) |

---

## Key Takeaway

> Both techniques exploit the same core palindrome property: a
> substring `s[i..j]` is a palindrome if and only if its outer
> characters match AND the inner substring `s[i+1..j-1]` is also a
> palindrome. Expand Around Center applies this outward from a guessed
> center; the DP table applies it by building up from the smallest
> substrings first. The DP version is worth knowing even though it uses
> more space, because the exact same `isPalin` table — and the exact
> same recurrence — is reused directly (and more thoroughly) in
> Palindromic Substrings, the very next problem in this phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/longest-palindromic-substring/)

---

{% include kotlin-dsa-nav.html %}
