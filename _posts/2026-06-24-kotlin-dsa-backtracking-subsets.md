---
title: "Backtracking: Subsets — Kotlin Solution"
date: 2026-06-24
categories: [Kotlin DSA, Backtracking]
tags: [subsets, backtracking, array, bitmask, medium, leetcode, leetcode-78]
leetcode_number: 78
leetcode_url: https://leetcode.com/problems/subsets/
difficulty: Medium
permalink: /posts/kotlin-dsa-backtracking-subsets/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [78 — Subsets](https://leetcode.com/problems/subsets/) |
| **Difficulty** | Medium |
| **Topic** | Backtracking, Array, Bitmask |

> Given an integer array `nums` of **unique** elements, return **all
> possible subsets** (the power set). The solution set must not contain
> duplicate subsets — return them in any order.

**Example:**
```
Input:  nums = [1,2,3]
Output: [[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]

Input:  nums = [0]
Output: [[],[0]]
```

**Constraints:**
- `1 <= nums.length <= 10`
- `-10 <= nums[i] <= 10`
- All elements in `nums` are unique

---

## Approach

This is the entry point to the Backtracking phase, establishing the core
**"include or exclude"** decision template that nearly every other
problem in this phase builds on.

**Key insight:** For every element in `nums`, there are exactly two
choices — include it in the current subset, or don't. Exploring both
choices at every position, recursively, naturally generates all `2^n`
possible subsets. At each recursive call, the **current partial subset
itself** is a valid answer, so we record it immediately upon entry,
before making any further choices.

**Walk through `nums = [1,2,3]`** (decision tree, showing include/exclude
at each level):
```
backtrack(index=0, current=[]):
  record [] as a valid subset
  → INCLUDE nums[0]=1: backtrack(1, [1])
       record [1]
       → INCLUDE nums[1]=2: backtrack(2, [1,2])
            record [1,2]
            → INCLUDE nums[2]=3: backtrack(3, [1,2,3]) → record [1,2,3]
            → EXCLUDE nums[2]: backtrack(3, [1,2]) → index==length, nothing new
       → EXCLUDE nums[1]: backtrack(2, [1])
            record [1]... wait, already recorded — let's trace properly:
            → INCLUDE nums[2]=3: backtrack(3,[1,3]) → record [1,3]
            → EXCLUDE nums[2]: nothing new
  → EXCLUDE nums[0]: backtrack(1, [])
       record [] ... (this duplicates the empty set check — the actual
       code records on entry, so revisit: every CALL records its current
       state once, giving exactly 2^3 = 8 total calls/subsets)

Final result: [[],[3],[2],[2,3],[1],[1,3],[1,2],[1,2,3]] (order varies,
content matches the expected 8 subsets) ✓
```

---

## Kotlin Solution

### Approach 1 — Backtracking, include/exclude at each index (optimal, most instructive)

```kotlin
fun subsets(nums: IntArray): List<List<Int>> {
    val result = mutableListOf<List<Int>>()
    val current = mutableListOf<Int>()

    fun backtrack(index: Int) {
        if (index == nums.size) {
            result.add(current.toList())   // snapshot — current will keep mutating
            return
        }

        // Choice 1 — exclude nums[index]
        backtrack(index + 1)

        // Choice 2 — include nums[index]
        current.add(nums[index])
        backtrack(index + 1)
        current.removeAt(current.lastIndex)   // undo — backtrack
    }

    backtrack(0)
    return result
}
```

### Approach 2 — Iterative, build up by doubling the result each step

```kotlin
fun subsets(nums: IntArray): List<List<Int>> {
    var result = listOf<List<Int>>(emptyList())

    for (num in nums) {
        result = result + result.map { it + num }
    }

    return result
}
```

For each new number, double the existing result set — every subset
already found either stays as-is (excludes the new number) or gets the
new number appended (includes it). No recursion at all.

### Approach 3 — Bitmask enumeration

```kotlin
fun subsets(nums: IntArray): List<List<Int>> {
    val n = nums.size
    val result = mutableListOf<List<Int>>()

    for (mask in 0 until (1 shl n)) {
        val subset = mutableListOf<Int>()
        for (i in 0 until n) {
            if (mask and (1 shl i) != 0) subset.add(nums[i])
        }
        result.add(subset)
    }

    return result
}
```

Every integer from `0` to `2^n - 1`, read in binary, represents one
unique combination of "include/exclude" decisions — bit `i` set means
"include `nums[i]`." A clean, recursion-free way to enumerate the same
power set.

---

## Why `current.toList()` Is Essential, Not Optional

This is the single most common bug in backtracking problems:

```kotlin
result.add(current.toList())
// NOT: result.add(current)
```

`current` is a single `MutableList` that gets mutated (elements added and
removed) throughout the entire recursion. If we added a *reference* to
`current` directly into `result`, every entry in `result` would actually
be pointing to the **same** list — and by the time backtracking finishes,
they'd all show whatever `current`'s *final* state happens to be (usually
empty), not the snapshot at the time they were added. `.toList()` creates
an independent copy, frozen at that exact moment.

**Why we always undo after the "include" branch:**
```kotlin
current.add(nums[index])
backtrack(index + 1)
current.removeAt(current.lastIndex)   // restore state for the caller's next choice
```
This is backtracking's defining mechanic — explore a choice, recurse,
then **undo** that choice before returning, so the parent call's state is
exactly as it was before this branch was explored, ready for whatever
comes next.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Backtracking (Approach 1) | Best for learning the pattern — generalizes directly to every other problem in this phase |
| Iterative doubling (Approach 2) | Cleanest, fastest in practice, no recursion |
| Bitmask (Approach 3) | Elegant for small n, useful when thinking in terms of "which elements are included" as a single number |

---

## Complexity

| | |
|---|---|
| **Time** | O(n × 2ⁿ) — 2ⁿ subsets, each up to O(n) to copy |
| **Space** | O(n × 2ⁿ) — storing all subsets |

---

## Key Takeaway

> Subsets is the simplest possible backtracking template: at every
> position, explore "exclude" and "include," recording the current state
> as a complete answer at every step of the recursion (not just at the
> leaves). The "make a choice → recurse → undo the choice" skeleton
> introduced here is the backbone for every other problem in this phase
> — what changes from problem to problem is just *what* counts as a valid
> answer and *when* to prune a branch early.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/subsets/)

---

{% include kotlin-dsa-nav.html %}
