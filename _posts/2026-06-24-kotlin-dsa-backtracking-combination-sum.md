---
title: "Backtracking: Combination Sum — Kotlin Solution"
date: 2026-06-24
categories: [Kotlin DSA, Backtracking]
tags: [combination-sum, backtracking, array, medium, leetcode, leetcode-39]
leetcode_number: 39
leetcode_url: https://leetcode.com/problems/combination-sum/
difficulty: Medium
permalink: /posts/kotlin-dsa-backtracking-combination-sum/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [39 — Combination Sum](https://leetcode.com/problems/combination-sum/) |
| **Difficulty** | Medium |
| **Topic** | Backtracking, Array |

> Given an array of **distinct** integers `candidates` and a `target`,
> return all unique combinations where the chosen numbers sum to
> `target`. The **same number may be used unlimited times**. Return
> combinations in any order, with no duplicate combinations.

**Example:**
```
Input:  candidates = [2,3,6,7], target = 7
Output: [[2,2,3],[7]]
Explanation: 2+2+3=7, and 7 itself works — both valid, each number
             reused as needed

Input:  candidates = [2,3,5], target = 8
Output: [[2,2,2,2],[2,3,3],[3,5]]
```

**Constraints:**
- `1 <= candidates.length <= 30`
- `2 <= candidates[i] <= 40`
- All elements of `candidates` are distinct
- `1 <= target <= 40`

---

## Approach

Building on Subsets' include/exclude template, with two new twists:
numbers can be **reused indefinitely**, and we need combinations summing
to an exact **target** — which gives us a natural way to **prune**
branches early.

**Key insight:** At each recursive call, try including the current
candidate **again** (since reuse is allowed) by recursing **without**
advancing the index, or move on to the next candidate by advancing the
index. Track the running sum; if it ever exceeds `target`, prune
immediately — no need to keep adding more positive numbers to something
already too large.

**Walk through `candidates = [2,3,6,7], target = 7`:**
```
backtrack(index=0, current=[], remaining=7):
  try candidates[0]=2: remaining becomes 5 → current=[2]
    try candidates[0]=2 again: remaining becomes 3 → current=[2,2]
      try candidates[0]=2 again: remaining becomes 1 → current=[2,2,2]
        try 2 again: remaining=-1 → PRUNE (negative)
        try 3: remaining=1-3=-2 → PRUNE
        try 6,7: also prune (too big)
        → backtrack, remove last 2 → current=[2,2]
      try candidates[1]=3: remaining=3-3=0 → current=[2,2,3] → SUM MATCHES! record [2,2,3] ✓
      try candidates[2]=6: remaining=3-6<0 → prune
      try candidates[3]=7: prune
    try candidates[1]=3 (skip index 0 entirely now): remaining=5-3=2 → current=[2,3]
      ... continues, eventually no valid completion found through this branch
  try candidates[1]=3 (don't use 2 at all): remaining=7-3=4 → current=[3]
    ... continues exploring ...
  try candidates[3]=7: remaining=7-7=0 → current=[7] → SUM MATCHES! record [7] ✓

Result: [[2,2,3],[7]] ✓
```

---

## Kotlin Solution

### Approach 1 — Backtracking with same-index reuse and early pruning (optimal)

```kotlin
fun combinationSum(candidates: IntArray, target: Int): List<List<Int>> {
    val result = mutableListOf<List<Int>>()
    val current = mutableListOf<Int>()

    fun backtrack(start: Int, remaining: Int) {
        if (remaining == 0) {
            result.add(current.toList())
            return
        }
        if (remaining < 0) return   // prune — overshot the target

        for (i in start until candidates.size) {
            current.add(candidates[i])
            backtrack(i, remaining - candidates[i])   // note: i, NOT i+1 — allows reuse
            current.removeAt(current.lastIndex)
        }
    }

    backtrack(0, target)
    return result
}
```

### Approach 2 — Sort first, prune more aggressively using sorted order

```kotlin
fun combinationSum(candidates: IntArray, target: Int): List<List<Int>> {
    candidates.sort()
    val result = mutableListOf<List<Int>>()
    val current = mutableListOf<Int>()

    fun backtrack(start: Int, remaining: Int) {
        if (remaining == 0) {
            result.add(current.toList())
            return
        }

        for (i in start until candidates.size) {
            if (candidates[i] > remaining) break   // sorted — everything after is too big too

            current.add(candidates[i])
            backtrack(i, remaining - candidates[i])
            current.removeAt(current.lastIndex)
        }
    }

    backtrack(0, target)
    return result
}
```

Sorting first lets us `break` out of the loop entirely the moment a
candidate exceeds the remaining target — since everything after it in
sorted order is even larger, there's no point checking them at all. This
prunes more branches earlier than Approach 1's per-call negative check.

---

## Why Passing `i` (Not `i + 1`) Is What Allows Reuse

This is the one-character difference that distinguishes Combination Sum
from Subsets-style problems where each element is used at most once:

```kotlin
backtrack(i, remaining - candidates[i])
// passing "i" means candidates[i] is still eligible to be chosen AGAIN
// in the next recursive call — this is what permits unlimited reuse
```

Compare this to Combination Sum II (a later problem in this phase), which
passes `i + 1` instead — because there, each *original* element can only
be used once (though the input array may contain duplicate values).

**Why the `for` loop starts at `start`, not `0`:**
```kotlin
for (i in start until candidates.size) { ... }
```
This prevents generating the same combination in different orders — e.g.,
without this restriction, `[2,3]` and `[3,2]` would both be generated as
separate (but actually identical, just reordered) combinations. Starting
the loop at `start` enforces a canonical "non-decreasing index" order,
naturally avoiding duplicate combinations.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Basic pruning (Approach 1) | Works correctly, slightly less optimized |
| Sort + early break (Approach 2) | Better pruning, fewer wasted recursive calls — preferred in practice |

---

## Complexity

| | |
|---|---|
| **Time** | O(2^target) worst case — bounded by how many combinations can sum to target |
| **Space** | O(target / min(candidates)) — maximum recursion depth |

---

## Key Takeaway

> Combination Sum extends the Subsets template with two changes: passing
> the same index forward (not advancing it) permits reusing an element,
> and tracking a running "remaining target" gives a natural, powerful
> pruning condition — stop exploring the instant the partial sum can't
> possibly reach the target anymore. Sorting the candidates first turns
> that prune into an early `break`, cutting off entire unexplored ranges
> of the loop rather than checking and rejecting them one at a time.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/combination-sum/)

---

{% include kotlin-dsa-nav.html %}
