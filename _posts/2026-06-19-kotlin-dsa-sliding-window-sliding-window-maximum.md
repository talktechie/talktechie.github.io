---
title: "Sliding Window: Sliding Window Maximum — Kotlin Solution"
date: 2026-06-19
categories: [Kotlin DSA, Sliding Window]
tags: [sliding-window-maximum, sliding-window, monotonic-deque, array, hard, leetcode, leetcode-239]
leetcode_number: 239
leetcode_url: https://leetcode.com/problems/sliding-window-maximum/
difficulty: Hard
permalink: /posts/kotlin-dsa-sliding-window-sliding-window-maximum/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [239 — Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) |
| **Difficulty** | Hard |
| **Topic** | Sliding Window, Monotonic Deque, Array |

> Given an array `nums` and a sliding window of size `k` moving from the
> very left to the very right of the array, return an array of the
> maximum value in the window at each position.

**Example:**
```
Input:  nums = [1,3,-1,-3,5,3,6,7], k = 3
Output: [3,3,5,5,6,7]
Explanation:
Window position                Max
---------------                ---
[1  3  -1] -3  5  3  6  7        3
 1 [3  -1  -3] 5  3  6  7        3
 1  3 [-1  -3  5] 3  6  7        5
 1  3  -1 [-3  5  3] 6  7        5
 1  3  -1  -3 [5  3  6] 7        6
 1  3  -1  -3  5 [3  6  7]       7

Input:  nums = [1], k = 1
Output: [1]
```

**Constraints:**
- `1 <= nums.length <= 10⁵`
- `-10⁴ <= nums[i] <= 10⁴`
- `1 <= k <= nums.length`

---

## Approach

A brute-force scan of every window is O(n × k) — too slow for `n = 10⁵`.
We need O(n) total, which means each element should be processed in
roughly constant amortized work.

**Key insight — monotonic decreasing deque:** Maintain a deque of
**indices** such that the values they point to are in strictly decreasing
order from front to back. The front of the deque is always the index of
the current window's maximum.

Two cleanup rules keep the deque valid as the window slides:
1. **Remove from the back** any index whose value is ≤ the new incoming
   value — they can never be the maximum again while a bigger, more
   recent value is in the window.
2. **Remove from the front** any index that has fallen outside the
   current window (`index <= right - k`).

**Walk through `nums = [1,3,-1,-3,5,3,6,7], k = 3`:**
```
right=0 (1)  → deque empty → push 0 → deque: [0]
right=1 (3)  → 3 >= nums[0]=1 → pop 0 → deque empty → push 1 → deque: [1]
right=2 (-1) → -1 < nums[1]=3 → push 2 → deque: [1,2]
            → window size reached (k=3) → front=1, nums[1]=3 → output: [3]

right=3 (-3) → -3 < nums[2]=-1 → push 3 → deque: [1,2,3]
            → front index 1 still in window (1 > 3-3=0)? yes
            → output: [3,3]

right=4 (5)  → 5>=nums[3]=-3 → pop 3 → 5>=nums[2]=-1 → pop 2 → 5>=nums[1]=3 → pop 1
            → deque empty → push 4 → deque: [4]
            → output: [3,3,5]

right=5 (3)  → 3 < nums[4]=5 → push 5 → deque: [4,5]
            → front index 4 in window (4 > 5-3=2)? yes → output: [3,3,5,5]

right=6 (6)  → 6>=nums[5]=3 → pop 5 → 6>=nums[4]=5 → pop 4 → deque empty → push 6
            → deque: [6] → output: [3,3,5,5,6]

right=7 (7)  → 7>=nums[6]=6 → pop 6 → deque empty → push 7 → deque: [7]
            → output: [3,3,5,5,6,7]

Result: [3,3,5,5,6,7] ✓
```

---

## Kotlin Solution

### Approach 1 — Monotonic deque of indices (optimal)

```kotlin
fun maxSlidingWindow(nums: IntArray, k: Int): IntArray {
    val deque = ArrayDeque<Int>()  // stores indices, decreasing values front→back
    val result = mutableListOf<Int>()

    for (right in nums.indices) {
        // Remove indices from the back whose values are beaten by the new value
        while (deque.isNotEmpty() && nums[deque.last()] <= nums[right]) {
            deque.removeLast()
        }
        deque.addLast(right)

        // Remove the front index if it has slid outside the window
        if (deque.first() <= right - k) {
            deque.removeFirst()
        }

        // Once the window is fully formed, record the max (front of deque)
        if (right >= k - 1) {
            result.add(nums[deque.first()])
        }
    }

    return result.toIntArray()
}
```

### Approach 2 — Brute force (O(n×k), for comparison only)

```kotlin
fun maxSlidingWindow(nums: IntArray, k: Int): IntArray {
    val result = IntArray(nums.size - k + 1)

    for (i in result.indices) {
        var maxVal = nums[i]
        for (j in i until i + k) {
            maxVal = maxOf(maxVal, nums[j])
        }
        result[i] = maxVal
    }

    return result
}
```

Correct but O(n×k) — TLEs on large inputs with large `k`. Included only to
contrast against the deque solution.

---

## Why the Deque Stays Small and the Algorithm Is O(n)

Each index is pushed onto the deque exactly once. It can be popped at most
once too — either from the back (beaten by a larger value) or from the
front (aged out of the window). That bounds total deque operations at
O(n), giving an amortized O(1) per element despite the inner `while` loop.

**Why pop from the back on `<=` (not just `<`)** — if two elements are
equal, the more recent one will stay in the window longer, so the older
equal value is useless to keep around:
```kotlin
while (deque.isNotEmpty() && nums[deque.last()] <= nums[right]) {
    deque.removeLast()
}
```

**Front index validity check** — `right - k` is the index that just fell
out of the window (window is `[right - k + 1, right]`):
```kotlin
if (deque.first() <= right - k) {
    deque.removeFirst()
}
```

**Only recording results once the window is full size:**
```kotlin
if (right >= k - 1) {
    result.add(nums[deque.first()])
}
// Before right reaches index k-1, the window hasn't fully formed yet
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Monotonic deque (Approach 1) | Always — true O(n), the intended solution |
| Brute force (Approach 2) | Understanding the problem only; never submit for large inputs |

---

## Complexity

| | Monotonic Deque | Brute Force |
|---|---|---|
| **Time** | O(n) amortized | O(n × k) |
| **Space** | O(k) — deque holds at most k indices | O(1) extra |

---

## Key Takeaway

> A fixed-size window's running maximum is best tracked with a monotonic
> decreasing deque of indices, not a full re-scan per window. Pop smaller
> values from the back (they're permanently irrelevant once a bigger value
> arrives), and pop stale indices from the front (they've aged out of the
> window). The front of the deque is always the current maximum, available
> in O(1). This closes out the Sliding Window phase by combining the
> "fixed-size window" idea from Permutation In String with the monotonic
> stack/deque technique from the Stack phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/sliding-window-maximum/)

---

{% include kotlin-dsa-nav.html %}
