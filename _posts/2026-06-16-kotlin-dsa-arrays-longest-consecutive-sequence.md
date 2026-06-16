---
title: "Arrays: Longest Consecutive Sequence — Kotlin Solution"
date: 2026-06-16
categories: [Kotlin DSA, Arrays]
tags: [longest-consecutive-sequence, hashset, array, medium, leetcode, leetcode-128]
leetcode_number: 128
leetcode_url: https://leetcode.com/problems/longest-consecutive-sequence/
difficulty: Medium
permalink: /posts/kotlin-dsa-arrays-longest-consecutive-sequence/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [128 — Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/) |
| **Difficulty** | Medium |
| **Topic** | Arrays, HashSet |

> Given an unsorted array of integers `nums`, return the length of the
> longest consecutive elements sequence.
>
> **You must write an algorithm that runs in O(n) time.**

**Example:**
```
Input:  nums = [100,4,200,1,3,2]
Output: 4
Explanation: The longest consecutive sequence is [1,2,3,4]

Input:  nums = [0,3,7,2,5,8,4,6,0,1]
Output: 9
Explanation: The longest consecutive sequence is [0,1,2,3,4,5,6,7,8]
```

**Constraints:**
- `0 <= nums.length <= 10⁵`
- `-10⁹ <= nums[i] <= 10⁹`
- **Follow-up:** Must run in O(n) time — sorting is not allowed

---

## Approach

Sorting would give O(n log n) — the problem forbids this. We need O(n).

**Key insight — only start counting from sequence starts:**
A number `n` is the **start** of a consecutive sequence if and only if `n - 1`
is **not** in the set. If `n - 1` exists, then `n` is just the middle of a
longer sequence — we'll count it when we process the actual start.

This means each element is visited at most twice — once to check if it's a
start, and once as part of a `while` loop from its sequence's start. O(n) total.

**Walk through `[100, 4, 200, 1, 3, 2]`:**
```
numSet = {1, 2, 3, 4, 100, 200}

100 → is 99 in set? No → it's a sequence start
      101 in set? No → length = 1

4   → is 3 in set? Yes → skip (not a start)

200 → is 199 in set? No → it's a sequence start
      201 in set? No → length = 1

1   → is 0 in set? No → it's a sequence start
      2 in set? Yes → length = 2
      3 in set? Yes → length = 3
      4 in set? Yes → length = 4
      5 in set? No  → stop

3   → is 2 in set? Yes → skip
2   → is 1 in set? Yes → skip

longest = 4 ✓
```

---

## Kotlin Solution

### Approach 1 — HashSet + sequence start detection (optimal, O(n))

```kotlin
fun longestConsecutive(nums: IntArray): Int {
    val numSet = nums.toHashSet()
    var longest = 0

    for (n in numSet) {
        // Only start counting if n is the beginning of a sequence
        if (n - 1 !in numSet) {
            var length = 1
            while (n + length in numSet) length++
            longest = maxOf(longest, length)
        }
    }

    return longest
}
```

Iterating over `numSet` (not `nums`) automatically skips duplicates and avoids
re-processing the same sequence start multiple times.

### Approach 2 — Sort then scan (O(n log n), simpler but violates follow-up)

```kotlin
fun longestConsecutive(nums: IntArray): Int {
    if (nums.isEmpty()) return 0

    nums.sort()
    var longest = 1
    var current = 1

    for (i in 1..nums.lastIndex) {
        when {
            nums[i] == nums[i - 1] -> Unit          // duplicate — skip
            nums[i] == nums[i - 1] + 1 -> {         // consecutive
                current++
                longest = maxOf(longest, current)
            }
            else -> current = 1                      // gap — reset
        }
    }

    return longest
}
```

Clean and easy to follow in an interview, but O(n log n) — only use if the
follow-up constraint isn't enforced.

---

## Why Iterating `numSet` Instead of `nums` Matters

```kotlin
// CORRECT — iterate the set
for (n in numSet) { ... }

// ALSO CORRECT but wasteful — iterate original array
for (n in nums) {
    if (n - 1 !in numSet) { ... }  // duplicates cause repeated work
}
```

If `nums = [1, 1, 1, 2, 3]`, iterating `nums` would attempt to start a sequence
from `1` three times. Iterating `numSet` processes `1` exactly once.

**`n - 1 !in numSet`** — Kotlin's `in` operator on `HashSet` is O(1):
```kotlin
if (n - 1 !in numSet) { ... }
// Kotlin operator overloading — calls numSet.contains(n - 1) under the hood
// More readable than: if (!numSet.contains(n - 1))
```

**`nums.toHashSet()`** — deduplication and O(1) lookup in one call:
```kotlin
val numSet = nums.toHashSet()
// vs Java: Set<Integer> numSet = new HashSet<>();
//          for (int n : nums) numSet.add(n);
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| HashSet + start detection (Approach 1) | Follow-up constraint enforced, O(n) required |
| Sort + scan (Approach 2) | Readability matters, O(n log n) acceptable |

---

## Follow-up — Why can't we sort?

Sorting is O(n log n). The follow-up requires O(n). The HashSet approach
achieves this by:
1. O(n) to build the set
2. O(n) total across all `while` loops — each element is touched at most once
   as part of exactly one sequence (no element is a start of two sequences)

---

## Complexity

| | HashSet | Sort |
|---|---|---|
| **Time** | O(n) | O(n log n) |
| **Space** | O(n) | O(1) or O(n) depending on sort |

---

## Key Takeaway

> Put all numbers in a HashSet first. Then for each number, only start
> counting if `n - 1` is absent — that makes it the true start of a sequence.
> Walk forward while consecutive numbers exist. Each number is visited at most
> twice total, giving O(n). The `!in` check is the entire insight.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/longest-consecutive-sequence/)

---

{% include kotlin-dsa-nav.html %}
