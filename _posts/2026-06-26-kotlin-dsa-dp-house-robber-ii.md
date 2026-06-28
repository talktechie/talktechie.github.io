---
title: "1-D DP: House Robber II — Kotlin Solution"
date: 2026-06-26
categories: [Kotlin DSA, 1-D Dynamic Programming]
tags: [house-robber-ii, dynamic-programming, array, medium, leetcode, leetcode-213]
leetcode_number: 213
leetcode_url: https://leetcode.com/problems/house-robber-ii/
difficulty: Medium
permalink: /posts/kotlin-dsa-dp-house-robber-ii/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [213 — House Robber II](https://leetcode.com/problems/house-robber-ii/) |
| **Difficulty** | Medium |
| **Topic** | Dynamic Programming, Array |

> Same as House Robber, except the houses are arranged in a **circle** —
> house `0` and house `n-1` are now also adjacent to each other.

**Example:**
```
Input:  nums = [2,3,2]
Output: 3
Explanation: robbing house 0 AND house 2 isn't allowed (they're adjacent
             in the circle) — best is just house 1, value 3

Input:  nums = [1,2,3,1]
Output: 4
Explanation: rob houses 0 and 2 → 1+3=4 (still valid, since 0 and 2
             aren't the circular-adjacent pair here — only 0 and 3 are)
```

**Constraints:**
- `1 <= nums.length <= 100`
- `0 <= nums[i] <= 1000`

---

## Approach

The circular constraint means house `0` and house `n-1` can never
**both** be robbed. Rather than designing a new algorithm to handle
circularity directly, we can cleverly **reduce** this to two separate
calls of the already-solved linear House Robber problem.

**Key insight:** Any valid robbery plan either **excludes house 0** or
**excludes house n-1** (it can't include both endpoints, but it could
validly exclude either one, or both). So: run House Robber's algorithm
on the subarray **excluding the last house** (`nums[0..n-2]`), and
separately on the subarray **excluding the first house**
(`nums[1..n-1]`). The answer is the maximum of these two results — this
correctly covers every valid circular arrangement, since any valid plan
fits within at least one of these two linear sub-cases.

**Walk through `nums = [2,3,2]`:**
```
Case A — exclude last house: consider nums[0..1] = [2,3]
  House Robber on [2,3]: max(2,3) = 3

Case B — exclude first house: consider nums[1..2] = [3,2]
  House Robber on [3,2]: max(3,2) = 3

Answer: max(3, 3) = 3 ✓
```

---

## Kotlin Solution

### Approach 1 — Reduce to two linear House Robber calls (optimal)

```kotlin
fun rob(nums: IntArray): Int {
    if (nums.size == 1) return nums[0]   // single house — no circular conflict possible

    fun robLinear(houses: IntArray): Int {
        var prev2 = 0
        var prev1 = 0
        for (num in houses) {
            val current = maxOf(prev1, prev2 + num)
            prev2 = prev1
            prev1 = current
        }
        return prev1
    }

    val excludeLast = robLinear(nums.copyOfRange(0, nums.size - 1))
    val excludeFirst = robLinear(nums.copyOfRange(1, nums.size))

    return maxOf(excludeLast, excludeFirst)
}
```

### Approach 2 — Same idea, avoid array copying by passing index ranges

```kotlin
fun rob(nums: IntArray): Int {
    if (nums.size == 1) return nums[0]

    fun robRange(start: Int, end: Int): Int {
        var prev2 = 0
        var prev1 = 0
        for (i in start..end) {
            val current = maxOf(prev1, prev2 + nums[i])
            prev2 = prev1
            prev1 = current
        }
        return prev1
    }

    return maxOf(
        robRange(0, nums.size - 2),   // exclude last house
        robRange(1, nums.size - 1)    // exclude first house
    )
}
```

Avoids the overhead of `copyOfRange` allocating new arrays — instead,
the helper function operates directly on index ranges within the
original array.

---

## Why Excluding EITHER Endpoint (Not Both Simultaneously) Covers All Valid Plans

This is the key reduction insight worth understanding precisely. The
circular constraint only forbids robbing **both** house 0 and house
n-1 together — it says nothing about excluding both, or excluding just
one. So every valid robbery plan falls into (at least) one of two
buckets:

```kotlin
val excludeLast = robLinear(nums.copyOfRange(0, nums.size - 1))   // house n-1 is OFF the table
val excludeFirst = robLinear(nums.copyOfRange(1, nums.size))      // house 0 is OFF the table
```

- If a plan doesn't rob house `n-1`, it's a perfectly valid **linear**
  robbery plan over houses `0..n-2` — exactly what `excludeLast`
  computes optimally.
- If a plan doesn't rob house `0`, it's a valid linear plan over houses
  `1..n-1` — exactly what `excludeFirst` computes.
- A plan that robs **neither** endpoint is still correctly captured by
  both calculations (it's a valid candidate within either sub-range).
- The only plan **excluded** entirely from consideration is one that
  robs **both** endpoints — which is exactly the one truly forbidden by
  the circular constraint.

Taking the max of both linear results therefore always finds the true
circular-constrained optimum.

**Why the `n == 1` check is necessary:** with only one house, there's no
circular adjacency conflict to begin with (a single house can't be
adjacent to itself) — `nums.copyOfRange(0, 0)` would otherwise produce an
empty array, leading to an incorrect answer of `0` instead of `nums[0]`.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Copy subarrays (Approach 1) | Clearer to read, fine for this problem's small constraint (n ≤ 100) |
| Index ranges, no copying (Approach 2) | Avoids extra allocations, marginally more efficient |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — two linear passes, each O(n) |
| **Space** | O(1) — Approach 2; O(n) — Approach 1, due to array copying |

---

## Key Takeaway

> When a circular constraint complicates an otherwise-solved linear
> problem, look for a way to **reduce** it to multiple linear sub-cases
> rather than designing an entirely new algorithm. Here, "can't rob both
> endpoints" decomposes cleanly into "exclude the last house" OR
> "exclude the first house" — two independent calls to the already-solved
> House Robber, with the final answer being whichever achieves more.
> This reduction technique — turning a harder constrained variant into
> multiple instances of an easier already-solved problem — is broadly
> useful whenever a new constraint only restricts a small, specific
> subset of otherwise-valid configurations.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/house-robber-ii/)

---

{% include kotlin-dsa-nav.html %}
