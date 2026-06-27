---
title: "Backtracking: Subsets II — Kotlin Solution"
date: 2026-06-24
categories: [Kotlin DSA, Backtracking]
tags: [subsets-ii, backtracking, array, medium, leetcode, leetcode-90]
leetcode_number: 90
leetcode_url: https://leetcode.com/problems/subsets-ii/
difficulty: Medium
permalink: /posts/kotlin-dsa-backtracking-subsets-ii/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [90 — Subsets II](https://leetcode.com/problems/subsets-ii/) |
| **Difficulty** | Medium |
| **Topic** | Backtracking, Array |

> Given an integer array `nums` that **may contain duplicates**, return
> all possible subsets (the power set), with **no duplicate subsets** in
> the output.

**Example:**
```
Input:  nums = [1,2,2]
Output: [[],[1],[1,2],[1,2,2],[2],[2,2]]
Explanation: note [1,2] only appears once, even though there are two
             ways to "pick the first 2" — they're the same subset

Input:  nums = [0]
Output: [[],[0]]
```

**Constraints:**
- `1 <= nums.length <= 10`
- `-10 <= nums[i] <= 10`

---

## Approach

This merges two ideas already seen in this phase: Subsets' "advance index,
record on every call" template, combined with Combination Sum II's
"sort first, skip duplicates at the same recursion depth" trick.

**Key insight:** Sort `nums` so duplicate values become adjacent. Use the
standard "advance the start index" backtracking template from Subsets,
but at each level of the loop, skip over a value if it's identical to the
**previous** value tried **at that same loop level** (not globally) —
this is exactly the same duplicate-skip condition as Combination Sum II.

**Walk through `nums = [1,2,2]` (already sorted):**
```
backtrack(start=0, current=[]):
  record [] (current state on entry)

  i=0, nums[0]=1: include → current=[1]
    backtrack(start=1, current=[1]): record [1]
      i=1, nums[1]=2: include → current=[1,2]
        backtrack(start=2, current=[1,2]): record [1,2]
          i=2, nums[2]=2: include → current=[1,2,2]
            backtrack(start=3, ...): record [1,2,2]
          undo
      undo → current=[1]
      i=2, nums[2]=2: SAME as nums[1] at this same loop level → SKIP
    undo → current=[]

  i=1, nums[1]=2: include → current=[2]
    backtrack(start=2, current=[2]): record [2]
      i=2, nums[2]=2: include → current=[2,2]
        record [2,2]
    undo

  i=2, nums[2]=2: SAME as nums[1] at this loop level → SKIP

Result: [[],[1],[1,2],[1,2,2],[2],[2,2]] ✓ (no duplicates)
```

---

## Kotlin Solution

### Approach 1 — Sort, then standard subsets backtracking with duplicate-skip (optimal)

```kotlin
fun subsetsWithDup(nums: IntArray): List<List<Int>> {
    nums.sort()
    val result = mutableListOf<List<Int>>()
    val current = mutableListOf<Int>()

    fun backtrack(start: Int) {
        result.add(current.toList())   // every state is a valid subset

        for (i in start until nums.size) {
            if (i > start && nums[i] == nums[i - 1]) continue   // skip duplicates at this depth

            current.add(nums[i])
            backtrack(i + 1)
            current.removeAt(current.lastIndex)
        }
    }

    backtrack(0)
    return result
}
```

### Approach 2 — Generate all subsets including duplicates, dedupe with a Set

```kotlin
fun subsetsWithDup(nums: IntArray): List<List<Int>> {
    nums.sort()
    val result = mutableSetOf<List<Int>>()
    val current = mutableListOf<Int>()

    fun backtrack(index: Int) {
        if (index == nums.size) {
            result.add(current.toList())
            return
        }

        current.add(nums[index])
        backtrack(index + 1)
        current.removeAt(current.lastIndex)

        backtrack(index + 1)   // exclude branch
    }

    backtrack(0)
    return result.toList()
}
```

Generates every combination of include/exclude decisions (same as
Subsets' Approach 1), then relies on a `Set<List<Int>>` to silently
discard duplicates. Conceptually simpler, but wastes work generating and
then throwing away duplicate subsets — the pruning approach (Approach 1)
never generates them in the first place.

---

## Why This Skip Condition Is Identical to Combination Sum II's

```kotlin
if (i > start && nums[i] == nums[i - 1]) continue
```

The reasoning is exactly the same as in Combination Sum II: at any given
recursion depth, once we've fully explored every subset that **includes**
`nums[i-1]` at this position, trying an identical value `nums[i]` at the
same position would only retrace subsets we've already generated.
Skipping it prevents that duplicate work entirely — both in terms of
correctness (no duplicate output) and efficiency (no wasted recursive
calls exploring branches we'd just discard).

**Why sorting is a prerequisite for this trick to work at all:**
without sorting, equal values might not be adjacent, and the
`nums[i] == nums[i-1]` check would miss most actual duplicates (or
worse, accidentally skip valid non-duplicate branches if unsorted data
happened to place different equal-valued duplicates non-adjacently).

**Why recording happens at the top of `backtrack`, not only at the
base case (unlike Combination Sum II, which records only on an exact
sum match):** every partial state here is itself a valid subset — there's
no "target" to hit, so every node in the recursion tree, not just the
leaves, contributes to the final answer.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Sort + skip duplicates (Approach 1) | Always — never generates duplicate work, the standard efficient solution |
| Generate all + Set dedup (Approach 2) | Simpler to write quickly, acceptable for very small inputs only |

---

## Complexity

| | Approach 1 | Approach 2 |
|---|---|---|
| **Time** | O(n × 2ⁿ) | O(n × 2ⁿ) generated, with extra Set overhead for deduplication |
| **Space** | O(n × 2ⁿ) | O(n × 2ⁿ) |

---

## Key Takeaway

> Subsets II is a direct merge of two patterns already covered in this
> phase: the "record on every call, advance the index" backtracking
> shape from Subsets, combined with the "sort first, then skip adjacent
> duplicates at the same recursion depth" trick from Combination Sum II.
> Once both of those individual patterns are internalized, this problem
> requires no new insight — just recognizing which combination of
> previously-seen templates applies.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/subsets-ii/)

---

{% include kotlin-dsa-nav.html %}
