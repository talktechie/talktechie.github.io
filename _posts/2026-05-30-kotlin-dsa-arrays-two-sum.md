---
title: "Arrays: Two Sum — Kotlin Solution"
date: 2026-05-30
categories: [Kotlin DSA, Arrays]
tags: [two-sum, hashmap, easy, leetcode, leetcode-1, arrays]
leetcode_number: 1
leetcode_url: https://leetcode.com/problems/two-sum/
difficulty: Easy
permalink: /posts/kotlin-dsa-arrays-two-sum/
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [1 — Two Sum](https://leetcode.com/problems/two-sum/) |
| **Difficulty** | Easy |
| **Topic** | Arrays, HashMap |

> Given an array of integers `nums` and an integer `target`, return the indices of the two numbers that add up to `target`.

**Example:**
```
Input:  nums = [2, 7, 11, 15], target = 9
Output: [0, 1]  // nums[0] + nums[1] = 2 + 7 = 9
```

---

## Approach

The brute force way is O(n²) — check every pair. We can do better.

**Key insight:** For each number `x`, we need `target - x`. Instead of searching for it,
store every number we've seen in a HashMap. Then for each new number, just check if
its complement already exists.

One pass. O(n) time.

---

## Kotlin Solution

```kotlin
fun twoSum(nums: IntArray, target: Int): IntArray {
    val seen = HashMap<Int, Int>() // value -> index

    for ((index, num) in nums.withIndex()) {
        val complement = target - num

        if (complement in seen) {
            return intArrayOf(seen[complement]!!, index)
        }

        seen[num] = index
    }

    return intArrayOf() // no solution found
}
```

---

## Why Kotlin Shines Here

**`withIndex()`** — clean destructuring of index and value, no manual `i` counter:
```kotlin
for ((index, num) in nums.withIndex())
// vs Java: for (int i = 0; i < nums.length; i++)
```

**`in` operator** — readable HashMap lookup:
```kotlin
if (complement in seen)
// vs Java: if (seen.containsKey(complement))
```

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — single pass |
| **Space** | O(n) — HashMap stores up to n elements |

---

## Key Takeaway

> Store what you've seen. Check if the complement exists. One pass beats two loops every time.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/two-sum/)

---

{% include kotlin-dsa-nav.html %}
