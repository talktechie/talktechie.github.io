---
title: "Backtracking: Palindrome Partitioning — Kotlin Solution"
date: 2026-06-24
categories: [Kotlin DSA, Backtracking]
tags: [palindrome-partitioning, backtracking, string, dynamic-programming, medium, leetcode, leetcode-131]
leetcode_number: 131
leetcode_url: https://leetcode.com/problems/palindrome-partitioning/
difficulty: Medium
permalink: /posts/kotlin-dsa-backtracking-palindrome-partitioning/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [131 — Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/) |
| **Difficulty** | Medium |
| **Topic** | Backtracking, String, Dynamic Programming |

> Given a string `s`, partition it such that **every substring** of the
> partition is a palindrome. Return all possible palindrome partitionings.

**Example:**
```
Input:  s = "aab"
Output: [["a","a","b"],["aa","b"]]
Explanation: both partitions split "aab" entirely into palindromic
             pieces — "a"+"a"+"b" and "aa"+"b"

Input:  s = "a"
Output: [["a"]]
```

**Constraints:**
- `1 <= s.length <= 16`
- `s` consists of only lowercase English letters

---

## Approach

**Key insight:** At each position in the string, try every possible
**next cut point** — for each candidate substring starting at the current
position, check if it's a palindrome. If it is, recurse on the remainder
of the string; if not, skip that cut point entirely (pruning the branch
immediately, since no valid partition can use a non-palindromic piece).

**Walk through `s = "aab"`:**
```
backtrack(start=0, current=[]):
  try substring s[0..0]="a" → palindrome? yes → current=["a"]
    backtrack(start=1, current=["a"]):
      try s[1..1]="a" → palindrome? yes → current=["a","a"]
        backtrack(start=2, current=["a","a"]):
          try s[2..2]="b" → palindrome? yes → current=["a","a","b"]
            backtrack(start=3): start==length → record ["a","a","b"] ✓
      try s[1..2]="ab" → palindrome? NO ("ab" reversed is "ba") → skip
  try s[0..1]="aa" → palindrome? yes → current=["aa"]
    backtrack(start=2, current=["aa"]):
      try s[2..2]="b" → palindrome? yes → current=["aa","b"]
        backtrack(start=3): record ["aa","b"] ✓

Result: [["a","a","b"],["aa","b"]] ✓
```

---

## Kotlin Solution

### Approach 1 — Backtracking with palindrome check at each cut (straightforward)

```kotlin
fun partition(s: String): List<List<String>> {
    val result = mutableListOf<List<String>>()
    val current = mutableListOf<String>()

    fun isPalindrome(start: Int, end: Int): Boolean {
        var i = start
        var j = end
        while (i < j) {
            if (s[i] != s[j]) return false
            i++
            j--
        }
        return true
    }

    fun backtrack(start: Int) {
        if (start == s.length) {
            result.add(current.toList())
            return
        }

        for (end in start until s.length) {
            if (isPalindrome(start, end)) {
                current.add(s.substring(start, end + 1))
                backtrack(end + 1)
                current.removeAt(current.lastIndex)
            }
        }
    }

    backtrack(0)
    return result
}
```

### Approach 2 — Precompute all palindromic substrings with DP, then backtrack using the lookup table

```kotlin
fun partition(s: String): List<List<String>> {
    val n = s.length
    val isPalin = Array(n) { BooleanArray(n) }

    // Precompute: isPalin[i][j] = true if s[i..j] is a palindrome
    for (end in 0 until n) {
        for (start in end downTo 0) {
            isPalin[start][end] = s[start] == s[end] &&
                (end - start < 2 || isPalin[start + 1][end - 1])
        }
    }

    val result = mutableListOf<List<String>>()
    val current = mutableListOf<String>()

    fun backtrack(start: Int) {
        if (start == n) {
            result.add(current.toList())
            return
        }

        for (end in start until n) {
            if (isPalin[start][end]) {
                current.add(s.substring(start, end + 1))
                backtrack(end + 1)
                current.removeAt(current.lastIndex)
            }
        }
    }

    backtrack(0)
    return result
}
```

Precomputes every `isPalin[i][j]` once using dynamic programming — each
lookup during backtracking becomes O(1) instead of O(length) to
recheck. Worth the extra setup when many palindrome checks are repeated
across different recursive branches.

---

## Why Precomputing with DP Avoids Redundant Work

Without precomputation, `isPalindrome(start, end)` is called repeatedly
during backtracking — and many of the same `(start, end)` ranges get
re-checked across different branches of the recursion tree, each costing
O(length) to verify character-by-character.

The DP table builds on a simple recurrence:
```kotlin
isPalin[start][end] = s[start] == s[end] &&
    (end - start < 2 || isPalin[start + 1][end - 1])
// A substring is a palindrome if its outer characters match AND
// (it's short enough not to need an inner check, OR the inner
// substring is already known to be a palindrome)
```

This is filled in order of increasing substring length (`end` increasing,
`start` decreasing from `end`), ensuring `isPalin[start+1][end-1]` is
already computed by the time we need it.

**Why the backtracking structure itself is unchanged between both
approaches:** the actual partitioning logic — try every cut point,
recurse if valid, undo — is identical. Only the *cost* of checking
"is this substring a palindrome" changes, from O(length) per check
(Approach 1) to O(1) per check after an O(n²) precomputation (Approach 2).

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Direct palindrome check (Approach 1) | Simpler to write, fine for this problem's small constraint (length ≤ 16) |
| DP precomputation (Approach 2) | Want the asymptotically faster lookup, useful if this logic were embedded in a larger/repeated computation |

---

## Complexity

| | Approach 1 | Approach 2 |
|---|---|---|
| **Time** | O(n × 2ⁿ) for backtracking, with O(n) per palindrome check | O(n²) precomputation + O(n × 2ⁿ) backtracking with O(1) checks |
| **Space** | O(n) recursion | O(n²) for the DP table |

---

## Key Takeaway

> Palindrome Partitioning is backtracking with a **validity-gated**
> branching condition — at every cut point, only recurse if the candidate
> substring satisfies the palindrome property, pruning invalid branches
> immediately rather than generating and checking complete partitions
> afterward. This problem also demonstrates a recurring optimization
> theme: when a sub-check (like "is this a palindrome?") gets called
> repeatedly across many backtracking branches, precomputing the results
> with dynamic programming can turn an O(n) repeated check into an O(1)
> lookup, at the cost of an upfront O(n²) computation.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/palindrome-partitioning/)

---

{% include kotlin-dsa-nav.html %}
