---
title: "Binary Search: Koko Eating Bananas — Kotlin Solution"
date: 2026-06-18
categories: [Kotlin DSA, Binary Search]
tags: [koko-eating-bananas, binary-search, binary-search-on-answer, array, medium, leetcode, leetcode-875]
leetcode_number: 875
leetcode_url: https://leetcode.com/problems/koko-eating-bananas/
difficulty: Medium
permalink: /posts/kotlin-dsa-binary-search-koko-eating-bananas/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [875 — Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/) |
| **Difficulty** | Medium |
| **Topic** | Binary Search, Binary Search on Answer, Array |

> Koko loves bananas. There are `n` piles, the `i`-th pile has `piles[i]`
> bananas. Guards return in `h` hours.
>
> Koko picks an eating speed `k` (bananas per hour). Each hour she chooses
> one pile and eats `k` bananas from it — if the pile has fewer than `k`,
> she eats the rest and waits until the next hour for a new pile.
>
> Return the **minimum** integer `k` such that she can eat all bananas
> within `h` hours.

**Example:**
```
Input:  piles = [3,6,7,11], h = 8
Output: 4

Input:  piles = [30,11,23,4,20], h = 5
Output: 30

Input:  piles = [30,11,23,4,20], h = 6
Output: 23
```

**Constraints:**
- `1 <= piles.length <= 10⁴`
- `piles.length <= h <= 10⁹`
- `1 <= piles[i] <= 10⁹`

---

## Approach

This is **binary search on the answer**, a different flavor than searching
for a value in an array. Instead of searching a data structure, we search
the **space of possible answers** (eating speeds) for the smallest one that
satisfies a condition.

**Key insight:** As `k` increases, the total hours needed to finish strictly
decreases (or stays the same) — it's a **monotonic relationship**. That
monotonicity is exactly what makes binary search valid here:

```
k=1 → hours = sum of all piles           (slowest, most hours)
k=max(piles) → hours = number of piles   (fastest, fewest hours)
```

We binary search `k` between `1` and `max(piles)`. For each candidate `k`,
compute the hours needed (`ceil(pile / k)` summed across all piles). If
hours ≤ h, this `k` works — try smaller. If hours > h, `k` is too slow —
try bigger.

**Walk through `piles = [3,6,7,11], h = 8`:**
```
left=1, right=11

mid=6 → hours = ceil(3/6)+ceil(6/6)+ceil(7/6)+ceil(11/6) = 1+1+2+2 = 6 ≤ 8
       → k=6 works, try smaller → right=5

mid=3 → hours = ceil(3/3)+ceil(6/3)+ceil(7/3)+ceil(11/3) = 1+2+3+4 = 10 > 8
       → too slow → left=4

mid=4 → hours = ceil(3/4)+ceil(6/4)+ceil(7/4)+ceil(11/4) = 1+2+2+3 = 8 ≤ 8
       → k=4 works, try smaller → right=3

left=4, right=3 → left > right, stop

Answer: 4 ✓
```

---

## Kotlin Solution

### Approach 1 — Binary search on answer space (optimal)

```kotlin
fun minEatingSpeed(piles: IntArray, h: Int): Int {
    var left = 1
    var right = piles.max()
    var answer = right

    while (left <= right) {
        val k = left + (right - left) / 2
        val hours = piles.sumOf { pile -> (pile + k - 1) / k }  // ceil division

        if (hours <= h) {
            answer = k          // k works — record it, try to do better
            right = k - 1
        } else {
            left = k + 1         // too slow — need a bigger k
        }
    }

    return answer
}
```

### Approach 2 — Same logic, explicit ceiling division helper

```kotlin
fun minEatingSpeed(piles: IntArray, h: Int): Int {
    fun hoursNeeded(k: Int): Long =
        piles.sumOf { pile -> kotlin.math.ceil(pile.toDouble() / k).toLong() }

    var left = 1
    var right = piles.max()

    while (left < right) {
        val mid = left + (right - left) / 2
        if (hoursNeeded(mid) <= h) right = mid else left = mid + 1
    }

    return left
}
```

Uses floating-point `ceil` for clarity instead of the integer trick — slightly
slower due to Double conversion, but easier to read for newcomers to the pattern.

---

## Why `(pile + k - 1) / k` Computes Ceiling Division

Integer division in Kotlin truncates toward zero. To round **up** without
floating point:

```kotlin
(pile + k - 1) / k
// pile=7, k=4 → (7+4-1)/4 = 10/4 = 2  (matches ceil(7/4) = 2) ✓
// pile=6, k=3 → (6+3-1)/3 = 8/3  = 2  (matches ceil(6/3) = 2, exact division) ✓
```

Adding `k - 1` before dividing pushes any remainder over the next integer
boundary, but exact multiples aren't pushed past their own boundary —
exactly what ceiling division should do.

**`piles.sumOf { ... }`** — Kotlin's clean way to sum a transformed sequence:
```kotlin
val hours = piles.sumOf { pile -> (pile + k - 1) / k }
// vs manual loop:
//   var hours = 0
//   for (pile in piles) hours += (pile + k - 1) / k
```

**Why `right = k - 1` (not `right = k`) when `k` works:** we've already
recorded `k` as a valid `answer` — now we look strictly left of it for
anything smaller that also works. Without the `-1`, the loop could spin
forever re-checking the same `k`.

---

## Recognizing "Binary Search on Answer" Problems

This pattern shows up whenever you see:
- "Find the **minimum/maximum** value such that [condition] holds"
- The condition is **monotonic** in the answer (as the answer increases,
  the condition flips from false→true or true→false exactly once)
- The "check if this answer works" function is easy to compute (here, O(n)
  to sum hours for a given `k`)

Once you spot monotonicity, binary search the answer space directly instead
of searching the input array.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Integer ceiling trick (Approach 1) | Always — avoids floating point, faster |
| Float `ceil` (Approach 2) | Teaching/learning — more explicit about intent |

---

## Complexity

| | |
|---|---|
| **Time** | O(n × log(max(piles))) — binary search over the speed range, O(n) work per check |
| **Space** | O(1) |

---

## Key Takeaway

> When asked for the minimum/maximum value satisfying a monotonic condition,
> binary search the **answer space**, not the input. Here, speed `k` versus
> hours-needed is monotonic — higher `k` never needs more hours. Define a
> "does this k work?" check, then binary search over candidate `k` values.
> This exact template reappears in Capacity to Ship Packages, Split Array
> Largest Sum, and many other "minimize the maximum" problems.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/koko-eating-bananas/)

---

{% include kotlin-dsa-nav.html %}
