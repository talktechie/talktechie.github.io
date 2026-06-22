---
title: "Linked List: Remove Nth Node From End of List — Kotlin Solution"
date: 2026-06-20
categories: [Kotlin DSA, Linked List]
tags: [remove-nth-node-from-end-of-list, linked-list, two-pointers, medium, leetcode, leetcode-19]
leetcode_number: 19
leetcode_url: https://leetcode.com/problems/remove-nth-node-from-end-of-list/
difficulty: Medium
permalink: /posts/kotlin-dsa-linked-list-remove-nth-node-from-end-of-list/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [19 — Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) |
| **Difficulty** | Medium |
| **Topic** | Linked List, Two Pointers |

> Given the head of a linked list, remove the `n`-th node **from the end**
> of the list, and return the head.

**Example:**
```
Input:  head = [1,2,3,4,5], n = 2
Output: [1,2,3,5]
Explanation: removing the 2nd node from the end (value 4)

Input:  head = [1], n = 1
Output: []

Input:  head = [1,2], n = 1
Output: [1]
```

**Constraints:**
- The number of nodes is `sz`
- `1 <= sz <= 30`
- `0 <= Node.val <= 100`
- `1 <= n <= sz`
- **Follow-up:** Can you do this in one pass?

---

## Approach

The naive two-pass approach — count the list length first, then walk again
to the target node — works but requires walking the list twice. The
follow-up asks for **one pass**, which means we need a way to know "how far
from the end" we are without first knowing the total length.

**Key insight — gap of `n` between two pointers:** Advance a `fast` pointer
`n` steps ahead first. Then move `slow` and `fast` together, one step at a
time. When `fast` reaches the end, `slow` is exactly `n` nodes behind it —
positioned right before the node we need to remove.

Using a **dummy head** node avoids special-casing when the node to remove
is the actual head of the list.

**Walk through `head = [1,2,3,4,5], n = 2`:**
```
dummy → 1 → 2 → 3 → 4 → 5

fast starts at dummy, advances n+1=3 steps: dummy → 1 → 2 → 3
slow starts at dummy

Now move both together until fast reaches the end (null):
  fast: 3→4→5→null   slow: dummy→1→2→3

slow is now at node "3", slow.next is node "4" — the one to remove
slow.next = slow.next.next   → skips "4", connects "3" directly to "5"

Result: 1 → 2 → 3 → 5 ✓
```

---

## Kotlin Solution

### Approach 1 — One pass, dummy head + gap pointers (optimal)

```kotlin
class ListNode(var `val`: Int) {
    var next: ListNode? = null
}

fun removeNthFromEnd(head: ListNode?, n: Int): ListNode? {
    val dummy = ListNode(0)
    dummy.next = head

    var slow: ListNode? = dummy
    var fast: ListNode? = dummy

    // Advance fast n+1 steps ahead — this creates the gap of n nodes
    repeat(n + 1) {
        fast = fast?.next
    }

    // Move both together until fast falls off the end
    while (fast != null) {
        slow = slow?.next
        fast = fast?.next
    }

    // slow is now right before the node to remove
    slow?.next = slow?.next?.next

    return dummy.next
}
```

### Approach 2 — Two pass (find length, then remove)

```kotlin
fun removeNthFromEnd(head: ListNode?, n: Int): ListNode? {
    var length = 0
    var curr = head
    while (curr != null) {
        length++
        curr = curr.next
    }

    val dummy = ListNode(0)
    dummy.next = head
    var node: ListNode? = dummy

    // Walk to the node just before the one to remove
    repeat(length - n) {
        node = node?.next
    }

    node?.next = node?.next?.next

    return dummy.next
}
```

Simpler to reason about — count first, then compute exactly how many steps
to walk. Violates the one-pass follow-up, but a perfectly valid first
solution to write before optimizing.

---

## Why Advance `fast` by `n + 1`, Not `n`

This off-by-one detail is the part most people get wrong on the first try.

```kotlin
repeat(n + 1) {
    fast = fast?.next
}
```

We want `slow` to end up at the node **just before** the one to remove
(so we can do `slow.next = slow.next.next`). That requires a gap of `n + 1`
nodes between `slow` and `fast` — not `n` — because `slow` starts at the
dummy node, one position "before" the real list.

```
n=2, list: dummy → 1 → 2 → 3 → 4 → 5

If we advanced fast only n=2 steps: dummy → 1 → 2
Moving both together, slow would end up at node "2" when fast hits null —
that's one position too early; slow.next would be "3", not "4".

Advancing n+1=3 steps: dummy → 1 → 2 → 3
Moving together, slow correctly lands on node "3", with slow.next = "4" ✓
```

**The dummy head eliminates a special case:** without it, removing the
actual head node (e.g. `n == length`) would require separate logic to
update what "head" even refers to. With `dummy.next = head`, that case is
handled identically to every other case.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| One-pass gap pointers (Approach 1) | Always — meets the follow-up, same simplicity once understood |
| Two-pass length count (Approach 2) | First attempt while building intuition, or when one-pass isn't required |

---

## Complexity

| | One Pass | Two Pass |
|---|---|---|
| **Time** | O(n) — single traversal | O(n) — two traversals, same big-O but 2x the constant |
| **Space** | O(1) | O(1) |

---

## Key Takeaway

> "Nth from the end" problems are solved by creating a fixed **gap** between
> two pointers, then moving both together — when the lead pointer reaches
> the end, the trailing pointer is exactly the right distance back. A dummy
> head node sidesteps the awkward "what if we're removing the actual head"
> edge case entirely. This gap-pointer technique generalizes to many
> "k-th from the end" or "k-th from some boundary" problems.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)

---

{% include kotlin-dsa-nav.html %}
