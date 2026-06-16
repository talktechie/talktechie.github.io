---
title: "Arrays: Group Anagrams — Kotlin Solution"
date: 2026-06-16
categories: [Kotlin DSA, Arrays]
tags: [group-anagrams, hashmap, array, sorting, medium, leetcode, leetcode-49]
leetcode_number: 49
leetcode_url: https://leetcode.com/problems/group-anagrams/
difficulty: Medium
permalink: /posts/kotlin-dsa-arrays-group-anagrams/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [49 — Group Anagrams](https://leetcode.com/problems/group-anagrams/) |
| **Difficulty** | Medium |
| **Topic** | Arrays, HashMap, Sorting |

> Given an array of strings `strs`, group the anagrams together.
> You can return the answer in any order.
>
> An anagram uses all the original characters exactly once — just rearranged.

**Example:**
```
Input:  strs = ["eat","tea","tan","ate","nat","bat"]
Output: [["bat"],["nat","tan"],["ate","eat","tea"]]

Input:  strs = [""]
Output: [[""]]

Input:  strs = ["a"]
Output: [["a"]]
```

**Constraints:**
- `1 <= strs.length <= 10⁴`
- `0 <= strs[i].length <= 100`
- `strs[i]` consists of lowercase English letters only

---

## Approach

Two strings are anagrams if and only if they have the **same characters with the same frequency**.

That means if you sort the characters of two anagrams, you get the **same string**.

**Key insight:** Use the sorted form as a HashMap key — all anagrams map to the same key, so they naturally land in the same bucket.

**Walk through `["eat","tea","tan","ate","nat","bat"]`:**
```
"eat" → sorted → "aet"  → map["aet"] = ["eat"]
"tea" → sorted → "aet"  → map["aet"] = ["eat","tea"]
"tan" → sorted → "ant"  → map["ant"] = ["tan"]
"ate" → sorted → "aet"  → map["aet"] = ["eat","tea","ate"]
"nat" → sorted → "ant"  → map["ant"] = ["tan","nat"]
"bat" → sorted → "abt"  → map["abt"] = ["bat"]

Result: [["eat","tea","ate"],["tan","nat"],["bat"]] ✓
```

This is a natural extension of Valid Anagram (LC 242) — instead of comparing two strings, we're grouping many under a shared canonical form.

---

## Kotlin Solution

### Approach 1 — Sort as key (idiomatic Kotlin)

```kotlin
fun groupAnagrams(strs: Array<String>): List<List<String>> {
    return strs
        .groupBy { it.toCharArray().sorted().joinToString("") }
        .values
        .toList()
}
```

Kotlin's `groupBy` does exactly what we need in one expression — no manual map management.

### Approach 2 — Frequency count as key (optimal)

```kotlin
fun groupAnagrams(strs: Array<String>): List<List<String>> {
    val map = HashMap<String, MutableList<String>>()

    for (s in strs) {
        val count = IntArray(26)
        for (c in s) count[c - 'a']++

        val key = count.joinToString(",")
        map.getOrPut(key) { mutableListOf() }.add(s)
    }

    return map.values.toList()
}
```

### Approach 3 — Explicit HashMap with sorted key

```kotlin
fun groupAnagrams(strs: Array<String>): List<List<String>> {
    val map = HashMap<String, MutableList<String>>()

    for (s in strs) {
        val key = s.toCharArray().also { it.sort() }.joinToString("")
        map.getOrPut(key) { mutableListOf() }.add(s)
    }

    return map.values.toList()
}
```

---

## Why the Frequency Count Key Is Faster

Sorting each string costs `O(k log k)`. Building a frequency count costs `O(k)` — a meaningful difference when strings can be up to 100 chars long.

| | Sort Key | Frequency Count Key |
|---|---|---|
| **Key build cost** | O(k log k) per string | O(k) per string |
| **Total time** | O(n · k log k) | O(n · k) |
| **Space** | O(n · k) | O(n · k) |
| **Readability** | ✅ Clear | ⚠️ Slightly more verbose |

**`count.joinToString(",")`** — turning the frequency array into a deterministic string key:
```kotlin
val count = IntArray(26)
for (c in s) count[c - 'a']++
val key = count.joinToString(",")
// "eat" → "1,0,0,0,1,0,...,1,0,0,0"  (a=1, e=1, t=1)
// "tea" → same key ✓
```

**`map.getOrPut(key) { mutableListOf() }.add(s)`** — clean Kotlin idiom:
```kotlin
map.getOrPut(key) { mutableListOf() }.add(s)
// vs Java: if (!map.containsKey(key)) map.put(key, new ArrayList<>());
//          map.get(key).add(s);
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| `groupBy` (Approach 1) | Clean production code, readability first |
| Frequency count (Approach 2) | Performance-critical, long strings |
| Explicit HashMap (Approach 3) | Interview — shows deliberate key construction |

---

## Follow-up — What if strings can contain Unicode?

The `IntArray(26)` key only works for lowercase `a-z`.
For Unicode strings, fall back to the sorted string key — it handles any character set without changes.

---

## Complexity

| | Sort Key | Frequency Count Key |
|---|---|---|
| **Time** | O(n · k log k) | O(n · k) |
| **Space** | O(n · k) | O(n · k) |

Where `n` = number of strings, `k` = max string length.

---

## Key Takeaway

> Anagrams share a canonical form. Sort the characters or count their frequencies
> to get a key every anagram in a group maps to identically.
> One HashMap pass groups them all — no comparisons between strings needed.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/group-anagrams/)

---

{% include kotlin-dsa-nav.html %}
