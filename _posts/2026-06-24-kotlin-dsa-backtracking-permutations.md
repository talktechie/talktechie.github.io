---
title: "Backtracking: Permutations — Kotlin Solution"
date: 2026-06-24
categories: [Kotlin DSA, Backtracking]
tags: [permutations, backtracking, array, medium, leetcode, leetcode-46]
leetcode_number: 46
leetcode_url: https://leetcode.com/problems/permutations/
difficulty: Medium
permalink: /posts/kotlin-dsa-backtracking-permutations/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [46 — Permutations](https://leetcode.com/problems/permutations/) |
| **Difficulty** | Medium |
| **Topic** | Backtracking, Array |

> Given an array `nums` of **distinct** integers, return **all possible
> permutations** (every possible ordering), in any order.

**Example:**
```
Input:  nums = [1,2,3]
Output: [[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]

Input:  nums = [0,1]
Output: [[0,1],[1,0]]
```

**Constraints:**
- `1 <= nums.length <= 6`
- `-10 <= nums[i] <= 10`
- All elements of `nums` are unique

---

## Approach

Unlike Subsets or Combination Sum, **order matters here** — `[1,2,3]` and
`[3,2,1]` are different valid answers. This means the choice at every
step is "which **unused** element goes next," rather than "include or
exclude this specific index."

**Key insight:** Track which elements have already been placed in the
current permutation (via a `used` boolean array, or by removing them
from a pool). At each recursive step, try **every** unused element as the
next position — recurse, then mark it unused again before trying the
next option.

**Walk through `nums = [1,2,3]`** (partial trace):
```
backtrack(current=[]):
  try 1 (unused) → current=[1], used={1}
    try 2 (unused) → current=[1,2], used={1,2}
      try 3 (unused) → current=[1,2,3] → length matches → record [1,2,3] ✓
      undo 3
    undo 2 → current=[1]
    try 3 (unused) → current=[1,3], used={1,3}
      try 2 (unused) → current=[1,3,2] → record [1,3,2] ✓
  undo 1 → current=[]
  try 2 (unused) → current=[2] ...
    (eventually produces [2,1,3] and [2,3,1])
  try 3 (unused) → current=[3] ...
    (eventually produces [3,1,2] and [3,2,1])

Result: all 6 permutations of [1,2,3] ✓
```

---

## Kotlin Solution

### Approach 1 — Backtracking with a `used` boolean array (optimal, most instructive)

```kotlin
fun permute(nums: IntArray): List<List<Int>> {
    val result = mutableListOf<List<Int>>()
    val current = mutableListOf<Int>()
    val used = BooleanArray(nums.size)

    fun backtrack() {
        if (current.size == nums.size) {
            result.add(current.toList())
            return
        }

        for (i in nums.indices) {
            if (used[i]) continue   // already placed this element

            used[i] = true
            current.add(nums[i])

            backtrack()

            current.removeAt(current.lastIndex)   // undo
            used[i] = false                         // undo
        }
    }

    backtrack()
    return result
}
```

### Approach 2 — Swap-based in-place permutation (no extra `used` array)

```kotlin
fun permute(nums: IntArray): List<List<Int>> {
    val result = mutableListOf<List<Int>>()

    fun backtrack(start: Int) {
        if (start == nums.size) {
            result.add(nums.toList())
            return
        }

        for (i in start until nums.size) {
            // Swap the element at i into the "start" position
            nums[start] = nums[i].also { nums[i] = nums[start] }

            backtrack(start + 1)

            // Swap back — undo
            nums[start] = nums[i].also { nums[i] = nums[start] }
        }
    }

    backtrack(0)
    return result
}
```

A clever alternative: instead of tracking "used" status separately,
physically swap candidates into the current position, recurse on the
remainder, then swap back. No extra boolean array needed.

---

## Why a `used` Array (or Swapping) Replaces the "Index" Concept From Subsets

In Subsets and Combination Sum, the loop always advances **forward**
through the array (`i+1` or similar), since order didn't matter and we
only needed to decide inclusion. Here, **any** remaining element could go
in **any** remaining position — there's no single "current index" to
advance past. We need to track availability explicitly:

```kotlin
for (i in nums.indices) {
    if (used[i]) continue
    // every unused index is a valid next choice, regardless of position
}
```

**Why the swap-based approach (Approach 2) avoids extra space:** by
swapping the chosen element into the `start` position and recursing on
`start + 1`, everything from index `0` to `start-1` represents the
"decided" prefix, and everything from `start` onward is the "still
available" pool — no separate `used` tracking structure needed, at the
cost of temporarily mutating `nums` itself (always restored before
returning).

```kotlin
nums[start] = nums[i].also { nums[i] = nums[start] }   // Kotlin swap idiom
backtrack(start + 1)
nums[start] = nums[i].also { nums[i] = nums[start] }   // swap back — undo
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| `used` boolean array (Approach 1) | Clearest to explain, doesn't mutate the input array |
| Swap-based (Approach 2) | Want O(1) extra tracking space, comfortable with in-place mutation + restoration |

---

## Complexity

| | |
|---|---|
| **Time** | O(n × n!) — n! permutations, each O(n) to build/copy |
| **Space** | O(n) — recursion depth and tracking structures (excluding output) |

---

## Key Takeaway

> Permutations introduces a new backtracking shape: instead of advancing
> through fixed positions deciding include/exclude, every recursive call
> chooses from **whatever remains unused**, in any order. Tracking
> "what's still available" — via a boolean array or in-place swapping —
> replaces the simple index-advancement seen in Subsets and Combination
> Sum. This "choose from the remaining pool" pattern, paired with
> sorting and a duplicate-skip check, is exactly what Permutations II
> (a common follow-up problem) builds on next.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/permutations/)

---

{% include kotlin-dsa-nav.html %}
