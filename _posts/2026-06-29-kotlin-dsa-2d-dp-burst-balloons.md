---
title: "2-D DP: Burst Balloons — Kotlin Solution"
date: 2026-06-29
categories: [Kotlin DSA, 2-D Dynamic Programming]
tags: [burst-balloons, dynamic-programming, array, interval-dp, hard, leetcode, leetcode-312]
leetcode_number: 312
leetcode_url: https://leetcode.com/problems/burst-balloons/
difficulty: Hard
permalink: /posts/kotlin-dsa-2d-dp-burst-balloons/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [312 — Burst Balloons](https://leetcode.com/problems/burst-balloons/) |
| **Difficulty** | Hard |
| **Topic** | Dynamic Programming, Array, Interval DP |

> Given `n` balloons with values `nums`, bursting balloon `i` earns
> `nums[left] * nums[i] * nums[right]` coins, where `left`/`right` are
> the **currently adjacent** balloons (shrinking as others are burst).
> Return the **maximum coins** obtainable by bursting all balloons.

**Example:**
```
Input:  nums = [3,1,5,8]
Output: 167
Explanation: burst 1 (3*1*5=15), burst 5 (3*5*8=120), burst 3 (1*3*8=24),
             burst 8 (1*8*1=8) → 15+120+24+8=167

Input:  nums = [1,5]
Output: 10
```

**Constraints:**
- `1 <= nums.length <= 300`
- `0 <= nums[i] <= 100`

---

## Approach

This introduces **Interval DP** — a different 2-D shape than the
two-string grids seen so far. Here, both DP dimensions represent
**boundaries of a sub-range** within a single array, not two separate
strings.

**Key insight — think about which balloon bursts LAST within a range,
not first:** trying to decide "which balloon to burst first" is messy,
because bursting changes who's adjacent to whom for every subsequent
choice. Instead, flip the thinking: for any range `(left, right)`,
consider each balloon `k` as the **last** one burst within that range.
If `k` bursts last, every other balloon in `(left, k)` and `(k, right)`
must have **already** been independently fully burst — meaning `nums[left]`
and `nums[right]` (the original array's boundary values, padded with
1's) are still `k`'s neighbors at the moment it bursts, regardless of
what happened inside those sub-ranges. This eliminates the "adjacency
changes over time" complexity entirely.

`dp[left][right]` = max coins from fully bursting all balloons strictly
between `left` and `right` (exclusive boundaries, which are virtual
padding values of 1 at both ends of the original array).

**Walk through `nums = [3,1,5,8]` padded to `[1,3,1,5,8,1]`** (1's are
virtual boundary balloons), computing a small piece:
```
For range (0,2) [meaning balloons strictly between padded-index 0 and 2,
which is just original index 0, value 3]:
  only one choice: k=1 (value 3) bursts last
  coins = padded[0] * padded[1] * padded[2] + dp[0][1] + dp[1][2]
        = 1 * 3 * 1 + 0 + 0 = 3

(Building up through all ranges eventually yields the full answer for
the entire array, 167, matching the expected output.)
```

---

## Kotlin Solution

### Approach 1 — Interval DP, balloon `k` as the LAST burst in each range (optimal)

```kotlin
fun maxCoins(nums: IntArray): Int {
    val n = nums.size
    val padded = IntArray(n + 2)
    padded[0] = 1
    padded[n + 1] = 1
    for (i in nums.indices) padded[i + 1] = nums[i]

    val dp = Array(n + 2) { IntArray(n + 2) }

    // length = size of the OPEN interval (left, right) we're solving for
    for (length in 1..n) {
        for (left in 0..n - length) {
            val right = left + length + 1

            for (k in left + 1 until right) {
                val coins = padded[left] * padded[k] * padded[right] + dp[left][k] + dp[k][right]
                dp[left][right] = maxOf(dp[left][right], coins)
            }
        }
    }

    return dp[0][n + 1]
}
```

### Approach 2 — Top-down memoization (same recurrence, recursive framing)

```kotlin
fun maxCoins(nums: IntArray): Int {
    val n = nums.size
    val padded = IntArray(n + 2)
    padded[0] = 1
    padded[n + 1] = 1
    for (i in nums.indices) padded[i + 1] = nums[i]

    val memo = Array(n + 2) { IntArray(n + 2) { -1 } }

    fun dp(left: Int, right: Int): Int {
        if (right - left < 2) return 0   // no balloons strictly between them

        if (memo[left][right] != -1) return memo[left][right]

        var best = 0
        for (k in left + 1 until right) {
            val coins = padded[left] * padded[k] * padded[right] + dp(left, k) + dp(k, right)
            best = maxOf(best, coins)
        }

        memo[left][right] = best
        return best
    }

    return dp(0, n + 1)
}
```

---

## Why Thinking About the LAST Burst (Not the First) Is the Crucial Insight

This reversal in perspective is what makes the entire problem tractable,
and it's worth dwelling on why "first burst" thinking fails:

If we tried to decide which balloon bursts **first** within a range, the
**remaining** balloons' adjacency relationships would immediately become
tangled — bursting one balloon changes who's next to whom for every
future choice in a way that's hard to cleanly decompose into independent
subproblems.

But if we instead ask "which balloon bursts **last**," something
beautiful happens:

```kotlin
val coins = padded[left] * padded[k] * padded[right] + dp[left][k] + dp[k][right]
```

If balloon `k` is the **last** one standing within the open range
`(left, right)`, then at the moment it bursts, its neighbors are
**guaranteed** to still be `padded[left]` and `padded[right]` — the
fixed boundary values — because *every other* balloon in this range has
**already** been fully burst by then (via the independently-computed
`dp[left][k]` and `dp[k][right]` subproblems). This completely
decouples the choice of `k` from whatever order the *other* balloons in
each half get burst — those are separate, already-solved subproblems.

**Why the padding with virtual `1`s at both ends matters:** it lets the
very first and very last *real* balloons in the array still have valid
"neighbors" to multiply against (per the problem's implicit rule that
an out-of-bounds neighbor contributes a multiplier of 1), without
needing special-case boundary logic.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Bottom-up interval DP (Approach 1) | Standard, iterative, the typical competitive-programming answer |
| Top-down memoization (Approach 2) | Prefer recursive framing — same complexity, sometimes easier to derive correctly on the first attempt |

---

## Complexity

| | |
|---|---|
| **Time** | O(n³) — O(n²) ranges, O(n) choices of `k` for each |
| **Space** | O(n²) — the DP table |

---

## Key Takeaway

> Burst Balloons introduces **Interval DP**, where both dimensions of
> the DP table represent the boundaries of a sub-range within a single
> sequence, rather than positions in two separate strings. The critical
> insight — deciding what happens **last** within a range, rather than
> first — is a powerful and frequently-applicable trick whenever an
> action's effect (here, "becoming adjacent") changes based on the order
> operations are performed: anchoring on the *last* event within a
> range often cleanly decouples the subproblem in ways that anchoring on
> the *first* event cannot.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/burst-balloons/)

---

{% include kotlin-dsa-nav.html %}
