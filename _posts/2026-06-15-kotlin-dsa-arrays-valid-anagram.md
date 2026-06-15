---
title: "Arrays: Valid Anagram — Kotlin Solution"
date: 2026-06-15
categories: [Kotlin DSA, Arrays]
tags: [valid-anagram, hashmap, array, frequency-count, easy, leetcode, leetcode-242]
leetcode_number: 242
leetcode_url: https://leetcode.com/problems/valid-anagram/
difficulty: Easy
permalink: /posts/kotlin-dsa-arrays-valid-anagram/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [242 — Valid Anagram](https://leetcode.com/problems/valid-anagram/) |
| **Difficulty** | Easy |
| **Topic** | Arrays, HashMap, Frequency Count |

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

Two strings are anagrams if they have the **same characters with the same frequency**.

The array-based approach — use an `IntArray(26)` to count character frequencies.
One array, no HashMap, O(1) space for lowercase English letters.

**Key insight:** Map each character to an index using `char - 'a'`:
```
'a' - 'a' = 0
'b' - 'a' = 1
'z' - 'a' = 25
```

Increment for every char in `s`, decrement for every char in `t`.
If the array is all zeros at the end — they're anagrams.

**Walk through `s="rat", t="car"`:**
```
Process 'r': count[17]++   → [0,0,0,...,1,...,0]
Process 'a': count[0]++    → [1,0,0,...,1,...,0]
Process 't': count[19]++   → [1,0,0,...,1,...,1,0]

Process 'c': count[2]--    → [1,0,-1,...,1,...,1,0]
Process 'a': count[0]--    → [0,0,-1,...,1,...,1,0]
Process 'r': count[17]--   → [0,0,-1,...,0,...,1,0]

Not all zeros → false ✓
```

---

## Kotlin Solution

### Approach 1 — IntArray frequency count (optimal)

```kotlin
fun isAnagram(s: String, t: String): Boolean {
    if (s.length != t.length) return false

    val count = IntArray(26)

    for (i in s.indices) {
        count[s[i] - 'a']++
        count[t[i] - 'a']--
    }

    return count.all { it == 0 }
}
```

### Approach 2 — Sort and compare

```kotlin
fun isAnagram(s: String, t: String): Boolean {
    if (s.length != t.length) return false
    return s.toCharArray().sorted() == t.toCharArray().sorted()
}
```

### Approach 3 — HashMap (handles Unicode)

```kotlin
fun isAnagram(s: String, t: String): Boolean {
    if (s.length != t.length) return false

    val count = HashMap<Char, Int>()

    for (c in s) count[c] = (count[c] ?: 0) + 1
    for (c in t) count[c] = (count[c] ?: 0) - 1

    return count.values.all { it == 0 }
}
```

---

## Why the IntArray Approach Is Best Here

Using `IntArray(26)` instead of `HashMap<Char, Int>` gives:

| | IntArray | HashMap |
|---|---|---|
| **Space** | O(1) — fixed 26 slots | O(k) — grows with unique chars |
| **Access** | O(1) direct index | O(1) amortized hash lookup |
| **Overhead** | Zero boxing | Boxing `Char` → `Character` |

**`s[i] - 'a'`** — character to array index:
```kotlin
count[s[i] - 'a']++
// Char arithmetic — direct index into IntArray
// No HashMap, no hashing, no boxing
```

**Single pass** — process both strings in one loop:
```kotlin
for (i in s.indices) {
    count[s[i] - 'a']++  // increment for s
    count[t[i] - 'a']--  // decrement for t
}
// vs two separate loops — same O(n) but half the iterations
```

**`count.all { it == 0 }`** — clean validation:
```kotlin
return count.all { it == 0 }
// vs Java: for (int c : count) if (c != 0) return false; return true;
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| `IntArray(26)` | Input is lowercase English letters — fastest |
| `HashMap` | Input has Unicode, special chars, or unknown charset |
| Sort | Readability matters more than performance |

---

## Follow-up — What if inputs contain Unicode?

The `IntArray(26)` approach only works for lowercase `a-z`.
For Unicode inputs, use the HashMap approach — it handles any character set.

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — single pass through both strings |
| **Space** | O(1) — fixed size IntArray of 26 |

---

## Key Takeaway

> Map chars to array indices with `char - 'a'`. Increment for one string,
> decrement for the other. All zeros at the end = anagram.
> IntArray beats HashMap when the character set is known and fixed.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/valid-anagram/)

---

{% include kotlin-dsa-nav.html %}
