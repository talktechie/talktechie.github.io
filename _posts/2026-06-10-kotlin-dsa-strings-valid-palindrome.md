---
title: "Strings: Valid Palindrome — Kotlin Solution"
date: 2026-06-10
categories: [Kotlin DSA, Strings]
tags: [valid-palindrome, two-pointer, strings, easy, leetcode, leetcode-125]
leetcode_number: 125
leetcode_url: https://leetcode.com/problems/valid-palindrome/
difficulty: Easy
permalink: /posts/kotlin-dsa-strings-valid-palindrome/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [125 — Valid Palindrome](https://leetcode.com/problems/valid-palindrome/) |
| **Difficulty** | Easy |
| **Topic** | Strings, Two Pointers |

> A phrase is a palindrome if, after converting all uppercase letters to lowercase
> and removing all non-alphanumeric characters, it reads the same forward and backward.
>
> Given a string `s`, return `true` if it is a palindrome, or `false` otherwise.

**Example:**
```
Input:  s = "A man, a plan, a canal: Panama"
Output: true   // "amanaplanacanalpanama"

Input:  s = "race a car"
Output: false  // "raceacar"
```

---

## Approach

Two steps:
1. Clean the string — keep only alphanumeric characters, lowercase everything
2. Check if it reads the same forward and backward

**Approach 1 — Clean then reverse:**
Filter, then compare with reversed version.

**Approach 2 — Two pointers (O(1) space):**
Left pointer starts at beginning, right at end.
Skip non-alphanumeric characters, compare characters at both pointers.
If any mismatch — not a palindrome.

---

## Kotlin Solution

### Approach 1 — Clean and Reverse (simple)

```kotlin
fun isPalindrome(s: String): Boolean {
    val cleaned = s.filter { it.isLetterOrDigit() }.lowercase()
    return cleaned == cleaned.reversed()
}
```

### Approach 2 — Two Pointers (optimal)

```kotlin
fun isPalindrome(s: String): Boolean {
    var left = 0
    var right = s.length - 1

    while (left < right) {
        while (left < right && !s[left].isLetterOrDigit()) left++
        while (left < right && !s[right].isLetterOrDigit()) right--

        if (s[left].lowercaseChar() != s[right].lowercaseChar()) return false

        left++
        right--
    }

    return true
}
```

---

## Why Kotlin Shines Here

**`filter { it.isLetterOrDigit() }`** — built-in alphanumeric check,
no regex needed:
```kotlin
s.filter { it.isLetterOrDigit() }.lowercase()
// vs Java: s.replaceAll("[^a-zA-Z0-9]", "").toLowerCase()
```

**`.reversed()`** — string reversal as a direct extension function:
```kotlin
cleaned == cleaned.reversed()
// vs Java: cleaned.equals(new StringBuilder(cleaned).reverse().toString())
```

**`lowercaseChar()`** — per-character lowercase without converting
the whole string:
```kotlin
s[left].lowercaseChar() != s[right].lowercaseChar()
// vs Java: Character.toLowerCase(s.charAt(left))
```

---

## Two Approaches Compared

| Approach | Time | Space | Notes |
|---|---|---|---|
| Clean + reverse | O(n) | O(n) | Most readable — Kotlin one-liner |
| Two pointers | O(n) | O(1) | No extra string allocation |

Both are O(n) time. The **two pointer** approach is preferred in interviews
for O(1) space. The **clean + reverse** approach is what you'd write in
production Kotlin for readability.

---

## Complexity (Two Pointers)

| | |
|---|---|
| **Time** | O(n) — each character visited at most once |
| **Space** | O(1) — no extra string created |

---

## Key Takeaway

> Skip non-alphanumeric, compare from both ends toward the middle.
> Kotlin's `isLetterOrDigit()` and `lowercaseChar()` make this
> cleaner than Java without any regex.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/valid-palindrome/)

---

{% include kotlin-dsa-nav.html %}
