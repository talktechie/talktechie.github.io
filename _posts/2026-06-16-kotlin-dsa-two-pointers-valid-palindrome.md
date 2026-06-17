---
title: "Two Pointers: Valid Palindrome — Kotlin Solution"
date: 2026-06-16
categories: [Kotlin DSA, Two Pointers]
tags: [valid-palindrome, two-pointers, string, easy, leetcode, leetcode-125]
leetcode_number: 125
leetcode_url: https://leetcode.com/problems/valid-palindrome/
difficulty: Easy
permalink: /posts/kotlin-dsa-two-pointers-valid-palindrome/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [125 — Valid Palindrome](https://leetcode.com/problems/valid-palindrome/) |
| **Difficulty** | Easy |
| **Topic** | Two Pointers, String |

> A phrase is a palindrome if, after converting all uppercase letters to
> lowercase and removing all non-alphanumeric characters, it reads the
> same forward and backward.
>
> Given a string `s`, return `true` if it is a palindrome, `false` otherwise.

**Example:**
```
Input:  s = "A man, a plan, a canal: Panama"
Output: true   // "amanaplanacanalpanama" is a palindrome

Input:  s = "race a car"
Output: false  // "raceacar" is not a palindrome

Input:  s = " "
Output: true   // empty after filtering → palindrome by definition
```

**Constraints:**
- `1 <= s.length <= 2 * 10⁵`
- `s` consists only of printable ASCII characters

---

## Approach

The naive approach — filter the string, then check — works but allocates an extra string.
Two pointers lets us check in place with **O(1) space**.

**Key insight:** Place `left` at the start and `right` at the end.
Skip non-alphanumeric characters on both sides, then compare lowercase versions.
If they ever differ → not a palindrome. If the pointers meet → palindrome.

**Walk through `"A man, a plan, a canal: Panama"`:**
```
left=0  'A'  ←→  right=29 'a'  → tolower match ✓  advance both
left=1  ' '  → skip (not alphanumeric)
left=2  'm'  ←→  right=28 'm'  → match ✓
left=3  'a'  ←→  right=27 'a'  → match ✓
...continues until pointers meet...
→ true ✓
```

---

## Kotlin Solution

### Approach 1 — Two pointers, in-place (optimal)

```kotlin
fun isPalindrome(s: String): Boolean {
    var left = 0
    var right = s.lastIndex

    while (left < right) {
        while (left < right && !s[left].isLetterOrDigit())  left++
        while (left < right && !s[right].isLetterOrDigit()) right--

        if (s[left].lowercaseChar() != s[right].lowercaseChar()) return false

        left++
        right--
    }

    return true
}
```

### Approach 2 — Filter then check (readable, O(n) space)

```kotlin
fun isPalindrome(s: String): Boolean {
    val filtered = s.filter { it.isLetterOrDigit() }.lowercase()
    return filtered == filtered.reversed()
}
```

One-liner that's easy to read — at the cost of O(n) extra space for the cleaned string.

---

## Why Two Pointers Is the Preferred Answer

| | Two Pointers | Filter + Reverse |
|---|---|---|
| **Time** | O(n) | O(n) |
| **Space** | O(1) | O(n) |
| **Allocations** | Zero | Two new strings |

**`isLetterOrDigit()`** — Kotlin's built-in alphanumeric check:
```kotlin
while (!s[left].isLetterOrDigit()) left++
// Handles letters a-z, A-Z and digits 0-9 in one call
// vs Java: Character.isLetterOrDigit(s.charAt(left))
```

**`lowercaseChar()`** — compare without modifying the original:
```kotlin
if (s[left].lowercaseChar() != s[right].lowercaseChar()) return false
// No new string allocation — just Char comparison
```

**Inner while guards** — both inner loops check `left < right` to avoid crossing:
```kotlin
while (left < right && !s[left].isLetterOrDigit()) left++
// Without the guard: " " (all spaces) would overshoot and crash
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Two pointers (Approach 1) | Interview — shows awareness of space complexity |
| Filter + reverse (Approach 2) | Readability matters, space is not a concern |

---

## Complexity

| | Two Pointers | Filter + Reverse |
|---|---|---|
| **Time** | O(n) | O(n) |
| **Space** | O(1) | O(n) |

---

## Key Takeaway

> Two pointers from opposite ends — skip non-alphanumeric on both sides,
> compare lowercase. If they ever mismatch, return false. If pointers
> meet, it's a palindrome. This is the gateway problem for the two-pointer
> pattern: converging pointers, symmetric comparison, O(1) space.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/valid-palindrome/)

---

{% include kotlin-dsa-nav.html %}
