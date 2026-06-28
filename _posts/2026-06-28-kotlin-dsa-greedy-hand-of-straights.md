---
title: "Greedy: Hand of Straights — Kotlin Solution"
date: 2026-06-28
categories: [Kotlin DSA, Greedy]
tags: [hand-of-straights, greedy, array, hashmap, sorting, medium, leetcode, leetcode-846]
leetcode_number: 846
leetcode_url: https://leetcode.com/problems/hand-of-straights/
difficulty: Medium
permalink: /posts/kotlin-dsa-greedy-hand-of-straights/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [846 — Hand of Straights](https://leetcode.com/problems/hand-of-straights/) |
| **Difficulty** | Medium |
| **Topic** | Greedy, Array, HashMap, Sorting |

> Given a `hand` of cards and a group size `groupSize`, return `true` if
> the hand can be rearranged into groups of `groupSize` **consecutive**
> cards.

**Example:**
```
Input:  hand = [1,2,3,6,2,3,4,7,8], groupSize = 3
Output: true
Explanation: groups [1,2,3], [2,3,4], [6,7,8]

Input:  hand = [1,2,3,4,5], groupSize = 4
Output: false
Explanation: 5 cards can't divide evenly into groups of 4
```

**Constraints:**
- `1 <= hand.length <= 10⁴`
- `0 <= hand[i] <= 10⁹`
- `1 <= groupSize <= hand.length`

---

## Approach

**Key insight — always start a new group with the smallest remaining
card:** if the smallest available card value is `x`, it **must** be the
start of some group of `groupSize` consecutive cards — there's no
possible group it could be the *middle* or *end* of, since nothing
smaller remains. Greedily form the group `[x, x+1, ..., x+groupSize-1]`,
consuming one count of each from a frequency map. If any required card
is missing, the hand can't be arranged — return `false` immediately.

**Walk through `hand = [1,2,3,6,2,3,4,7,8], groupSize = 3`:**
```
count = {1:1, 2:2, 3:2, 4:1, 6:1, 7:1, 8:1}

smallest remaining = 1 → form group [1,2,3]:
  consume one each of 1,2,3 → count = {2:1, 3:1, 4:1, 6:1, 7:1, 8:1}

smallest remaining = 2 → form group [2,3,4]:
  consume one each of 2,3,4 → count = {6:1, 7:1, 8:1}

smallest remaining = 6 → form group [6,7,8]:
  consume one each of 6,7,8 → count = {} (empty)

All cards consumed successfully → true ✓
```

---

## Kotlin Solution

### Approach 1 — Frequency map + sorted keys, greedily consume from the smallest (optimal)

```kotlin
fun isNStraightHand(hand: IntArray, groupSize: Int): Boolean {
    if (hand.size % groupSize != 0) return false   // can't divide evenly

    val count = sortedMapOf<Int, Int>()
    for (card in hand) {
        count[card] = (count[card] ?: 0) + 1
    }

    while (count.isNotEmpty()) {
        val smallest = count.firstKey()

        for (card in smallest until smallest + groupSize) {
            val remaining = count[card] ?: return false   // required card missing entirely

            if (remaining == 1) {
                count.remove(card)
            } else {
                count[card] = remaining - 1
            }
        }
    }

    return true
}
```

### Approach 2 — Min-heap instead of a sorted map (equivalent, different data structure)

```kotlin
fun isNStraightHand(hand: IntArray, groupSize: Int): Boolean {
    if (hand.size % groupSize != 0) return false

    val count = HashMap<Int, Int>()
    for (card in hand) count[card] = (count[card] ?: 0) + 1

    val minHeap = PriorityQueue(count.keys)

    while (minHeap.isNotEmpty()) {
        val smallest = minHeap.peek()

        for (card in smallest until smallest + groupSize) {
            val remaining = count[card] ?: return false

            count[card] = remaining - 1
            if (count[card] == 0) {
                count.remove(card)
                if (card == minHeap.peek()) minHeap.poll()
            }
        }
    }

    return true
}
```

Uses a min-heap of distinct card values instead of a sorted map — same
underlying logic (always process the smallest available value first),
expressed with a different ordered data structure.

---

## Why the Smallest Remaining Card Must Always Start a New Group

This is the key greedy proof, and it's worth being explicit about why no
other choice could ever be better:

```kotlin
val smallest = count.firstKey()
for (card in smallest until smallest + groupSize) { ... }
```

Since `smallest` is the minimum value currently available, **no** group
can place it anywhere except as the **first** card of that group — there
is no smaller card left to precede it. If a valid arrangement of the
hand exists at all, the smallest card *must* belong to a group starting
exactly at its own value. There's no ambiguity or choice to greedily
optimize over here — the smallest card's role is fully determined,
making this less a "greedy heuristic" and more a logical necessity.

**Why checking `hand.size % groupSize != 0` upfront is a cheap, valid
pre-filter:** if the total card count doesn't divide evenly into
`groupSize`-sized groups, no arrangement can possibly work, regardless of
the specific values — this O(1) check avoids unnecessary computation on
clearly impossible inputs.

**Why returning `false` immediately when a required card is missing
(`count[card] ?: return false`)** is both correct and efficient: once
we've committed to starting a group at the current smallest value, every
card in that group's range is *non-negotiably* required — there's no
alternative arrangement to fall back on if even one is missing.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Sorted map (Approach 1) | Clean, leverages `TreeMap`-style ordering directly via `firstKey()` |
| Min-heap (Approach 2) | Prefer heap-based "always get the smallest" access pattern |

---

## Complexity

| | |
|---|---|
| **Time** | O(n log n) — n = hand.length; dominated by maintaining sorted/heap order of distinct values |
| **Space** | O(n) — the frequency map |

---

## Key Takeaway

> Whenever the smallest (or largest) remaining element in a greedy
> problem has a *uniquely determined* role — not just a "probably good"
> choice, but the *only* possible choice — that's a strong signal the
> greedy approach is not just convenient but provably correct. Here, the
> smallest card can only ever start a group, since nothing smaller
> remains to precede it. This kind of "no other choice is even possible"
> reasoning is often easier to verify than the more delicate
> exchange-argument proofs greedy algorithms sometimes require.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/hand-of-straights/)

---

{% include kotlin-dsa-nav.html %}
