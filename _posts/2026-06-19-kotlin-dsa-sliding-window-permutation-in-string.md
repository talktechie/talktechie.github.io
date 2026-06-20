---
title: "Sliding Window: Permutation In String — Kotlin Solution"
date: 2026-06-19
categories: [Kotlin DSA, Sliding Window]
tags: [permutation-in-string, sliding-window, hashmap, string, medium, leetcode, leetcode-567]
leetcode_number: 567
leetcode_url: https://leetcode.com/problems/permutation-in-string/
difficulty: Medium
permalink: /posts/kotlin-dsa-sliding-window-permutation-in-string/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [567 — Permutation In String](https://leetcode.com/problems/permutation-in-string/) |
| **Difficulty** | Medium |
| **Topic** | Sliding Window, HashMap, String |

> Given two strings `s1` and `s2`, return `true` if `s2` contains a
> **permutation** of `s1`, or `false` otherwise.
>
> A permutation uses every character of `s1` exactly once, in any order.

**Example:**
```
Input:  s1 = "ab", s2 = "eidbaooo"
Output: true
Explanation: s2 contains "ba", a permutation of "ab"

Input:  s1 = "ab", s2 = "eidboaoo"
Output: false
```

**Constraints:**
- `1 <= s1.length, s2.length <= 10⁴`
- `s1` and `s2` consist of lowercase English letters only

---

## Approach

A permutation of `s1` has **exactly the same character frequencies** as
`s1`, just rearranged. So this becomes: does any **fixed-size window** of
`s2` (size `s1.length`) have the same character frequency count as `s1`?

**Key insight — fixed-size window:** Unlike the previous two problems
where the window grew and shrank dynamically, here the window size is
known up front: `s1.length`. Slide it across `s2` one character at a time
— add the new right character, remove the leftmost character that's
falling out of the window, and compare frequency counts.

**Naive comparison** of two 26-length frequency arrays on every slide is
O(26n) — fine, but we can do better by tracking a single `matches` counter
that tells us in O(1) whether the window currently matches.

**Walk through `s1 = "ab", s2 = "eidbaooo"`:**
```
s1Count = {a:1, b:1}
window size = 2

window="ei" (indices 0-1) → count={e:1,i:1} → matches s1Count? No
slide → window="id" → count={i:1,d:1} → No
slide → window="db" → count={d:1,b:1} → No
slide → window="ba" → count={b:1,a:1} → matches s1Count? Yes! → return true ✓
```

---

## Kotlin Solution

### Approach 1 — Fixed-size sliding window with frequency arrays (optimal)

```kotlin
fun checkInclusion(s1: String, s2: String): Boolean {
    if (s1.length > s2.length) return false

    val s1Count = IntArray(26)
    val windowCount = IntArray(26)

    for (c in s1) s1Count[c - 'a']++
    for (i in 0 until s1.length) windowCount[s2[i] - 'a']++

    if (s1Count.contentEquals(windowCount)) return true

    for (i in s1.length until s2.length) {
        windowCount[s2[i] - 'a']++              // add new right character
        windowCount[s2[i - s1.length] - 'a']--   // remove the character sliding out

        if (s1Count.contentEquals(windowCount)) return true
    }

    return false
}
```

### Approach 2 — O(1) per-slide comparison using a `matches` counter

```kotlin
fun checkInclusion(s1: String, s2: String): Boolean {
    if (s1.length > s2.length) return false

    val s1Count = IntArray(26)
    val windowCount = IntArray(26)

    for (c in s1) s1Count[c - 'a']++
    for (i in 0 until s1.length) windowCount[s2[i] - 'a']++

    var matches = (0 until 26).count { s1Count[it] == windowCount[it] }
    if (matches == 26) return true

    for (i in s1.length until s2.length) {
        val addIdx = s2[i] - 'a'
        val removeIdx = s2[i - s1.length] - 'a'

        // Update matches for the character being added
        if (windowCount[addIdx] == s1Count[addIdx]) matches--
        windowCount[addIdx]++
        if (windowCount[addIdx] == s1Count[addIdx]) matches++

        // Update matches for the character being removed
        if (windowCount[removeIdx] == s1Count[removeIdx]) matches--
        windowCount[removeIdx]--
        if (windowCount[removeIdx] == s1Count[removeIdx]) matches++

        if (matches == 26) return true
    }

    return false
}
```

Avoids comparing all 26 slots on every slide — instead tracks how many of
the 26 letter-counts currently agree, updating incrementally.

---

## Why `contentEquals` Comparison Is Already Fast Enough Here

Approach 1's `contentEquals` check is O(26) per slide, giving O(26n)
overall — and since 26 is a constant, this is still effectively O(n) for
practical purposes. Approach 2 reduces the constant factor with the
`matches` counter trick, but on LeetCode's constraints, Approach 1 is
simpler and plenty fast.

**The sliding update — add right, remove left, in one step:**
```kotlin
windowCount[s2[i] - 'a']++              // character entering the window
windowCount[s2[i - s1.length] - 'a']--   // character leaving the window
// i - s1.length is always the index of the character now outside the
// window's left boundary — window size stays fixed at s1.length
```

**Why check before the loop too** — the very first window (indices
`0 until s1.length`) needs to be validated before any sliding happens:
```kotlin
for (i in 0 until s1.length) windowCount[s2[i] - 'a']++
if (s1Count.contentEquals(windowCount)) return true   // check window #1 immediately
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| `contentEquals` comparison (Approach 1) | Simpler, still effectively O(n) due to fixed alphabet size |
| `matches` counter (Approach 2) | Want the tightest possible constant factor, comfortable with more bookkeeping |

---

## Complexity

| | Approach 1 | Approach 2 |
|---|---|---|
| **Time** | O(26n) ≈ O(n) | O(n) |
| **Space** | O(1) — fixed 26-slot arrays | O(1) |

---

## Key Takeaway

> When the target is a fixed length (a permutation has the exact same
> length as the original string), use a **fixed-size sliding window**
> instead of a growing/shrinking one. Slide by adding the new right
> character and removing the one leaving from the left — both happen on
> the same step, keeping window size constant throughout. Comparing
> frequency arrays (or a `matches` counter) tells you in O(1)-ish time
> whether the window is currently a permutation match.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/permutation-in-string/)

---

{% include kotlin-dsa-nav.html %}
