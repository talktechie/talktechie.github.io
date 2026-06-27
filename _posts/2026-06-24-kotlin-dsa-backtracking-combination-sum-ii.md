---
title: "Backtracking: Combination Sum II — Kotlin Solution"
date: 2026-06-24
categories: [Kotlin DSA, Backtracking]
tags: [combination-sum-ii, backtracking, array, medium, leetcode, leetcode-40]
leetcode_number: 40
leetcode_url: https://leetcode.com/problems/combination-sum-ii/
difficulty: Medium
permalink: /posts/kotlin-dsa-backtracking-combination-sum-ii/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [40 — Combination Sum II](https://leetcode.com/problems/combination-sum-ii/) |
| **Difficulty** | Medium |
| **Topic** | Backtracking, Array |

> Given a collection of candidate numbers `candidates` (which **may
> contain duplicates**) and a `target`, return all unique combinations
> where the numbers sum to `target`. **Each number may only be used
> once** per combination. The solution set must not contain duplicate
> combinations.

**Example:**
```
Input:  candidates = [10,1,2,7,6,1,5], target = 8
Output: [[1,1,6],[1,2,5],[1,7],[2,6]]
Explanation: note there are two 1's in the input — both can be used
             together (as in [1,1,6]), but we must not produce the same
             combination twice just because of which "1" was picked

Input:  candidates = [2,5,2,1,2], target = 5
Output: [[1,2,2],[5]]
```

**Constraints:**
- `1 <= candidates.length <= 100`
- `1 <= candidates[i] <= 50`
- `1 <= target <= 30`

---

## Approach

This combines two changes on top of Combination Sum (LC 39): each element
can be used **at most once** (so we advance the index, `i + 1`, not `i`),
and the input may contain **duplicate values**, which means we need an
explicit mechanism to avoid generating the same combination multiple
times just because duplicate values exist at different positions.

**Key insight — sort first, then skip duplicate values at the same
recursion depth:** After sorting, identical values become adjacent. At
any given point in the loop, if we've already explored taking
`candidates[i]`, taking an identical `candidates[i+1]` value **at the
same recursive depth** would only ever produce a combination we've
already generated — so skip it.

**Walk through `candidates = [1,1,2,5,6,7,10]` (sorted), target = 8:**
```
backtrack(start=0, remaining=8):
  i=0, candidates[0]=1: include it → current=[1], remaining=7
    backtrack(start=1, remaining=7):
      i=1, candidates[1]=1: include → current=[1,1], remaining=6
        ... eventually finds [1,1,6] ✓
      i=2, candidates[2]=2: include → current=[1,2], remaining=5
        ... eventually finds [1,2,5] ✓
      ... continues ...
      i=5, candidates[5]=7: include → current=[1,7], remaining=0 → record [1,7] ✓
  i=1, candidates[1]=1: SAME VALUE as candidates[0], and we're at the
       SAME recursion depth (both are choice #1 in the top-level loop)
       → SKIP to avoid duplicate combinations starting with "1"
  i=2, candidates[2]=2: include → current=[2], remaining=6
    ... eventually finds [2,6] ✓

Result: [[1,1,6],[1,2,5],[1,7],[2,6]] ✓
```

---

## Kotlin Solution

### Approach 1 — Sort, skip duplicates at the same depth, advance index (optimal)

```kotlin
fun combinationSum2(candidates: IntArray, target: Int): List<List<Int>> {
    candidates.sort()
    val result = mutableListOf<List<Int>>()
    val current = mutableListOf<Int>()

    fun backtrack(start: Int, remaining: Int) {
        if (remaining == 0) {
            result.add(current.toList())
            return
        }

        for (i in start until candidates.size) {
            if (candidates[i] > remaining) break   // sorted — prune the rest of the loop too

            // Skip duplicate values at this SAME recursion depth (but only
            // if it's not the very first choice being tried at this depth)
            if (i > start && candidates[i] == candidates[i - 1]) continue

            current.add(candidates[i])
            backtrack(i + 1, remaining - candidates[i])   // i+1 — each element used once
            current.removeAt(current.lastIndex)
        }
    }

    backtrack(0, target)
    return result
}
```

### Approach 2 — Same idea, using a frequency map instead of skip-checking the sorted array

```kotlin
fun combinationSum2(candidates: IntArray, target: Int): List<List<Int>> {
    val counts = candidates.toList().groupingBy { it }.eachCount()
    val uniqueValues = counts.keys.sorted()
    val result = mutableListOf<List<Int>>()
    val current = mutableListOf<Int>()

    fun backtrack(startIdx: Int, remaining: Int) {
        if (remaining == 0) {
            result.add(current.toList())
            return
        }
        if (startIdx == uniqueValues.size || remaining < 0) return

        for (idx in startIdx until uniqueValues.size) {
            val value = uniqueValues[idx]
            if (value > remaining) break

            val available = counts[value]!!
            val maxUse = minOf(available, remaining / value)

            for (count in 1..maxUse) {
                repeat(count) { current.add(value) }
                backtrack(idx + 1, remaining - value * count)
                repeat(count) { current.removeAt(current.lastIndex) }
            }
        }
    }

    backtrack(0, target)
    return result
}
```

Groups duplicate values upfront, then explicitly tries using each
distinct value `0` to `maxUse` times. More code, but conceptually
separates "which distinct values to use" from "how many of each" —
useful if a problem later needs explicit control over repetition counts.

---

## Why `if (i > start && ...)` Is the Precise Duplicate-Skip Condition

This condition is subtle and worth understanding carefully — getting it
slightly wrong (e.g., using `i > 0` instead of `i > start`) breaks
correctness:

```kotlin
if (i > start && candidates[i] == candidates[i - 1]) continue
```

- `i > start` ensures we're **not** looking at the very first option
  being considered at this recursion depth — the first occurrence of any
  value is always allowed to be tried.
- `candidates[i] == candidates[i-1]` checks if this is a repeat of the
  immediately preceding value.

Together: "if this isn't the first choice I'm trying right now, and it's
the same value as the previous choice I already fully explored, skip it
— I'd just be regenerating combinations I already produced."

**Why this is different from skipping duplicates in the *output*,**
which would require expensive de-duplication after the fact (e.g.,
storing results in a `Set` and comparing entire lists) — skipping at the
recursion level prevents the duplicate branches from ever being explored
in the first place, which is both correct and far more efficient.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Sort + skip-adjacent-duplicates (Approach 1) | Always — concise, the standard accepted solution |
| Frequency map + explicit counts (Approach 2) | Want explicit control over how many of each value to use, useful for related variant problems |

---

## Complexity

| | |
|---|---|
| **Time** | O(2ⁿ) worst case — bounded further in practice by target and pruning |
| **Space** | O(n) — recursion depth and current combination storage |

---

## Key Takeaway

> Combination Sum II layers two changes onto Combination Sum: advance the
> index (`i + 1`) since each element is used at most once, and add a
> duplicate-skip check at each recursion depth to avoid regenerating the
> same combination through different occurrences of equal values. The
> `i > start && candidates[i] == candidates[i-1]` pattern for skipping
> duplicates **after sorting** is one of the most broadly reusable tricks
> in backtracking — it reappears in Subsets II and Permutations II-style
> problems whenever duplicate input values threaten to produce duplicate
> outputs.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/combination-sum-ii/)

---

{% include kotlin-dsa-nav.html %}
