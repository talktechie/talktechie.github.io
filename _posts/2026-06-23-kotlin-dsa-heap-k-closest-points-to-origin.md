---
title: "Heap: K Closest Points to Origin — Kotlin Solution"
date: 2026-06-23
categories: [Kotlin DSA, Heap]
tags: [k-closest-points-to-origin, heap, priority-queue, array, sorting, medium, leetcode, leetcode-973]
leetcode_number: 973
leetcode_url: https://leetcode.com/problems/k-closest-points-to-origin/
difficulty: Medium
permalink: /posts/kotlin-dsa-heap-k-closest-points-to-origin/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [973 — K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/) |
| **Difficulty** | Medium |
| **Topic** | Heap, Priority Queue, Array, Sorting |

> Given an array of `points` where `points[i] = [xi, yi]`, return the `k`
> points **closest to the origin** `(0, 0)`. The answer may be returned
> in any order — distance is measured as standard Euclidean distance.

**Example:**
```
Input:  points = [[1,3],[-2,2]], k = 1
Output: [[-2,2]]
Explanation: distance of (1,3) = sqrt(10) ≈ 3.16
             distance of (-2,2) = sqrt(8) ≈ 2.83 — closer

Input:  points = [[3,3],[5,-1],[-2,4]], k = 2
Output: [[3,3],[-2,4]]
```

**Constraints:**
- `1 <= k <= points.length <= 10⁴`
- `-10⁴ <= xi, yi <= 10⁴`

---

## Approach

This directly reuses the "min-heap capped at size k" pattern from Kth
Largest Element In a Stream — except here we want the `k` points with
the **smallest** distance, so it's naturally a **max-heap** capped at
size `k` (the opposite heap direction from that problem, since here we
want to evict the *worst* of the top `k`, which is the *largest*
distance among them).

**Key insight:** We never need the actual square-root distance — since
all distances are non-negative, comparing squared distances
(`x² + y²`) preserves the same ordering and avoids unnecessary floating
point computation. Maintain a max-heap of size `k` based on squared
distance; the heap always holds the `k` closest points seen so far.

**Walk through `points = [[3,3],[5,-1],[-2,4]], k = 2`:**
```
dist(3,3)  = 9+9=18
dist(5,-1) = 25+1=26
dist(-2,4) = 4+16=20

max-heap (by distance), capped at size 2:
  push (3,3)→18   heap: {18}
  push (5,-1)→26  heap: {18,26}
  push (-2,4)→20  heap: {18,20,26} → size=3 > k=2 → pop largest (26, point (5,-1))
                  heap: {18,20}

Remaining in heap: (3,3) dist=18, (-2,4) dist=20

Result: [[3,3],[-2,4]] ✓
```

---

## Kotlin Solution

### Approach 1 — Max-heap by distance, capped at size k (optimal for streaming/large n)

```kotlin
fun kClosest(points: Array<IntArray>, k: Int): Array<IntArray> {
    val maxHeap = PriorityQueue<IntArray>(
        compareByDescending { p -> p[0] * p[0] + p[1] * p[1] }
    )

    for (point in points) {
        maxHeap.add(point)
        if (maxHeap.size > k) maxHeap.poll()
    }

    return maxHeap.toTypedArray()
}
```

### Approach 2 — Sort all points by distance, take the first k

```kotlin
fun kClosest(points: Array<IntArray>, k: Int): Array<IntArray> {
    return points
        .sortedBy { p -> p[0] * p[0] + p[1] * p[1] }
        .take(k)
        .toTypedArray()
}
```

Simpler and very readable — sorts everything by squared distance, then
slices the first `k`. O(n log n), slightly worse than the heap's O(n log k)
when `k` is much smaller than `n`, but for most practical inputs the
difference is negligible.

---

## Why Squared Distance Avoids Unnecessary `sqrt()` Calls

```kotlin
compareByDescending { p -> p[0] * p[0] + p[1] * p[1] }
// NOT: sqrt(p[0]*p[0] + p[1]*p[1])
```

Since `sqrt` is a strictly increasing function for non-negative inputs,
comparing `a² + b²` directly gives the **exact same ordering** as
comparing `sqrt(a² + b²)` — but skips a relatively expensive floating
point operation on every single comparison. This is a small but free
optimization worth applying any time you only care about *relative*
distance ordering, not the actual distance value.

**Why max-heap (not min-heap) here, unlike Kth Largest Element In a
Stream's min-heap:** we want to keep the `k` points with the **smallest**
distances. The heap should evict whichever point is currently the
**worst** among the kept ones — for "smallest distances," the worst is
the **largest** distance, so we need a max-heap to efficiently find and
evict it:
```kotlin
if (maxHeap.size > k) maxHeap.poll()
// poll() removes the LARGEST distance — the one we'd least want to keep
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Max-heap capped at k (Approach 1) | Large `n`, especially if points arrive as a stream rather than all at once |
| Sort and take k (Approach 2) | Simpler code, fine when the full point list is available upfront |

---

## Complexity

| | Max-Heap | Sort |
|---|---|---|
| **Time** | O(n log k) | O(n log n) |
| **Space** | O(k) | O(n) |

---

## Key Takeaway

> "Find the k closest/smallest" is the mirror image of "find the k
> largest" — same heap-capped-at-k pattern, but flip which heap direction
> evicts the worst candidate. Comparing squared distances instead of
> actual distances is a useful general trick whenever only relative
> ordering matters. Whether to reach for a heap or a plain sort comes
> down to whether `k` is meaningfully smaller than `n` and whether data
> arrives incrementally.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/k-closest-points-to-origin/)

---

{% include kotlin-dsa-nav.html %}
