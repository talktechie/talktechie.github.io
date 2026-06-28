---
title: "Greedy: Merge Triplets to Form Target Triplet — Kotlin Solution"
date: 2026-06-28
categories: [Kotlin DSA, Greedy]
tags: [merge-triplets-to-form-target-triplet, greedy, array, medium, leetcode, leetcode-2382]
leetcode_number: 2382
leetcode_url: https://leetcode.com/problems/merge-triplets-to-form-target-triplet/
difficulty: Medium
permalink: /posts/kotlin-dsa-greedy-merge-triplets-to-form-target-triplet/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [2382 — Merge Triplets to Form Target Triplet](https://leetcode.com/problems/merge-triplets-to-form-target-triplet/) |
| **Difficulty** | Medium |
| **Topic** | Greedy, Array |

> Given a 2D array `triplets` and a `target` triplet, you may **merge**
> any two triplets by taking the **elementwise maximum** of each
> position. Return `true` if you can produce `target` exactly through
> some sequence of merges.

**Example:**
```
Input:  triplets = [[2,5,3],[1,8,4],[1,7,5]], target = [2,7,5]
Output: true
Explanation: merge [1,8,4] and [1,7,5] → [max(1,1),max(8,7),max(4,5)]
             = [1,8,5]; then merge with [2,5,3] →
             [max(2,1),max(5,8),max(3,5)] = [2,8,5] ... 

Let's verify against the actual valid path: merge [2,5,3] and [1,7,5]
  → [max(2,1),max(5,7),max(3,5)] = [2,7,5] = target directly! ✓

Input:  triplets = [[3,4,5],[4,4,4]], target = [3,2,5]
Output: false
Explanation: every triplet has a value (4 in position 1) exceeding the
             target's corresponding value (2) — merging can only
             increase values via max, never reduce them, so target's
             position-1 value of 2 is unreachable
```

**Constraints:**
- `1 <= triplets.length <= 10⁵`
- `triplets[i].length == target.length == 3`
- `1 <= triplets[i][j], target[j] <= 1000`

---

## Approach

**Key insight 1 — discard any triplet with an "overshooting" value
immediately:** since merging only ever takes the **maximum**, a value
can never decrease through merging. If any triplet has even one position
where its value **exceeds** the target's corresponding value, that
triplet can **never** participate in a valid sequence of merges leading
to `target` — including it would force that position to overshoot.
Filter it out entirely.

**Key insight 2 — among the remaining "safe" triplets, check if each
target position can be achieved by SOME triplet:** since invalid
(overshooting) triplets are already excluded, every remaining triplet
has every value `<= target[j]` at every position `j`. The question
becomes: across all these safe triplets, does **every position** of
`target` get matched **exactly** by at least one of them? If so, merging
all the safe triplets together (taking the max at every position across
all of them) produces exactly `target`.

**Walk through `triplets = [[2,5,3],[1,8,4],[1,7,5]], target = [2,7,5]`:**
```
Check [2,5,3] against target [2,7,5]: 2<=2 ✓, 5<=7 ✓, 3<=5 ✓ → SAFE
  matches: position0(2==2) YES, position1(5==7) no, position2(3==5) no

Check [1,8,4] against target: 1<=2 ✓, 8<=7? NO (8>7) → DISCARD (overshoots)

Check [1,7,5] against target: 1<=2 ✓, 7<=7 ✓, 5<=5 ✓ → SAFE
  matches: position0(1==2) no, position1(7==7) YES, position2(5==5) YES

Across all SAFE triplets, is every position matched by at least one?
  position0: matched by [2,5,3] ✓
  position1: matched by [1,7,5] ✓
  position2: matched by [1,7,5] ✓

All 3 positions matched → true ✓
```

---

## Kotlin Solution

### Approach 1 — Filter overshooting triplets, track which positions get matched (optimal)

```kotlin
fun mergeTriplets(triplets: Array<IntArray>, target: IntArray): Boolean {
    val matched = BooleanArray(3)

    for (triplet in triplets) {
        // Skip any triplet that overshoots target at any position
        if (triplet[0] > target[0] || triplet[1] > target[1] || triplet[2] > target[2]) {
            continue
        }

        // This triplet is "safe" — record which positions it exactly matches
        for (j in 0..2) {
            if (triplet[j] == target[j]) matched[j] = true
        }
    }

    return matched.all { it }
}
```

### Approach 2 — Equivalent logic using a Set of matched indices

```kotlin
fun mergeTriplets(triplets: Array<IntArray>, target: IntArray): Boolean {
    val matchedPositions = mutableSetOf<Int>()

    for (triplet in triplets) {
        if (triplet.indices.any { triplet[it] > target[it] }) continue

        for (j in triplet.indices) {
            if (triplet[j] == target[j]) matchedPositions.add(j)
        }
    }

    return matchedPositions.size == 3
}
```

---

## Why Discarding Overshooting Triplets Is Always Safe, Never Premature

This is the crucial greedy insight worth proving carefully: filtering
out a triplet the instant it overshoots `target` at *any* position never
discards a triplet we might have needed.

```kotlin
if (triplet[0] > target[0] || triplet[1] > target[1] || triplet[2] > target[2]) {
    continue
}
```

Merging combines two triplets via elementwise **maximum** — this
operation can only ever **preserve or increase** values, never decrease
them. So if a triplet has `triplet[j] > target[j]` at any position `j`,
including it in **any** sequence of merges would force the final result
to have `result[j] >= triplet[j] > target[j]` — overshooting `target`
at position `j` permanently. There's no way to "undo" this via further
merging. This means such a triplet is **unconditionally** useless for
reaching `target` — not just unlikely to help, but mathematically
incapable of helping.

**Why checking "does every position get matched by SOME safe triplet"
is sufficient (not just necessary):** once every overshooting triplet is
removed, every remaining triplet's values are all `<= target` at every
position. Merging **all** of them together (taking the elementwise max
across the entire safe set) can therefore never exceed `target` at any
position — and if every position is matched exactly by at least one safe
triplet, that maximum will reach exactly `target[j]` at every `j`,
producing precisely `target` overall.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| BooleanArray tracking (Approach 1) | Simple, fixed-size (3 positions), no extra collection overhead |
| Set-based tracking (Approach 2) | Prefer a more general/idiomatic collection-based check |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — n = triplets.length, constant work (3 positions) per triplet |
| **Space** | O(1) — fixed-size tracking array |

---

## Key Takeaway

> The merge operation's elementwise-maximum nature gives an immediate,
> airtight greedy filter: any triplet exceeding the target at any
> position can be discarded outright, since merging can never reduce a
> value. What remains is a simpler coverage question — does every target
> position get hit exactly by at least one safe triplet? This is a clean
> example of a greedy "filter first, then check coverage" strategy,
> where the filtering step isn't a heuristic shortcut but a logically
> guaranteed elimination of impossible candidates.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/merge-triplets-to-form-target-triplet/)

---

{% include kotlin-dsa-nav.html %}
