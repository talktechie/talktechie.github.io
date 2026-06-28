---
title: "1-D DP: Palindromic Substrings — Kotlin Solution"
date: 2026-06-26
categories: [Kotlin DSA, 1-D Dynamic Programming]
tags: [palindromic-substrings, dynamic-programming, string, two-pointers, medium, leetcode, leetcode-647]
leetcode_number: 647
leetcode_url: https://leetcode.com/problems/palindromic-substrings/
difficulty: Medium
permalink: /posts/kotlin-dsa-dp-palindromic-substrings/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [647 — Palindromic Substrings](https://leetcode.com/problems/palindromic-substrings/) |
| **Difficulty** | Medium |
| **Topic** | Dynamic Programming, String, Two Pointers |

> Given a string `s`, return the **number** of palindromic substrings
> (counting different positions separately, even if the substring text
> is identical).

**Example:**
```
Input:  s = "abc"
Output: 3
Explanation: "a", "b", "c" — each single character counts

Input:  s = "aaa"
Output: 6
Explanation: "a","a","a" (3 singles), "aa","aa" (2 overlapping pairs),
             "aaa" (1 triple) = 6 total
```

**Constraints:**
- `1 <= s.length <= 1000`
- `s` consists of lowercase English letters

---

## Approach

This is Longest Palindromic Substring's twin — same underlying
palindrome-checking machinery, but instead of tracking the **longest**
one found, we need to **count every single one**.

**Key insight (Expand Around Center):** Every palindromic substring has
a unique center (either a character, for odd length, or a gap between
two characters, for even length). For each of the `2n-1` possible
centers, expand outward — **every successful expansion step represents
one more valid palindrome found** (not just the longest one at that
center, but every shorter one nested within it too).

**Walk through `s = "aaa"`, center at index 1 ('a'):**
```
Odd-length check, center=1:
  l=1, r=1 → s[1]='a' → match → count++ (this is "a", the single middle character)
  expand: l=0, r=2 → s[0]='a', s[2]='a' → match → count++ (this is "aaa")
  expand: l=-1 → out of bounds → stop

This single center contributed 2 palindromes: "a" and "aaa".
(Centers at index 0 and index 2 each contribute their own single "a".
Even-length centers between 0-1 and 1-2 each contribute one "aa".)

Total across all centers: 3 (singles) + 2 (pairs) + 1 (triple) = 6 ✓
```

---

## Kotlin Solution

### Approach 1 — Expand Around Center, count every successful expansion (optimal, O(1) space)

```kotlin
fun countSubstrings(s: String): Int {
    var count = 0

    fun expandFromCenter(left: Int, right: Int) {
        var l = left
        var r = right
        while (l >= 0 && r < s.length && s[l] == s[r]) {
            count++   // every successful match is one more valid palindrome
            l--
            r++
        }
    }

    for (i in s.indices) {
        expandFromCenter(i, i)       // odd-length palindromes centered on i
        expandFromCenter(i, i + 1)   // even-length palindromes centered between i, i+1
    }

    return count
}
```

### Approach 2 — DP table, count every `true` entry

```kotlin
fun countSubstrings(s: String): Int {
    val n = s.length
    val isPalin = Array(n) { BooleanArray(n) }
    var count = 0

    for (end in 0 until n) {
        for (begin in end downTo 0) {
            if (s[begin] == s[end] && (end - begin < 2 || isPalin[begin + 1][end - 1])) {
                isPalin[begin][end] = true
                count++
            }
        }
    }

    return count
}
```

---

## Why Counting Inside the Expansion Loop (Not After) Captures Every Nested Palindrome

This is the key difference from Longest Palindromic Substring's
approach, worth being precise about:

```kotlin
while (l >= 0 && r < s.length && s[l] == s[r]) {
    count++   // increment HERE — every successful iteration is its own valid palindrome
    l--
    r++
}
```

A string like `"aaa"` doesn't just contain *one* palindrome centered at
index 1 — it contains **two**: the single middle `"a"` (the smallest
match) and the full `"aaa"` (the largest match), since every step of
expansion that successfully matches represents a distinct, valid
palindromic substring nested at that same center. Longest Palindromic
Substring only cared about the **final** (longest) successful expansion;
this problem cares about **every single one** along the way.

**Why the DP table version counts every `true` cell, not just the
diagonal or some subset:** each `isPalin[begin][end] == true` entry
represents one specific, distinct palindromic substring (defined by its
exact start and end positions) — summing all `true` entries directly
gives the total count, since the DP table inherently enumerates every
possible substring exactly once.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Expand Around Center (Approach 1) | Always — O(1) space, same time complexity as the DP version |
| DP table (Approach 2) | Want to directly reuse Longest Palindromic Substring's table-building logic |

---

## Complexity

| | Expand Around Center | DP Table |
|---|---|---|
| **Time** | O(n²) | O(n²) |
| **Space** | O(1) | O(n²) |

---

## Key Takeaway

> Palindromic Substrings and Longest Palindromic Substring share
> identical underlying machinery — the only difference is **what you do
> with each successful match**: track the longest one (previous problem)
> versus count every single one (this problem). This is a clean
> illustration of a recurring DP-phase lesson: once you've built a
> correct way to enumerate all valid substructures (palindromes here),
> many different questions about those substructures — longest, count,
> sum, etc. — are just different aggregations applied to the exact same
> underlying enumeration.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/palindromic-substrings/)

---

{% include kotlin-dsa-nav.html %}
