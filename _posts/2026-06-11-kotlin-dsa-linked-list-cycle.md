---
title: "Linked List: Linked List Cycle — Kotlin Solution"
date: 2026-06-10
categories: [Kotlin DSA, Linked List]
tags: [linked-list-cycle, floyd, fast-slow-pointer, two-pointer, easy, leetcode, leetcode-141]
leetcode_number: 141
leetcode_url: https://leetcode.com/problems/linked-list-cycle/
difficulty: Easy
permalink: /posts/kotlin-dsa-linked-list-cycle/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [141 — Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/) |
| **Difficulty** | Easy |
| **Topic** | Linked List, Floyd's Algorithm, Fast & Slow Pointers |

> Given the head of a linked list, determine if the list has a cycle.
> Return `true` if there is a cycle, `false` otherwise.

**Example:**
```
Input:  3 → 2 → 0 → -4
                ↑________↑   (tail connects back to node at index 1)

Output: true
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

**Approach 1 — HashSet:**
Track every visited node. If you see the same node twice — cycle.
Simple, O(n) space.

**Approach 2 — Floyd's Cycle Detection:**
Two pointers — `slow` moves one step, `fast` moves two.

- No cycle → `fast` reaches `null`
- Cycle → `fast` laps `slow`, they meet inside the cycle

Like two runners on a circular track — the faster one always laps the slower one.

**Walk through:**
```
List: 3 → 2 → 0 → -4 → (back to 2)

Step 1: slow=2,  fast=0
Step 2: slow=0,  fast=2   (fast: -4 → 2)
Step 3: slow=-4, fast=-4  ← meet! cycle detected ✓
```

---

## Kotlin Solution

### Approach 1 — HashSet

```kotlin
fun hasCycle(head: ListNode?): Boolean {
    val visited = HashSet<ListNode>()
    var current = head

    while (current != null) {
        if (current in visited) return true
        visited.add(current)
        current = current.next
    }

    return false
}
```

### Approach 2 — Floyd's Fast & Slow Pointers (optimal)

```kotlin
fun hasCycle(head: ListNode?): Boolean {
    var slow = head
    var fast = head

    while (fast?.next != null) {
        slow = slow?.next
        fast = fast.next?.next

        if (slow == fast) return true
    }

    return false
}
```

---

## Why Kotlin Shines Here

**`fast?.next != null`** — safe call handles both null cases:
```kotlin
while (fast?.next != null)
// Stops if fast is null OR fast.next is null
// vs Java: while (fast != null && fast.next != null)
```

**`fast.next?.next`** — double hop safely:
```kotlin
fast = fast.next?.next
// If fast.next is null → fast becomes null safely, no NPE
// vs Java: fast = fast.next.next; — NPE risk
```

**`current in visited`** — readable set lookup:
```kotlin
if (current in visited) return true
// vs Java: if (visited.contains(current))
```

**`slow == fast`** compares references not values — exactly what we need:
```kotlin
if (slow == fast) return true
// ListNode has no custom equals() → defaults to reference equality ✓
// Two different nodes with same val would NOT be equal here
```

---

## Two Approaches Compared

| Approach | Time | Space | Notes |
|---|---|---|---|
| HashSet | O(n) | O(n) | Simple, intuitive |
| Floyd's | O(n) | O(1) | Optimal — interview standard |

Always go with **Floyd's** in interviews. O(1) space and demonstrates
knowledge of the classic algorithm.

---

## Why Does Floyd's Always Work?

Once `fast` enters the cycle, the gap between `fast` and `slow`
decreases by exactly 1 each step (fast gains 2, slow gains 1).
So they must meet within `cycle_length` steps — guaranteed.

If no cycle, `fast` reaches `null` in O(n/2) steps and the loop exits.

---

## Complexity

| | |
|---|---|
| **Time** | O(n) |
| **Space** | O(1) Floyd's / O(n) HashSet |

---

## Key Takeaway

> Fast pointer moves twice as fast. If there's a cycle it laps slow —
> they must meet. If no cycle, fast falls off the end.
> Two pointers, zero extra space.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/linked-list-cycle/)

---

{% include kotlin-dsa-nav.html %}
