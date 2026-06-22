---
title: "Linked List: Add Two Numbers — Kotlin Solution"
date: 2026-06-20
categories: [Kotlin DSA, Linked List]
tags: [add-two-numbers, linked-list, math, medium, leetcode, leetcode-2]
leetcode_number: 2
leetcode_url: https://leetcode.com/problems/add-two-numbers/
difficulty: Medium
permalink: /posts/kotlin-dsa-linked-list-add-two-numbers/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [2 — Add Two Numbers](https://leetcode.com/problems/add-two-numbers/) |
| **Difficulty** | Medium |
| **Topic** | Linked List, Math |

> You are given two non-empty linked lists representing two non-negative
> integers. The digits are stored in **reverse order**, and each node
> contains a single digit.
>
> Add the two numbers and return the sum as a linked list, in the same
> reverse-digit format.

**Example:**
```
Input:  l1 = [2,4,3], l2 = [5,6,4]
Output: [7,0,8]
Explanation: 342 + 465 = 807, stored as 7 → 0 → 8

Input:  l1 = [0], l2 = [0]
Output: [0]

Input:  l1 = [9,9,9,9,9,9,9], l2 = [9,9,9,9]
Output: [8,9,9,9,0,0,0,1]
Explanation: 9999999 + 9999 = 10009998
```

**Constraints:**
- Each list contains `1` to `100` nodes
- `0 <= Node.val <= 9`
- No leading zeros except for the number `0` itself

---

## Approach

Storing digits in reverse order is actually a gift — it means the
**least significant digit comes first**, exactly the order you'd add
numbers by hand starting from the ones place. No need to reverse anything.

**Key insight:** Walk both lists simultaneously, adding corresponding
digits plus any carry from the previous step. Build the result list one
node at a time. Continue as long as either list still has digits, **or**
there's a leftover carry to account for.

**Walk through `l1 = [9,9,9,9,9,9,9], l2 = [9,9,9,9]`:**
```
digit 0: 9+9+0(carry) = 18 → write 8, carry=1
digit 1: 9+9+1(carry) = 19 → write 9, carry=1
digit 2: 9+9+1(carry) = 19 → write 9, carry=1
digit 3: 9+9+1(carry) = 19 → write 9, carry=1
digit 4: 9+0+1(carry) = 10 → write 0, carry=1   (l2 exhausted, treat as 0)
digit 5: 9+0+1(carry) = 10 → write 0, carry=1
digit 6: 9+0+1(carry) = 10 → write 0, carry=1
digit 7: 0+0+1(carry) = 1  → write 1, carry=0   (both exhausted, but carry remains)

Result: [8,9,9,9,0,0,0,1] → represents 89990001... wait, in reverse digit
order this reads as 1,0,0,0,9,9,9,8 → 10009998 ✓
```

---

## Kotlin Solution

### Approach 1 — Dummy head, single pass with carry (optimal)

```kotlin
class ListNode(var `val`: Int) {
    var next: ListNode? = null
}

fun addTwoNumbers(l1: ListNode?, l2: ListNode?): ListNode? {
    val dummy = ListNode(0)
    var curr = dummy
    var carry = 0
    var p1 = l1
    var p2 = l2

    while (p1 != null || p2 != null || carry != 0) {
        val sum = (p1?.`val` ?: 0) + (p2?.`val` ?: 0) + carry
        carry = sum / 10
        curr.next = ListNode(sum % 10)
        curr = curr.next!!

        p1 = p1?.next
        p2 = p2?.next
    }

    return dummy.next
}
```

### Approach 2 — Recursive

```kotlin
fun addTwoNumbers(l1: ListNode?, l2: ListNode?, carry: Int = 0): ListNode? {
    if (l1 == null && l2 == null && carry == 0) return null

    val sum = (l1?.`val` ?: 0) + (l2?.`val` ?: 0) + carry
    val node = ListNode(sum % 10)
    node.next = addTwoNumbers(l1?.next, l2?.next, sum / 10)

    return node
}
```

Clean and expressive — the carry is threaded through as a parameter
instead of a mutable variable. Uses O(n) call stack space, where n is the
length of the longer list.

---

## Why the Loop Condition Includes `|| carry != 0`

This is the detail that's easy to forget — and the example above
(`9999999 + 9999`) is specifically designed to expose it. Even after both
lists are fully consumed, a leftover carry (from a final digit sum ≥ 10)
still needs one more node in the result.

```kotlin
while (p1 != null || p2 != null || carry != 0) { ... }
// Without "|| carry != 0", a final carry like the one in 999...+999... 
// would be silently dropped, producing a wrong (too-short) result
```

**Treating a missing node as 0** with the elvis operator avoids
null-checking and branching for lists of different lengths:
```kotlin
val sum = (p1?.`val` ?: 0) + (p2?.`val` ?: 0) + carry
// If p1 is null (its list ran out), contribute 0 instead of crashing
```

**Dummy head pattern again** — same trick as Merge Two Sorted Lists,
avoiding special-casing the very first node of the result:
```kotlin
val dummy = ListNode(0)
var curr = dummy
// ...
return dummy.next   // skip the placeholder, return the real head
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Iterative with dummy head (Approach 1) | Always preferred — O(1) extra space, no recursion depth limit |
| Recursive (Approach 2) | Elegant for short lists, demonstrates threading state through recursion |

---

## Complexity

| | |
|---|---|
| **Time** | O(max(m, n)) — m, n are the lengths of the two lists |
| **Space** | O(max(m, n)) — for the result list (unavoidable); O(1) extra otherwise (iterative) |

---

## Key Takeaway

> Reverse-order digit storage is convenient, not a complication — addition
> naturally proceeds least-significant-digit first. Track a carry across
> iterations exactly like grade-school long addition, and don't forget the
> loop must continue if a carry remains even after both input lists are
> exhausted. The dummy head and "missing node treated as 0" patterns from
> earlier problems make another appearance here.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/add-two-numbers/)

---

{% include kotlin-dsa-nav.html %}
