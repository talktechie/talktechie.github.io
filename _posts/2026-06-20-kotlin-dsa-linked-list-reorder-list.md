---
title: "Linked List: Reorder List — Kotlin Solution"
date: 2026-06-20
categories: [Kotlin DSA, Linked List]
tags: [reorder-list, linked-list, two-pointers, recursion, medium, leetcode, leetcode-143]
leetcode_number: 143
leetcode_url: https://leetcode.com/problems/reorder-list/
difficulty: Medium
permalink: /posts/kotlin-dsa-linked-list-reorder-list/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [143 — Reorder List](https://leetcode.com/problems/reorder-list/) |
| **Difficulty** | Medium |
| **Topic** | Linked List, Two Pointers, Recursion |

> You are given the head of a singly linked list. Reorder it to follow:
> `L0 → Ln → L1 → Ln-1 → L2 → Ln-2 → ...`
>
> You may not modify the values in the list's nodes — only the node
> ordering. Modify the list in place.

**Example:**
```
Input:  1 → 2 → 3 → 4
Output: 1 → 4 → 2 → 3

Input:  1 → 2 → 3 → 4 → 5
Output: 1 → 5 → 2 → 4 → 3
```

**Constraints:**
- `1 <= list.length <= 5 * 10⁴`
- `1 <= Node.val <= 1000`

---

## Approach

This problem combines three skills covered earlier in this phase: finding
the middle with fast/slow pointers, reversing a list, and merging two lists.

**Key insight — three steps:**
1. Find the **middle** of the list using fast and slow pointers.
2. **Reverse** the second half.
3. **Merge** the two halves by alternating nodes — one from the front, one
   from the reversed back.

**Walk through `1 → 2 → 3 → 4 → 5`:**
```
Step 1 — find middle (slow stops at 3):
  first half:  1 → 2 → 3
  second half: 4 → 5

Step 2 — reverse second half:
  4 → 5  becomes  5 → 4

Step 3 — merge alternately:
  take 1 (first) → take 5 (second) → take 2 (first) → take 4 (second) → take 3 (first)
  result: 1 → 5 → 2 → 4 → 3 ✓
```

---

## Kotlin Solution

### Approach 1 — Find middle, reverse, merge (optimal, O(1) space)

```kotlin
class ListNode(var `val`: Int) {
    var next: ListNode? = null
}

fun reorderList(head: ListNode?) {
    if (head?.next == null) return

    // Step 1 — find the middle using fast/slow pointers
    var slow = head
    var fast = head
    while (fast?.next != null && fast.next?.next != null) {
        slow = slow?.next
        fast = fast.next?.next
    }

    // Step 2 — reverse the second half
    var prev: ListNode? = null
    var curr = slow?.next
    slow?.next = null  // cut the first half off from the second
    while (curr != null) {
        val next = curr.next
        curr.next = prev
        prev = curr
        curr = next
    }
    val secondHead = prev

    // Step 3 — merge the two halves alternately
    var first = head
    var second = secondHead
    while (second != null) {
        val firstNext = first?.next
        val secondNext = second.next

        first?.next = second
        second.next = firstNext

        first = firstNext
        second = secondNext
    }
}
```

### Approach 2 — Convert to ArrayList, rebuild with two pointers

```kotlin
fun reorderList(head: ListNode?) {
    val nodes = mutableListOf<ListNode>()
    var curr = head
    while (curr != null) {
        nodes.add(curr)
        curr = curr.next
    }

    var left = 0
    var right = nodes.lastIndex
    while (left < right) {
        nodes[left].next = nodes[right]
        left++
        if (left == right) break
        nodes[right].next = nodes[left]
        right--
    }
    nodes[left].next = null  // terminate the list
}
```

Uses O(n) extra space to store node references, but the merge logic
becomes simple two-pointer index manipulation. Easier to get right under
interview pressure, at the cost of space complexity.

---

## Why We Cut the List Before Reversing

A subtle but critical step: `slow?.next = null` disconnects the first half
from the second **before** reversing the second half.

```kotlin
var curr = slow?.next   // save reference to start of second half first
slow?.next = null        // now cut the connection
while (curr != null) { ... }  // reverse using the saved reference
```

Without this cut, the first half would still technically be "connected"
to the original list structure even after reversal, causing either an
infinite loop or incorrect merging.

**The merge loop alternates next pointers in pairs:**
```kotlin
val firstNext = first?.next   // save before overwriting
val secondNext = second.next

first?.next = second   // 1 → 5
second.next = firstNext // 5 → 2 (the original "rest of first half")

first = firstNext   // advance to 2
second = secondNext // advance to 4
```

Saving `firstNext` and `secondNext` **before** any reassignment is what
prevents losing track of where to continue — a classic linked-list
"save before you overwrite" pattern.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Find/reverse/merge (Approach 1) | Always — true O(1) space, the expected interview solution |
| ArrayList rebuild (Approach 2) | Want simpler index-based logic, O(n) space is acceptable |

---

## Complexity

| | Approach 1 | Approach 2 |
|---|---|---|
| **Time** | O(n) | O(n) |
| **Space** | O(1) | O(n) |

---

## Key Takeaway

> Reorder List is a composition problem — it doesn't introduce a new
> technique, it combines three you already know: fast/slow pointers to
> find the middle, iterative reversal, and alternating merge. Recognizing
> that a hard-looking problem decomposes into familiar sub-steps is itself
> a skill worth building. Always disconnect the two halves before
> reversing one of them, and save "next" references before overwriting
> pointers during the merge.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/reorder-list/)

---

{% include kotlin-dsa-nav.html %}
