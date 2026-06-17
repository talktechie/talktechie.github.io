---
title: "Two Pointers: 3Sum — Kotlin Solution"
date: 2026-06-16
categories: [Kotlin DSA, Two Pointers]
tags: [3sum, two-pointers, sorting, medium, leetcode, leetcode-15]
leetcode_number: 15
leetcode_url: https://leetcode.com/problems/3sum/
difficulty: Medium
permalink: /posts/kotlin-dsa-two-pointers-3sum/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [15 — 3Sum](https://leetcode.com/problems/3sum/) |
| **Difficulty** | Medium |
| **Topic** | Two Pointers, Sorting, Array |

> Given an integer array `nums`, return all the triplets
> `[nums[i], nums[j], nums[k]]` such that `i != j`, `i != k`, `j != k`,
> and `nums[i] + nums[j] + nums[k] == 0`.
>
> The solution set must **not contain duplicate triplets**.

**Example:**
```
Input:  nums = [-1,0,1,2,-1,-4]
Output: [[-1,-1,2],[-1,0,1]]

Input:  nums = [0,1,1]
Output: []

Input:  nums = [0,0,0]
Output: [[0,0,0]]
```

**Constraints:**
- `3 <= nums.length <= 3000`
- `-10⁵ <= nums[i] <= 10⁵`

---

## Approach

Fix one number, reduce to Two Sum II on the rest.

**Key insight:** Sort the array first. Then iterate — for each element at index `i`,
run a two-pointer search on `nums[i+1..n-1]` for a pair that sums to `-nums[i]`.
Sorting lets us skip duplicates cleanly: if `nums[i] == nums[i-1]`, we've already
processed all triplets that start with this value — skip it.

**Walk through `[-1, 0, 1, 2, -1, -4]` → sorted: `[-4, -1, -1, 0, 1, 2]`:**
```
i=0, nums[i]=-4 → need pair summing to 4
  left=1(-1) right=5(2) → -1+2=1 < 4 → left++
  left=2(-1) right=5(2) → -1+2=1 < 4 → left++
  left=3(0)  right=5(2) → 0+2=2  < 4 → left++
  left=4(1)  right=5(2) → 1+2=3  < 4 → left++ → pointers cross, no pair

i=1, nums[i]=-1 → need pair summing to 1
  left=2(-1) right=5(2) → -1+2=1 == 1 → add [-1,-1,2] ✓
    advance both, skip duplicates
  left=3(0)  right=4(1) → 0+1=1  == 1 → add [-1,0,1] ✓

i=2, nums[i]=-1 → same as i=1 (duplicate) → skip

i=3, nums[i]=0  → need pair summing to 0
  left=4(1)  right=5(2) → 1+2=3  > 0 → right--
  → pointers cross, no pair

Result: [[-1,-1,2],[-1,0,1]] ✓
```

---

## Kotlin Solution

### Approach 1 — Sort + two pointers (optimal)

```kotlin
fun threeSum(nums: IntArray): List<List<Int>> {
    nums.sort()
    val result = mutableListOf<List<Int>>()

    for (i in 0..nums.size - 3) {
        // Skip duplicates for the fixed element
        if (i > 0 && nums[i] == nums[i - 1]) continue
        // Early exit: smallest possible triplet sum > 0
        if (nums[i] > 0) break

        var left  = i + 1
        var right = nums.lastIndex

        while (left < right) {
            val sum = nums[i] + nums[left] + nums[right]
            when {
                sum == 0 -> {
                    result.add(listOf(nums[i], nums[left], nums[right]))
                    // Skip duplicates for left and right
                    while (left < right && nums[left]  == nums[left + 1])  left++
                    while (left < right && nums[right] == nums[right - 1]) right--
                    left++
                    right--
                }
                sum < 0  -> left++
                else     -> right--
            }
        }
    }

    return result
}
```

### Approach 2 — HashSet deduplication (less idiomatic, for comparison)

```kotlin
fun threeSum(nums: IntArray): List<List<Int>> {
    nums.sort()
    val resultSet = HashSet<List<Int>>()

    for (i in nums.indices) {
        val seen = HashSet<Int>()
        for (j in i + 1..nums.lastIndex) {
            val complement = -nums[i] - nums[j]
            if (complement in seen) {
                resultSet.add(listOf(nums[i], complement, nums[j]))
            }
            seen.add(nums[j])
        }
    }

    return resultSet.toList()
}
```

Works but the HashSet overhead makes it slower in practice. The pointer skip
approach in Approach 1 handles duplicates without extra space.

---

## The Two Duplicate-Skip Rules

Getting the duplicate skipping right is the hardest part of this problem.

**Rule 1 — skip duplicate fixed element:**
```kotlin
if (i > 0 && nums[i] == nums[i - 1]) continue
// [-1, -1, 0, 1] → processing i=0 (-1) covers all triplets starting with -1
// When i=1 we'd produce the same triplets → skip
```

**Rule 2 — skip duplicate left/right after a match:**
```kotlin
while (left < right && nums[left]  == nums[left + 1])  left++
while (left < right && nums[right] == nums[right - 1]) right--
left++   // still advance past the last duplicate
right--
// [-2, 0, 0, 0, 2] with i fixed → only add [−2,0,2] once
```

**Early exit optimisation:**
```kotlin
if (nums[i] > 0) break
// After sorting, if the smallest remaining element > 0,
// no three elements can sum to 0 — all further sums are positive
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Sort + two pointers (Approach 1) | Always — O(n²) optimal, O(1) extra space |
| HashSet deduplication (Approach 2) | When you need a quick first pass; clean up after |

---

## Complexity

| | |
|---|---|
| **Time** | O(n²) — O(n log n) sort + O(n) per element for two-pointer scan |
| **Space** | O(1) extra — output list not counted |

---

## Key Takeaway

> Sort first. Fix one element, run Two Sum II on the rest.
> Two duplicate rules: skip `nums[i] == nums[i-1]` at the outer loop,
> and skip equal values after finding a match in the inner loop.
> O(n²) is optimal for this problem — no known general solution does better.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/3sum/)

---

{% include kotlin-dsa-nav.html %}
