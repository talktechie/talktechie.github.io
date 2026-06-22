---
title: "Linked List: Merge Two Sorted Lists — Kotlin Solution"
date: 2026-06-10
categories: [Kotlin DSA, Linked List]
tags: [merge-two-sorted-lists, linked-list, recursion, easy, leetcode, leetcode-21]
leetcode_number: 21
leetcode_url: https://leetcode.com/problems/merge-two-sorted-lists/
difficulty: Easy
permalink: /posts/kotlin-dsa-linkedlist-merge-two-sorted-lists/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [21 — Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/) |
| **Difficulty** | Easy |
| **Topic** | Linked List, Recursion |

> Given the heads of two sorted linked lists `list1` and `list2`,
> merge them into one sorted list and return the head.

**Example:**
```
Input:  list1 = 1 → 2 → 4
        list2 = 1 → 3 → 4

Output: 1 → 1 → 2 → 3 → 4 → 4
```

---

## The Node Definition

```kotlin
class ListNode(var `val`: Int) {
    var next: ListNode? = null
}
```

---

## Approach

Like merging two sorted arrays — but with pointers instead of indices.

**The dummy node trick** — create a fake head to avoid special-casing
an empty result list.

**Walk through:**
```
list1: 1 → 2 → 4
list2: 1 → 3 → 4

dummy → ?
Compare 1 vs 1: pick list1(1), list1 = 2 → 4
Compare 2 vs 1: pick list2(1), list2 = 3 → 4
Compare 2 vs 3: pick list1(2), list1 = 4
Compare 4 vs 3: pick list2(3), list2 = 4
Compare 4 vs 4: pick list1(4), list1 = null
Remaining: append list2(4)

Result: 1 → 1 → 2 → 3 → 4 → 4 ✓
```

---

## Kotlin Solution

### Approach 1 — Iterative (with dummy node)

```kotlin
fun mergeTwoLists(list1: ListNode?, list2: ListNode?): ListNode? {
    val dummy = ListNode(0)
    var current: ListNode = dummy
    var l1 = list1
    var l2 = list2

    while (l1 != null && l2 != null) {
        if (l1.`val` <= l2.`val`) {
            current.next = l1
            l1 = l1.next
        } else {
            current.next = l2
            l2 = l2.next
        }
        current = current.next!!
    }

    current.next = l1 ?: l2  // attach remaining nodes

    return dummy.next
}
```

### Approach 2 — Recursive

```kotlin
fun mergeTwoLists(list1: ListNode?, list2: ListNode?): ListNode? {
    if (list1 == null) return list2
    if (list2 == null) return list1

    return if (list1.`val` <= list2.`val`) {
        list1.next = mergeTwoLists(list1.next, list2)
        list1
    } else {
        list2.next = mergeTwoLists(list1, list2.next)
        list2
    }
}
```

---

## Why Kotlin Shines Here

**Elvis operator `?:`** — attach remaining list in one line:
```kotlin
current.next = l1 ?: l2
// vs Java: current.next = (l1 != null) ? l1 : l2;
```

**`if` as expression** — recursive solution reads as pure logic:
```kotlin
return if (list1.`val` <= list2.`val`) {
    list1.next = mergeTwoLists(list1.next, list2)
    list1
} else { ... }
// No ternary gymnastics needed
```

**`current.next!!`** — we just assigned `current.next` above,
safe to use `!!` here:
```kotlin
current = current.next!!
```

---

## Why the Dummy Node?

```kotlin
// Without dummy — messy special case
var head: ListNode? = null
// need to check if head is null on first iteration...

// With dummy — clean always
val dummy = ListNode(0)
var current: ListNode = dummy
// current is always non-null — no branching needed
```

One allocation, saves a lot of branching.

---

## Complexity

| | |
|---|---|
| **Time** | O(m + n) — visit every node in both lists |
| **Space** | O(1) iterative / O(m+n) recursive |

---

## Key Takeaway

> Dummy head eliminates empty-list edge cases. Compare, attach
> the smaller node, advance. Attach the remaining non-null list at the end.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/merge-two-sorted-lists/)

---

{% include kotlin-dsa-nav.html %}
