---
title: "Sliding Window: Longest Substring Without Repeating Characters — Kotlin Solution"
date: 2026-06-19
categories: [Kotlin DSA, Sliding Window]
tags: [longest-substring-without-repeating-characters, sliding-window, hashmap, string, medium, leetcode, leetcode-3]
leetcode_number: 3
leetcode_url: https://leetcode.com/problems/longest-substring-without-repeating-characters/
difficulty: Medium
permalink: /posts/kotlin-dsa-sliding-window-longest-substring-without-repeating-characters/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [3 — Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) |
| **Difficulty** | Medium |
| **Topic** | Sliding Window, HashMap, String |

> Given a string `s`, find the length of the **longest substring** without
> repeating characters.

**Example:**
```
Input:  s = "abcabcbb"
Output: 3
Explanation: "abc" has length 3, no repeats

Input:  s = "bbbbb"
Output: 1
Explanation: "b" — only one character can be in the window at a time

Input:  s = "pwwkew"
Output: 3
Explanation: "wke" — note "pwke" is not a substring, the characters
must be contiguous
```

**Constraints:**
- `0 <= s.length <= 5 * 10⁴`
- `s` consists of English letters, digits, symbols, and spaces

---

## Approach

This is the canonical **variable-size sliding window** problem.

**Key insight:** Expand the window by moving `right` forward, adding
characters as we go. The moment we'd add a character that's **already
inside the current window**, shrink from the left — moving `left` forward
past the previous occurrence — until the duplicate is gone. Track the
longest window length seen at every step.

Using a `HashMap<Char, Int>` storing each character's most recent index
lets us jump `left` directly to where it needs to go, instead of shrinking
one character at a time.

**Walk through `"abcabcbb"`:**
```
right=0 'a' → not seen → window="a"      → lastSeen={a:0} → maxLen=1
right=1 'b' → not seen → window="ab"     → lastSeen={a:0,b:1} → maxLen=2
right=2 'c' → not seen → window="abc"    → lastSeen={a:0,b:1,c:2} → maxLen=3
right=3 'a' → seen at 0, and 0 >= left(0) → left = 0+1 = 1
            → window="bca"   → lastSeen={a:3,b:1,c:2} → maxLen=3
right=4 'b' → seen at 1, and 1 >= left(1) → left = 1+1 = 2
            → window="cab"   → lastSeen={a:3,b:4,c:2} → maxLen=3
right=5 'c' → seen at 2, and 2 >= left(2) → left = 2+1 = 3
            → window="abc"   → lastSeen={a:3,b:4,c:5} → maxLen=3
right=6 'b' → seen at 4, and 4 >= left(3) → left = 4+1 = 5
            → window="cb"    → lastSeen={a:3,b:6,c:5} → maxLen=3
right=7 'b' → seen at 6, and 6 >= left(5) → left = 6+1 = 7
            → window="b"     → lastSeen={a:3,b:7,c:5} → maxLen=3

Answer: 3 ✓
```

---

## Kotlin Solution

### Approach 1 — HashMap tracking last seen index (optimal)

```kotlin
fun lengthOfLongestSubstring(s: String): Int {
    val lastSeen = HashMap<Char, Int>()
    var left = 0
    var maxLen = 0

    for (right in s.indices) {
        val c = s[right]

        if (lastSeen.containsKey(c) && lastSeen[c]!! >= left) {
            left = lastSeen[c]!! + 1   // jump left past the duplicate
        }

        lastSeen[c] = right
        maxLen = maxOf(maxLen, right - left + 1)
    }

    return maxLen
}
```

### Approach 2 — HashSet, shrink one step at a time

```kotlin
fun lengthOfLongestSubstring(s: String): Int {
    val window = HashSet<Char>()
    var left = 0
    var maxLen = 0

    for (right in s.indices) {
        while (s[right] in window) {
            window.remove(s[left])
            left++
        }
        window.add(s[right])
        maxLen = maxOf(maxLen, right - left + 1)
    }

    return maxLen
}
```

Shrinks one character at a time instead of jumping — both are O(n)
amortized, but the HashMap version (Approach 1) avoids a nested loop
entirely, which can feel cleaner to reason about.

---

## Why We Check `lastSeen[c]!! >= left`

This check matters more than it looks. A character might have appeared
**before** the current window started — in that case, its old occurrence is
already outside the window and shouldn't affect `left` at all.

```kotlin
if (lastSeen.containsKey(c) && lastSeen[c]!! >= left) {
    left = lastSeen[c]!! + 1
}
```

Without the `>= left` check, a stale index from before the window's start
could incorrectly move `left` **backward** — shrinking the window when it
shouldn't, and potentially producing a wrong (smaller) answer.

**`maxOf(maxLen, right - left + 1)`** computes the current window's size on
every iteration — `right - left + 1` is the standard window-length formula:
```kotlin
maxLen = maxOf(maxLen, right - left + 1)
// window spans indices [left, right] inclusive → size = right - left + 1
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| HashMap with jump (Approach 1) | Always — avoids the inner while loop, true O(n) per character |
| HashSet with shrink loop (Approach 2) | Easier to explain conceptually first, before optimizing |

---

## Complexity

| | HashMap Jump | HashSet Shrink |
|---|---|---|
| **Time** | O(n) | O(n) amortized |
| **Space** | O(min(n, charset size)) | O(min(n, charset size)) |

---

## Key Takeaway

> Variable-size sliding window: expand `right` freely, and shrink `left`
> only when a constraint is violated — here, "no duplicate characters in
> the window." Storing each character's last-seen index lets `left` jump
> directly to where it needs to be, rather than shrinking step by step.
> Always re-validate that a stored index is actually inside the current
> window before using it. This expand-then-conditionally-shrink template
> is the backbone of every problem in this phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

---

{% include kotlin-dsa-nav.html %}
