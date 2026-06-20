---
title: "Sliding Window: Longest Repeating Character Replacement — Kotlin Solution"
date: 2026-06-19
categories: [Kotlin DSA, Sliding Window]
tags: [longest-repeating-character-replacement, sliding-window, hashmap, string, medium, leetcode, leetcode-424]
leetcode_number: 424
leetcode_url: https://leetcode.com/problems/longest-repeating-character-replacement/
difficulty: Medium
permalink: /posts/kotlin-dsa-sliding-window-longest-repeating-character-replacement/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [424 — Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/) |
| **Difficulty** | Medium |
| **Topic** | Sliding Window, HashMap, String |

> Given a string `s` and an integer `k`, you can choose **any** character
> in the string and change it to any other uppercase English letter, at
> most `k` times.
>
> Return the length of the longest substring containing the same letter
> after performing the replacements.

**Example:**
```
Input:  s = "ABAB", k = 2
Output: 4
Explanation: Replace the two 'A's with 'B's (or vice versa) → "BBBB"

Input:  s = "AABABBA", k = 1
Output: 4
Explanation: Replace one 'A' in "ABBA" → "BBBB", substring length 4
```

**Constraints:**
- `1 <= s.length <= 10⁵`
- `s` consists of only uppercase English letters
- `0 <= k <= s.length`

---

## Approach

A window of length `windowLen` is **achievable** with at most `k`
replacements if and only if:

```
windowLen - (count of the most frequent character in the window) <= k
```

In other words: replace every character in the window *except* the most
frequent one, and check if that requires ≤ `k` replacements.

**Key insight:** Expand the window greedily. We don't need to find the
true most-frequent count after every shrink — we only need to track the
**maximum frequency ever seen** in any window of this size. If that's
still good enough to satisfy the formula, keep the window size; if not,
shrink by exactly one from the left (never more) and move on.

This subtle trick — never shrinking the window size, only sliding it — is
what keeps this O(n) instead of O(26n).

**Walk through `s = "AABABBA", k = 1`:**
```
right=0 'A' → count={A:1}        maxFreq=1  len=1  1-1=0<=1 ✓
right=1 'A' → count={A:2}        maxFreq=2  len=2  2-2=0<=1 ✓
right=2 'B' → count={A:2,B:1}    maxFreq=2  len=3  3-2=1<=1 ✓
right=3 'A' → count={A:3,B:1}    maxFreq=3  len=4  4-3=1<=1 ✓
right=4 'B' → count={A:3,B:2}    maxFreq=3  len=5  5-3=2>1  → shrink: remove s[left]='A'
              count={A:2,B:2}  left=1  len=4  4-3=1<=1 (maxFreq stays 3, stale but harmless)
right=5 'B' → count={A:2,B:3}    maxFreq=3  len=5  5-3=2>1  → shrink: remove s[left]='A'
              count={A:1,B:3}  left=2  len=4  4-3=1<=1 ✓
right=6 'A' → count={A:2,B:3}    maxFreq=3  len=5  5-3=2>1  → shrink: remove s[left]='B'
              count={A:2,B:2}  left=3  len=4  4-3=1<=1 ✓

Max window length reached at any point: 4 ✓
```

---

## Kotlin Solution

### Approach 1 — Sliding window, never shrink window size (optimal)

```kotlin
fun characterReplacement(s: String, k: Int): Int {
    val count = IntArray(26)
    var left = 0
    var maxFreq = 0
    var maxLen = 0

    for (right in s.indices) {
        count[s[right] - 'A']++
        maxFreq = maxOf(maxFreq, count[s[right] - 'A'])

        val windowLen = right - left + 1
        if (windowLen - maxFreq > k) {
            count[s[left] - 'A']--
            left++
        }

        maxLen = maxOf(maxLen, right - left + 1)
    }

    return maxLen
}
```

### Approach 2 — Recompute max frequency explicitly each time (clearer, slower)

```kotlin
fun characterReplacement(s: String, k: Int): Int {
    val count = IntArray(26)
    var left = 0
    var maxLen = 0

    for (right in s.indices) {
        count[s[right] - 'A']++

        while ((right - left + 1) - count.max() > k) {
            count[s[left] - 'A']--
            left++
        }

        maxLen = maxOf(maxLen, right - left + 1)
    }

    return maxLen
}
```

Recomputes `count.max()` (the true current max frequency) on every check —
correct, but O(26) per character, making this O(26n) overall. Approach 1
avoids this entirely.

---

## Why a "Stale" maxFreq Doesn't Break Approach 1

This is the trick that makes Approach 1 work, and it trips people up the
first time they see it: `maxFreq` is **never decreased**, even after a
character is removed from the window.

```kotlin
maxFreq = maxOf(maxFreq, count[s[right] - 'A'])
// maxFreq only ever grows or stays the same — it represents the best
// frequency ever achieved in ANY window of the current size, not
// necessarily the current window specifically
```

Why is this safe? Because we're looking for the **longest** valid window.
If `maxFreq` is stale (higher than the true current max), the formula
`windowLen - maxFreq <= k` becomes *harder* to satisfy, not easier — so we
might shrink a window that's actually still valid, but we'll never report
a window as valid when it isn't. The final `maxLen` is never overstated,
and since we're hunting for the maximum length, the answer stays correct.

**The window only ever shrinks by 1, never resets:**
```kotlin
if (windowLen - maxFreq > k) {
    count[s[left] - 'A']--
    left++
}
// not a while loop — only one character is ever removed per iteration
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Stale maxFreq trick (Approach 1) | Always — true O(n), the standard accepted solution |
| Recompute max (Approach 2) | Building intuition first; still passes but slower |

---

## Complexity

| | Approach 1 | Approach 2 |
|---|---|---|
| **Time** | O(n) | O(26n) |
| **Space** | O(1) — fixed 26-slot array | O(1) |

---

## Key Takeaway

> A window is valid if `windowLen - mostFrequentCount <= k`. Track the
> highest frequency ever seen rather than recomputing it — it's safe to
> let it go stale because that only makes the check stricter, never looser,
> and we only care about the longest valid window overall. This "track a
> monotonic best-so-far value instead of recomputing" trick shows up
> across many sliding window problems where the exact internal state
> doesn't need to be perfectly accurate at every step.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/longest-repeating-character-replacement/)

---

{% include kotlin-dsa-nav.html %}
