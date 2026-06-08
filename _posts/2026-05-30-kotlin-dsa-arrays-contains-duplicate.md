---
title: "Arrays: Contains Duplicate — Kotlin Solution"
date: 2026-05-30
categories: [Kotlin DSA, Arrays]
tags: [contains-duplicate, hashset, easy, leetcode, leetcode-217, arrays]
leetcode_number: 217
leetcode_url: https://leetcode.com/problems/contains-duplicate/
difficulty: Easy
permalink: /posts/kotlin-dsa-arrays-contains-duplicate/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [217 — Contains Duplicate](https://leetcode.com/problems/contains-duplicate/) |
| **Difficulty** | Easy |
| **Topic** | Arrays, HashSet |

> Given an integer array `nums`, return `true` if any value appears at least twice.

**Example:**
```
Input:  nums = [1, 2, 3, 1]
Output: true

Input:  nums = [1, 2, 3, 4]
Output: false
```

---

## Approach

Use a HashSet. For each number, check if it's already in the set.
If yes — duplicate found. If no — add it and continue.

---

## Kotlin Solution

```kotlin
fun containsDuplicate(nums: IntArray): Boolean {
    val seen = HashSet<Int>()

    for (num in nums) {
        if (!seen.add(num)) return true
    }

    return false
}
```

### One-liner version (idiomatic Kotlin)

```kotlin
fun containsDuplicate(nums: IntArray): Boolean {
    return nums.size != nums.toHashSet().size
}
```

---

## Why Kotlin Shines Here

**`seen.add()` returns Boolean** — `HashSet.add()` returns `false`
if the element already exists. We use that directly:
```kotlin
if (!seen.add(num)) return true
// No separate contains() check needed
```

**`toHashSet()`** — Kotlin's collection extension makes the one-liner possible:
```kotlin
nums.toHashSet() // converts IntArray to HashSet<Int> in one call
```

---

## Two Approaches Compared

| Approach | Time | Space | Notes |
|---|---|---|---|
| HashSet with early exit | O(n) | O(n) | Stops as soon as duplicate found |
| One-liner `toHashSet()` | O(n) | O(n) | Clean but no early exit |

Use the **first approach** in interviews — shows you understand early termination.

---

## Complexity

| | |
|---|---|
| **Time** | O(n) |
| **Space** | O(n) |

---

## Key Takeaway

> HashSet `add()` returns false on duplicates — use that return value directly instead of a separate `contains()` check.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/contains-duplicate/)

---

{% include kotlin-dsa-nav.html %}
