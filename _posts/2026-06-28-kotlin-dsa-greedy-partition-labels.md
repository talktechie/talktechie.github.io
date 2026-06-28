---
title: "Greedy: Partition Labels — Kotlin Solution"
date: 2026-06-28
categories: [Kotlin DSA, Greedy]
tags: [partition-labels, greedy, string, hashmap, two-pointers, medium, leetcode, leetcode-763]
leetcode_number: 763
leetcode_url: https://leetcode.com/problems/partition-labels/
difficulty: Medium
permalink: /posts/kotlin-dsa-greedy-partition-labels/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [763 — Partition Labels](https://leetcode.com/problems/partition-labels/) |
| **Difficulty** | Medium |
| **Topic** | Greedy, String, HashMap, Two Pointers |

> Given a string `s`, partition it into as **many parts as possible**
> such that each letter appears in **at most one** part. Return a list
> of the sizes of these parts.

**Example:**
```
Input:  s = "ababcbacadefegdehijhklij"
Output: [9,7,8]
Explanation: "ababcbaca", "defegde", "hijhklij" — each letter (a,b,c in
             part 1; d,e,f,g in part 2; h,i,j,k,l in part 3) is fully
             contained within exactly one partition

Input:  s = "eccbbbbdec"
Output: [10]
```

**Constraints:**
- `1 <= s.length <= 500`
- `s` consists of lowercase English letters

---

## Approach

**Key insight — track the LAST occurrence of every character, then
greedily extend the current partition's boundary:** for the partition
to be valid, once we include a character, our partition's boundary must
extend at least as far as that character's **last** occurrence anywhere
in the string — otherwise, that character would appear again outside
this partition, violating the "each letter in at most one part" rule.

Scan left to right, maintaining the **farthest last-occurrence** seen so
far among all characters encountered in the current partition. The
moment the current index **reaches** that farthest boundary, the
partition is complete and can be closed — every character within it is
now guaranteed to never reappear later.

**Walk through `s = "eccbbbbdec"`** (lastOccurrence: e→8, c→9, b→6, d→7):
```
i=0 'e': lastOcc['e']=8 → boundary=max(0,8)=8
i=1 'c': lastOcc['c']=9 → boundary=max(8,9)=9
i=2 'c': lastOcc['c']=9 → boundary=max(9,9)=9
i=3 'b': lastOcc['b']=6 → boundary=max(9,6)=9
i=4 'b': boundary stays 9
i=5 'b': boundary stays 9
i=6 'b': boundary stays 9
i=7 'd': lastOcc['d']=7 → boundary=max(9,7)=9
i=8 'e': lastOcc['e']=8 → boundary=max(9,8)=9
i=9 'c': lastOcc['c']=9 → boundary=max(9,9)=9
  i(9) == boundary(9) → partition complete! size = 9-0+1 = 10

Result: [10] ✓ (the entire string is one partition, since 'c' both
                starts early and reappears at the very last position)
```

---

## Kotlin Solution

### Approach 1 — Two pointers, greedy boundary extension based on last occurrence (optimal)

```kotlin
fun partitionLabels(s: String): List<Int> {
    val lastOccurrence = HashMap<Char, Int>()
    for (i in s.indices) {
        lastOccurrence[s[i]] = i   // overwritten each time — ends up holding the LAST index
    }

    val result = mutableListOf<Int>()
    var start = 0
    var boundary = 0

    for (i in s.indices) {
        boundary = maxOf(boundary, lastOccurrence[s[i]]!!)

        if (i == boundary) {
            result.add(boundary - start + 1)
            start = i + 1
        }
    }

    return result
}
```

### Approach 2 — Same idea, using an IntArray(26) instead of a HashMap

```kotlin
fun partitionLabels(s: String): List<Int> {
    val lastOccurrence = IntArray(26)
    for (i in s.indices) {
        lastOccurrence[s[i] - 'a'] = i
    }

    val result = mutableListOf<Int>()
    var start = 0
    var boundary = 0

    for (i in s.indices) {
        boundary = maxOf(boundary, lastOccurrence[s[i] - 'a'])

        if (i == boundary) {
            result.add(boundary - start + 1)
            start = i + 1
        }
    }

    return result
}
```

Avoids HashMap overhead since the alphabet is fixed and small (lowercase
English letters only) — direct array indexing via `c - 'a'` is faster.

---

## Why `i == boundary` Is the Precise Moment to Close a Partition

This check is the heart of the greedy algorithm, and it's worth tracing
through why it's exactly correct, not just approximately so:

```kotlin
boundary = maxOf(boundary, lastOccurrence[s[i]]!!)

if (i == boundary) {
    result.add(boundary - start + 1)
    start = i + 1
}
```

`boundary` represents the **farthest point** any character seen so far
in this partition is guaranteed to need — it can only grow as we
encounter characters with later last-occurrences, never shrink. The
moment our scanning position `i` **catches up** to `boundary`, it means
every character we've seen since `start` has already had its last
occurrence accounted for by this point — there's nothing left that could
force the partition to extend further. This is the earliest possible
valid closing point, which is exactly why greedily closing here (rather
than waiting longer) maximizes the **number** of partitions, as required
by the problem.

**Why finding the maximum number of partitions specifically requires
closing as early as possible:** any later closing point would just
merge what could have been two separate valid partitions into one larger
one — strictly reducing the partition count. Closing the instant it's
valid to do so is what guarantees the maximum count.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| HashMap (Approach 1) | General-purpose, works for any character set |
| IntArray(26) (Approach 2) | Known lowercase-only constraint, marginally faster |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — one pass to record last occurrences, one pass to partition |
| **Space** | O(1) — fixed 26-character alphabet bound |

---

## Key Takeaway

> Partition Labels is solved by tracking how far the **current**
> partition must extend, based on the **last** occurrence of every
> character seen since the partition began — and closing the partition
> at the very first instant that boundary is satisfied. This "extend a
> boundary based on future-occurrence information, then close as soon as
> possible" technique is structurally similar to Jump Game II's
> level-boundary tracking, reinforcing how often greedy algorithms
> reduce to "maintain the tightest valid frontier, and commit the moment
> you reach it."

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/partition-labels/)

---

{% include kotlin-dsa-nav.html %}
