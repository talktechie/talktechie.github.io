---
title: "Linked List: Reverse Linked List — Kotlin Solution"
date: 2026-06-10
categories: [Kotlin DSA, Linked List]
tags: [reverse-linked-list, linked-list, two-pointer, easy, leetcode, leetcode-206]
leetcode_number: 206
leetcode_url: https://leetcode.com/problems/reverse-linked-list/
difficulty: Easy
permalink: /posts/kotlin-dsa-linked-list-reverse-linked-list/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [206 — Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/) |
| **Difficulty** | Easy |
| **Topic** | Linked List, Iteration, Recursion |

> Given the head of a singly linked list, reverse the list and return the reversed list.

**Example:**
```
Input:  1 → 2 → 3 → 4 → 5 → null
Output: 5 → 4 → 3 → 2 → 1 → null
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

Reversing a linked list means flipping every `next` pointer to point backward.

You need three references at all times:
- `prev` — the node behind current (starts as `null`)
- `curr` — the node you're currently processing
- `next` — save next node before overwriting `curr.next`

**Walk through `1 → 2 → 3`:**
```
prev=null, curr=1: save next=2, point 1→null, move prev=1, curr=2
prev=1,    curr=2: save next=3, point 2→1,    move prev=2, curr=3
prev=2,    curr=3: save next=null, point 3→2, move prev=3, curr=null
return prev → 3 → 2 → 1 → null ✓
```

---

## Kotlin Solution

### Approach 1 — Iterative

```kotlin
fun reverseList(head: ListNode?): ListNode? {
    var prev: ListNode? = null
    var curr = head

    while (curr != null) {
        val next = curr.next   // save next before overwriting
        curr.next = prev       // reverse the pointer
        prev = curr            // advance prev
        curr = next            // advance curr
    }

    return prev  // prev is the new head
}
```

### Approach 2 — Recursive

```kotlin
fun reverseList(head: ListNode?): ListNode? {
    if (head?.next == null) return head

    val newHead = reverseList(head.next)
    head.next!!.next = head  // reverse the pointer
    head.next = null         // disconnect old forward link

    return newHead
}
```

---

## Why Kotlin Shines Here

**Nullable types built in** — no separate null checks needed:
```kotlin
var prev: ListNode? = null
// Kotlin enforces null safety at compile time
// vs Java: no guarantee — NPE risk everywhere
```

**`head?.next == null`** — safe call in one expression:
```kotlin
if (head?.next == null) return head
// vs Java: if (head == null || head.next == null) return head;
```

**`val next = curr.next`** — `val` signals this is a one-time save:
```kotlin
val next = curr.next  // won't be reassigned — intent is clear
```

---

## Iterative vs Recursive

| Approach | Time | Space | Notes |
|---|---|---|---|
| Iterative | O(n) | O(1) | Preferred — no stack overhead |
| Recursive | O(n) | O(n) | Elegant but risks stack overflow on long lists |

---

## Common Mistake

```kotlin
// ❌ Forgetting to save next before reversing
curr.next = prev   // curr.next is now lost forever!
curr = curr.next   // this is now prev, not the original next
```

Always save `next` first — it's the very first line inside the loop.

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — visit every node once |
| **Space** | O(1) iterative / O(n) recursive |

---

## Key Takeaway

> Save next, reverse the pointer, advance both pointers.
> Three lines, three moves — that's the whole algorithm.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/reverse-linked-list/)

---

{% include kotlin-dsa-nav.html %}
