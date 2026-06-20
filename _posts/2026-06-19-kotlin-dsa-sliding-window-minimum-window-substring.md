---
title: "Sliding Window: Minimum Window Substring — Kotlin Solution"
date: 2026-06-19
categories: [Kotlin DSA, Sliding Window]
tags: [minimum-window-substring, sliding-window, hashmap, string, hard, leetcode, leetcode-76]
leetcode_number: 76
leetcode_url: https://leetcode.com/problems/minimum-window-substring/
difficulty: Hard
permalink: /posts/kotlin-dsa-sliding-window-minimum-window-substring/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [76 — Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) |
| **Difficulty** | Hard |
| **Topic** | Sliding Window, HashMap, String |

> Given two strings `s` and `t`, return the **shortest substring** of `s`
> that contains all the characters of `t` (including duplicates). If no
> such substring exists, return `""`.

**Example:**
```
Input:  s = "ADOBECODEBANC", t = "ABC"
Output: "BANC"
Explanation: The substring "BANC" includes 'A', 'B', and 'C' from t

Input:  s = "a", t = "a"
Output: "a"

Input:  s = "a", t = "aa"
Output: ""
Explanation: t needs two 'a's, but s only has one
```

**Constraints:**
- `1 <= s.length, t.length <= 10⁵`
- `s` and `t` consist of uppercase and lowercase English letters
- It's guaranteed that the answer is unique when it exists

---

## Approach

This is the most general sliding window pattern: a variable-size window
that **expands until valid, then shrinks as much as possible while
staying valid** — recording the smallest valid window found at any point.

**Key insight:** Build a frequency map of `t`. Expand the window (`right`
pointer) until it contains **at least** the required count of every
character in `t`. Once valid, greedily shrink from the left — every time
we successfully shrink and stay valid, we might have found a new minimum.
The moment shrinking breaks validity, stop shrinking and resume expanding.

Track validity with a single `have` vs `need` counter pair, instead of
comparing full frequency maps on every step — this is what keeps it O(n+m).

**Walk through `s = "ADOBECODEBANC", t = "ABC"`:**
```
need = {A:1, B:1, C:1}   required = 3 distinct chars to satisfy

Expand right until valid:
  right reaches index 9 ('A') → window = "ADOBECODEBA" → have all of A,B,C
  → valid! window length = 10 → record as best so far

Shrink left while still valid:
  remove 'A' (index 0) → still have enough A's elsewhere? No → invalid, stop shrinking
  Actually: shrink one at a time, re-checking validity after each removal.
  left moves from 0 → 6 while window stays valid (removing extra D,O,B,E,C,O)
  window = "CODEBA" (indices 6-9)... continuing the slide:

Eventually right reaches index 12 ('C'), window becomes valid again,
shrink left from current position until invalid:
  Final minimum found: "BANC" (indices 9-12), length 4

Answer: "BANC" ✓
```

---

## Kotlin Solution

### Approach 1 — `have`/`need` counter sliding window (optimal)

```kotlin
fun minWindow(s: String, t: String): String {
    if (t.isEmpty() || s.length < t.length) return ""

    val need = HashMap<Char, Int>()
    for (c in t) need[c] = (need[c] ?: 0) + 1

    val window = HashMap<Char, Int>()
    var have = 0
    val required = need.size

    var resultLen = Int.MAX_VALUE
    var resultLeft = 0
    var left = 0

    for (right in s.indices) {
        val c = s[right]
        window[c] = (window[c] ?: 0) + 1

        if (need.containsKey(c) && window[c] == need[c]) {
            have++
        }

        while (have == required) {
            // Record this window if it's the smallest so far
            if (right - left + 1 < resultLen) {
                resultLen = right - left + 1
                resultLeft = left
            }

            // Try shrinking from the left
            val leftChar = s[left]
            window[leftChar] = window[leftChar]!! - 1
            if (need.containsKey(leftChar) && window[leftChar]!! < need[leftChar]!!) {
                have--
            }
            left++
        }
    }

    return if (resultLen == Int.MAX_VALUE) "" else s.substring(resultLeft, resultLeft + resultLen)
}
```

### Approach 2 — IntArray(128) for ASCII instead of HashMap

```kotlin
fun minWindow(s: String, t: String): String {
    if (t.isEmpty() || s.length < t.length) return ""

    val need = IntArray(128)
    for (c in t) need[c.code]++

    var required = t.length   // total characters still needed (with duplicates)
    var left = 0
    var resultLen = Int.MAX_VALUE
    var resultLeft = 0

    for (right in s.indices) {
        val c = s[right]
        if (need[c.code] > 0) required--
        need[c.code]--

        while (required == 0) {
            if (right - left + 1 < resultLen) {
                resultLen = right - left + 1
                resultLeft = left
            }

            need[s[left].code]++
            if (need[s[left].code] > 0) required++
            left++
        }
    }

    return if (resultLen == Int.MAX_VALUE) "" else s.substring(resultLeft, resultLeft + resultLen)
}
```

A clever variant: `need[c]` going **positive** means we're missing that
character; decrementing it below zero just means "extra copies available."
`required` tracks total missing characters directly, avoiding the
`have == required` distinct-character bookkeeping of Approach 1.

---

## Why `have == required` Avoids Comparing Full Maps

Comparing two full HashMaps on every iteration would be O(26) (or O(52)
with mixed case) per step — not terrible, but unnecessary. Instead:

```kotlin
if (need.containsKey(c) && window[c] == need[c]) {
    have++   // this specific character JUST became fully satisfied
}
```

`have` only increments the **instant** a character's count in the window
exactly matches what's needed — not every time that character appears.
This keeps each window update O(1) instead of O(26).

**Why the inner `while`, not `if`, when shrinking:**
```kotlin
while (have == required) {
    // ... shrink as much as possible while still valid
}
```
A single valid window might allow multiple consecutive left-shifts before
becoming invalid — using `while` instead of `if` ensures we find the
truly minimal window at this expansion point, not just shrink by one.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| HashMap `have`/`need` (Approach 1) | Clearer mental model — distinct characters fully satisfied |
| IntArray with `required` count (Approach 2) | Slightly faster (array vs HashMap), elegant single-counter trick |

---

## Complexity

| | |
|---|---|
| **Time** | O(n + m) — n = length of s, m = length of t |
| **Space** | O(m) — frequency map sized to t's distinct characters |

---

## Key Takeaway

> The most general sliding window template: expand until valid, then
> shrink as much as possible while staying valid, recording the best
> result at every valid state. A `have`/`need` (or `required`) counter
> turns "is the window valid?" into an O(1) check instead of comparing
> full frequency maps. This is the hardest pattern in the Sliding Window
> phase — but it's really just one more `while` loop layered onto the
> same expand/shrink skeleton from Longest Substring Without Repeating
> Characters.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/minimum-window-substring/)

---

{% include kotlin-dsa-nav.html %}
