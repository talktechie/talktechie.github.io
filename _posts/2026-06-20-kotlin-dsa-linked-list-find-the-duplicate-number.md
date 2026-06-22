---
title: "Linked List: Find The Duplicate Number — Kotlin Solution"
date: 2026-06-20
categories: [Kotlin DSA, Linked List]
tags: [find-the-duplicate-number, linked-list, fast-slow-pointers, binary-search, array, medium, leetcode, leetcode-287]
leetcode_number: 287
leetcode_url: https://leetcode.com/problems/find-the-duplicate-number/
difficulty: Medium
permalink: /posts/kotlin-dsa-linked-list-find-the-duplicate-number/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [287 — Find The Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/) |
| **Difficulty** | Medium |
| **Topic** | Linked List, Fast & Slow Pointers, Binary Search, Array |

> Given an array `nums` of `n + 1` integers, where each value is in the
> range `[1, n]` inclusive, there is exactly one repeated number — possibly
> more than once.
>
> Return the duplicate, **without modifying the array**, using only
> **O(1) extra space**.

**Example:**
```
Input:  nums = [1,3,4,2,2]
Output: 2

Input:  nums = [3,1,3,4,2]
Output: 3
```

**Constraints:**
- `1 <= n <= 10⁵`
- `nums.length == n + 1`
- `1 <= nums[i] <= n`
- All values appear at least once except one value that appears 2+ times
- **Follow-up:** O(1) space, can't modify the array, better than O(n²)

---

## Approach

This problem belongs in the Linked List phase because the optimal solution
borrows **directly** from Linked List Cycle (LC 141) — Floyd's
tortoise-and-hare algorithm. The trick is reframing the array as an
**implicit linked list**.

**Key insight — treat values as "next pointers":** Define `next(i) =
nums[i]`. Since values are bounded in `[1, n]` and there are `n+1` slots,
following `next` repeatedly from index 0 *must* eventually revisit a node —
this is mathematically guaranteed to form a cycle, and the **duplicate
value is exactly the entry point of that cycle**.

This is the same shape of problem as "Linked List Cycle II — find where
the cycle begins," just expressed over an array instead of node objects.

**Walk through `nums = [1,3,4,2,2]`:**
```
Think of indices as nodes, and nums[i] as a pointer to the next node:
  0 → nums[0]=1 → nums[1]=3 → nums[3]=2 → nums[2]=4 → nums[4]=2 → ... (cycle: 2→4→2→4...)

Phase 1 — detect the cycle (slow=1 step, fast=2 steps):
  slow=nums[0]=1        fast=nums[nums[0]]=nums[1]=3
  slow=nums[1]=3        fast=nums[nums[3]]=nums[2]=4
  slow=nums[3]=2        fast=nums[nums[4]]=nums[2]=4
  slow=nums[2]=4        fast=nums[nums[4]]=nums[2]=4   → slow==fast! cycle detected at value 4

Phase 2 — find cycle entry (the duplicate):
  reset slow to index 0, keep fast where it is (value 4)
  move both one step at a time:
    slow=nums[0]=1   fast=nums[4]=2
    slow=nums[1]=3   fast=nums[2]=4
    slow=nums[3]=2   fast=nums[4]=2   → slow==fast at value 2!

Answer: 2 ✓
```

---

## Kotlin Solution

### Approach 1 — Floyd's cycle detection on implicit linked list (optimal)

```kotlin
fun findDuplicate(nums: IntArray): Int {
    var slow = nums[0]
    var fast = nums[0]

    // Phase 1 — detect that a cycle exists
    do {
        slow = nums[slow]
        fast = nums[nums[fast]]
    } while (slow != fast)

    // Phase 2 — find the entrance to the cycle (the duplicate value)
    slow = nums[0]
    while (slow != fast) {
        slow = nums[slow]
        fast = nums[fast]
    }

    return slow
}
```

### Approach 2 — Binary search on value range (alternative O(n log n))

```kotlin
fun findDuplicate(nums: IntArray): Int {
    var low = 1
    var high = nums.size - 1

    while (low < high) {
        val mid = low + (high - low) / 2

        // Count how many values in nums are <= mid
        val count = nums.count { it <= mid }

        if (count > mid) {
            high = mid   // duplicate is in [low, mid]
        } else {
            low = mid + 1 // duplicate is in [mid+1, high]
        }
    }

    return low
}
```

If more than `mid` numbers are ≤ `mid`, the pigeonhole principle guarantees
a duplicate exists in that range. Doesn't modify the array and uses O(1)
space, but each binary search step costs O(n), giving O(n log n) overall —
slower than Floyd's, but a useful alternative if pointer-chasing logic
feels unintuitive.

---

## Why This Is "Linked List Cycle" in Disguise

The mapping is the entire insight:

```
Linked List Cycle (LC 141):  node.next  → next node
Find The Duplicate (LC 287): nums[i]    → "next" index
```

Because every value in `nums` is in `[1, n]` and there are `n+1` indices,
by the pigeonhole principle, **some value must repeat** — and that
repetition is exactly what creates a cycle when you follow `nums[i]` as a
pointer. The node where the cycle begins is reached by two different
paths (since the duplicate value points to the same index from two
different array entries) — which is exactly the duplicate number itself.

**Why Phase 2 finds the cycle entrance correctly** is a property of
Floyd's algorithm proven via the cycle's mathematical structure (distance
from start to cycle entrance equals distance from meeting point to
entrance, when traversed at equal speed) — the same proof used in Linked
List Cycle II.

```kotlin
slow = nums[0]            // reset to the start
while (slow != fast) {    // both now move ONE step at a time
    slow = nums[slow]
    fast = nums[fast]
}
return slow                // they meet exactly at the cycle's entrance
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Floyd's cycle detection (Approach 1) | Always — O(n) time, O(1) space, meets every constraint |
| Binary search on value range (Approach 2) | Alternative if asked for a different technique, or pointer-chasing feels unclear |

---

## Complexity

| | Floyd's | Binary Search |
|---|---|---|
| **Time** | O(n) | O(n log n) |
| **Space** | O(1) | O(1) |

---

## Key Takeaway

> When an array's values are bounded indices into itself, you can treat it
> as an implicit linked list and reuse cycle-detection techniques directly.
> This is one of the most elegant cross-pollinations in DSA — a problem
> that looks like pure array manipulation collapses into "Linked List
> Cycle II" once you see `nums[i]` as a `next` pointer. Recognizing this
> kind of structural disguise is a high-leverage skill for interviews.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/find-the-duplicate-number/)

---

{% include kotlin-dsa-nav.html %}
