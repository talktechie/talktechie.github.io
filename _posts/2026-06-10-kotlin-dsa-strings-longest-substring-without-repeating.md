---
title: "Strings: Longest Substring Without Repeating Characters — Kotlin Solution"
date: 2026-06-10
categories: [Kotlin DSA, Strings]
tags: [longest-substring-without-repeating, sliding-window, hashset, strings, medium, leetcode, leetcode-3]
leetcode_number: 3
leetcode_url: https://leetcode.com/problems/longest-substring-without-repeating-characters/
difficulty: Medium
permalink: /posts/kotlin-dsa-strings-longest-substring-without-repeating/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [3 — Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) |
| **Difficulty** | Medium |
| **Topic** | Strings, Sliding Window, HashSet |

> Given a string `s`, find the length of the longest substring
> without repeating characters.

**Example:**
```
Input:  s = "abcabcbb"
Output: 3   // "abc"

Input:  s = "bbbbb"
Output: 1   // "b"

Input:  s = "pwwkew"
Output: 3   // "wke"
```

---

## Approach

This is a classic **sliding window** problem.

Maintain a window with two pointers — `left` and `right`.
Expand `right` to grow the window. When a duplicate character is found,
shrink from `left` until the duplicate is gone.

**Walk through `"abcabcbb"`:**
```
left=0, right=0: window="a"      set={a}       max=1
left=0, right=1: window="ab"     set={a,b}     max=2
left=0, right=2: window="abc"    set={a,b,c}   max=3
left=0, right=3: 'a' duplicate!
  → remove s[left]='a', left=1   set={b,c}
left=1, right=3: window="bca"    set={b,c,a}   max=3
left=1, right=4: 'b' duplicate!
  → remove s[left]='b', left=2   set={c,a}
left=2, right=4: window="cab"    set={c,a,b}   max=3
...and so on
```

Answer: **3**

---

## Kotlin Solution

### Approach 1 — Sliding Window with HashSet

```kotlin
fun lengthOfLongestSubstring(s: String): Int {
    val seen = HashSet<Char>()
    var left = 0
    var maxLength = 0

    for (right in s.indices) {
        while (s[right] in seen) {
            seen.remove(s[left])
            left++
        }
        seen.add(s[right])
        maxLength = maxOf(maxLength, right - left + 1)
    }

    return maxLength
}
```

### Approach 2 — Sliding Window with HashMap (jump left pointer directly)

```kotlin
fun lengthOfLongestSubstring(s: String): Int {
    val lastSeen = HashMap<Char, Int>() // char -> last index
    var left = 0
    var maxLength = 0

    for (right in s.indices) {
        if (s[right] in lastSeen && lastSeen[s[right]]!! >= left) {
            left = lastSeen[s[right]]!! + 1
        }
        lastSeen[s[right]] = right
        maxLength = maxOf(maxLength, right - left + 1)
    }

    return maxLength
}
```

---

## Why Kotlin Shines Here

**`s.indices`** — clean range iteration over string indices:
```kotlin
for (right in s.indices)
// vs Java: for (int right = 0; right < s.length(); right++)
```

**`in` operator for HashSet lookup:**
```kotlin
while (s[right] in seen)
// vs Java: while (seen.contains(s.charAt(right)))
```

**`!!` for known non-null** — when you've already checked `in lastSeen`,
you know it's not null:
```kotlin
lastSeen[s[right]]!! >= left
// The !! is intentional here — we just confirmed the key exists
```

---

## Two Approaches Compared

| Approach | Time | Space | Notes |
|---|---|---|---|
| HashSet | O(n) | O(min(n,m)) | Simple, easy to reason about |
| HashMap | O(n) | O(min(n,m)) | Fewer steps — jumps `left` directly |

`m` = size of the character set (26 for lowercase letters, 128 for ASCII).

The **HashMap approach** is more efficient in practice because `left`
jumps directly to the right position instead of moving one step at a time.

---

## Common Mistake

```kotlin
// ❌ Wrong — doesn't handle when duplicate is outside the current window
if (s[right] in lastSeen) {
    left = lastSeen[s[right]]!! + 1  // left might jump backward!
}

// ✅ Correct — only update left if the duplicate is inside the window
if (s[right] in lastSeen && lastSeen[s[right]]!! >= left) {
    left = lastSeen[s[right]]!! + 1
}
```

Without the `>= left` check, `left` could jump to an old position
before the current window, giving a wrong (larger) result.

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — each character visited at most twice |
| **Space** | O(min(n, m)) — window size bounded by character set |

---

## Key Takeaway

> Sliding window: expand right freely, shrink left only when a duplicate
> appears. Track what's in the window with a HashSet or HashMap.
> The HashMap variant lets you jump `left` directly instead of crawling.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

---

{% include kotlin-dsa-nav.html %}
