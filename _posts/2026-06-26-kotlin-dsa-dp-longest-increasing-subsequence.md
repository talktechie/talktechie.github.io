---
title: "1-D DP: Longest Increasing Subsequence — Kotlin Solution"
date: 2026-06-26
categories: [Kotlin DSA, 1-D Dynamic Programming]
tags: [longest-increasing-subsequence, dynamic-programming, binary-search, array, medium, leetcode, leetcode-300]
leetcode_number: 300
leetcode_url: https://leetcode.com/problems/longest-increasing-subsequence/
difficulty: Medium
permalink: /posts/kotlin-dsa-dp-longest-increasing-subsequence/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [300 — Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/) |
| **Difficulty** | Medium |
| **Topic** | Dynamic Programming, Binary Search, Array |

> Given an integer array `nums`, return the length of the **longest
> strictly increasing subsequence** (not necessarily contiguous).

**Example:**
```
Input:  nums = [10,9,2,5,3,7,101,18]
Output: 4
Explanation: the LIS is [2,3,7,101] (or [2,3,7,18])

Input:  nums = [0,1,0,3,2,3]
Output: 4
Explanation: [0,1,2,3]
```

**Constraints:**
- `1 <= nums.length <= 2500`
- `-10⁴ <= nums[i] <= 10⁴`
- **Follow-up:** Can you do it in O(n log n)?

---

## Approach

**Key insight (O(n²) DP):** Define `dp[i]` as the length of the longest
increasing subsequence **ending exactly at** index `i`. For each `i`,
check every earlier index `j < i` — if `nums[j] < nums[i]`, then `i`
could extend whatever subsequence ends at `j`, giving a candidate length
of `dp[j] + 1`. Take the best such extension across all valid `j`.

**Key insight (O(n log n), the follow-up answer):** Maintain a
**`tails`** array, where `tails[k]` holds the smallest possible **tail
value** of any increasing subsequence of length `k+1` found so far. For
each new number, binary search `tails` for where it belongs — either
extending the longest subsequence found so far, or improving (lowering)
the tail value of some shorter subsequence, which keeps future
extensions more flexible.

**Walk through `nums = [10,9,2,5,3,7,101,18]` using the O(n log n)
approach:**
```
tails = []

process 10: tails=[10]
process 9:  9 < 10 → replace → tails=[9]
process 2:  2 < 9  → replace → tails=[2]
process 5:  5 > 2  → append  → tails=[2,5]
process 3:  3 fits between 2 and 5 → replace 5 → tails=[2,3]
process 7:  7 > 3  → append  → tails=[2,3,7]
process 101: 101 > 7 → append → tails=[2,3,7,101]
process 18: 18 fits between 7 and 101 → replace 101 → tails=[2,3,7,18]

Final tails length = 4 ✓ (the VALUES in tails aren't necessarily a real
subsequence, but its LENGTH always equals the true LIS length)
```

---

## Kotlin Solution

### Approach 1 — O(n²) DP, try every earlier index as a predecessor

```kotlin
fun lengthOfLIS(nums: IntArray): Int {
    val n = nums.size
    val dp = IntArray(n) { 1 }   // every single element is a subsequence of length 1

    for (i in 1 until n) {
        for (j in 0 until i) {
            if (nums[j] < nums[i]) {
                dp[i] = maxOf(dp[i], dp[j] + 1)
            }
        }
    }

    return dp.max()
}
```

### Approach 2 — O(n log n), binary search on a "tails" array (optimal, the follow-up answer)

```kotlin
fun lengthOfLIS(nums: IntArray): Int {
    val tails = mutableListOf<Int>()

    for (num in nums) {
        var left = 0
        var right = tails.size

        // Binary search for the first index where tails[idx] >= num
        while (left < right) {
            val mid = left + (right - left) / 2
            if (tails[mid] < num) left = mid + 1 else right = mid
        }

        if (left == tails.size) {
            tails.add(num)        // num extends the longest subsequence found so far
        } else {
            tails[left] = num     // num improves (lowers) an existing tail value
        }
    }

    return tails.size
}
```

---

## Why `tails` Isn't a Real Subsequence, But Its Length Still Equals the True LIS Length

This is the most counter-intuitive part of the O(n log n) approach, and
worth understanding carefully. `tails[k]` represents "the smallest tail
value achievable among all increasing subsequences of length `k+1`
processed so far" — it does **not** claim that `tails` itself, read as a
sequence, is a valid increasing subsequence that actually appears in
`nums`.

```kotlin
if (left == tails.size) {
    tails.add(num)        // num is bigger than every existing tail —
                            // it extends the LONGEST subsequence by one
} else {
    tails[left] = num      // num REPLACES a larger tail value at this length —
                            // keeping a smaller tail value gives future
                            // numbers an easier time extending this length
}
```

Keeping tail values as **small** as possible at every length is a greedy
choice that never hurts future extensions and sometimes helps — a smaller
tail value means more future numbers could potentially extend that
subsequence. The binary search finds exactly where the new number fits
into this "smallest tail per length" structure, and the final length of
`tails` always equals the true LIS length, even though the actual
elements in `tails` may never have coexisted in one real subsequence.

**Why binary search works at all:** `tails` is maintained as **strictly
increasing** at all times (a property preserved by every replace/append
operation), which is exactly the precondition binary search requires.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| O(n²) DP (Approach 1) | Simpler to derive and explain, fine for this problem's constraint (n ≤ 2500) |
| O(n log n) binary search (Approach 2) | Meeting the explicit follow-up requirement, scales to much larger inputs |

---

## Complexity

| | O(n²) DP | O(n log n) |
|---|---|---|
| **Time** | O(n²) | O(n log n) |
| **Space** | O(n) | O(n) — the tails array |

---

## Key Takeaway

> The O(n²) DP answer is the natural first solution: try every earlier
> element as a potential predecessor, extending whichever gives the
> longest result. The O(n log n) follow-up answer requires a genuine
> leap in thinking — maintaining the smallest possible tail value at
> every subsequence length, and using binary search (since this
> structure stays sorted) to find where each new number fits in O(log n)
> instead of O(n). This problem is a strong example of how the "best"
> DP solution isn't always the most obvious one, and sometimes requires
> combining DP with an entirely different technique — here, binary
> search — to reach the asymptotically optimal complexity.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/longest-increasing-subsequence/)

---

{% include kotlin-dsa-nav.html %}
