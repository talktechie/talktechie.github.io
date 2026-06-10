---
title: "Strings: Valid Anagram — Kotlin Solution"
date: 2026-06-10
categories: [Kotlin DSA, Strings]
tags: [valid-anagram, hashmap, strings, easy, leetcode, leetcode-242]
leetcode_number: 242
leetcode_url: https://leetcode.com/problems/valid-anagram/
difficulty: Easy
permalink: /posts/kotlin-dsa-strings-valid-anagram/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [242 — Valid Anagram](https://leetcode.com/problems/valid-anagram/) |
| **Difficulty** | Easy |
| **Topic** | Strings, HashMap |

> Given two strings `s` and `t`, return `true` if `t` is an anagram of `s`,
> and `false` otherwise.
>
> An anagram uses all the original characters exactly once — just rearranged.

**Example:**
```
Input:  s = "anagram", t = "nagaram"
Output: true

Input:  s = "rat", t = "car"
Output: false
```

---

## Approach

Two strings are anagrams if they contain the exact same characters
with the same frequency.

**Approach 1 — Sort both strings and compare:**
If they're anagrams, sorting both gives the same string.

```
"anagram" → sorted → "aaagmnr"
"nagaram" → sorted → "aaagmnr"  ✓ match
```

Simple but O(n log n).

**Approach 2 — Frequency count (O(n)):**
Count character frequencies in `s`, then decrement for each character in `t`.
If any count goes non-zero — not an anagram.

---

## Kotlin Solution

### Approach 1 — Sort (simple)

```kotlin
fun isAnagram(s: String, t: String): Boolean {
    if (s.length != t.length) return false
    return s.toCharArray().sorted() == t.toCharArray().sorted()
}
```

### Approach 2 — Frequency Map (optimal)

```kotlin
fun isAnagram(s: String, t: String): Boolean {
    if (s.length != t.length) return false

    val count = HashMap<Char, Int>()

    for (c in s) count[c] = count.getOrDefault(c, 0) + 1
    for (c in t) count[c] = count.getOrDefault(c, 0) - 1

    return count.values.all { it == 0 }
}
```

---

## Why Kotlin Shines Here

**`toCharArray().sorted()`** — sort-and-compare in one chain:
```kotlin
s.toCharArray().sorted() == t.toCharArray().sorted()
// vs Java: Arrays.sort(s.toCharArray()) — can't compare arrays with ==
// Kotlin's sorted() returns a List, so == works correctly
```

**`getOrDefault`** shorthand via Kotlin's own extension:
```kotlin
count.getOrDefault(c, 0) + 1
// or even cleaner:
count[c] = (count[c] ?: 0) + 1
```

**`all { it == 0 }`** — readable validation over all values:
```kotlin
return count.values.all { it == 0 }
// vs Java: count.values().stream().allMatch(v -> v == 0)
```

---

## Two Approaches Compared

| Approach | Time | Space | Notes |
|---|---|---|---|
| Sort | O(n log n) | O(n) | Simple, readable |
| Frequency map | O(n) | O(n) | Optimal, interview preferred |

Use the **frequency map** in interviews — it shows you understand the
O(n) solution exists. Use sort when readability matters more than performance.

---

## Complexity (Frequency Map)

| | |
|---|---|
| **Time** | O(n) — two passes through strings |
| **Space** | O(1) — at most 26 characters in the map |

---

## Key Takeaway

> Anagram = same characters, same frequency. Count up for one string,
> count down for the other. If everything cancels to zero — it's an anagram.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/valid-anagram/)

---

{% include kotlin-dsa-nav.html %}
