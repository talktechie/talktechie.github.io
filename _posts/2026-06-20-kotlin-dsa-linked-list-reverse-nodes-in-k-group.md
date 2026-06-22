---
title: "Linked List: Reverse Nodes In K-Group — Kotlin Solution"
date: 2026-06-20
categories: [Kotlin DSA, Linked List]
tags: [reverse-nodes-in-k-group, linked-list, recursion, hard, leetcode, leetcode-25]
leetcode_number: 25
leetcode_url: https://leetcode.com/problems/reverse-nodes-in-k-group/
difficulty: Hard
permalink: /posts/kotlin-dsa-linked-list-reverse-nodes-in-k-group/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [25 — Reverse Nodes In K-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/) |
| **Difficulty** | Hard |
| **Topic** | Linked List, Recursion |

> Given the head of a linked list, reverse the nodes of the list **k at a
> time**, and return the modified list.
>
> `k` is a positive integer ≤ the list length. If the number of nodes
> isn't a multiple of `k`, the leftover nodes at the end remain in their
> original order (not reversed).
>
> You may not alter the node values — only reorder the nodes.

**Example:**
```
Input:  head = [1,2,3,4,5], k = 2
Output: [2,1,4,3,5]
Explanation: groups of 2 reversed: [1,2]→[2,1], [3,4]→[4,3], leftover [5] stays

Input:  head = [1,2,3,4,5], k = 3
Output: [3,2,1,4,5]
Explanation: [1,2,3]→[3,2,1], leftover [4,5] stays (not a full group of 3)
```

**Constraints:**
- The number of nodes is `n`
- `1 <= k <= n <= 5000`
- `0 <= Node.val <= 1000`
- **Follow-up:** Can you solve it in O(1) extra memory (not counting recursion stack)?

---

## Approach

This builds directly on the basic Reverse Linked List skill (LC 206), with
two new wrinkles: reversing only **k nodes at a time**, and **checking
ahead** that a full group of k nodes actually exists before reversing
(otherwise leave the remainder untouched).

**Key insight:** Process the list in chunks. For each chunk:
1. Check if at least `k` nodes remain — if not, stop and leave the rest as is.
2. Reverse exactly `k` nodes using the standard three-pointer technique.
3. Connect the **previous chunk's tail** to the **new head** of this
   reversed chunk, and recurse/continue on the remainder.

**Walk through `head = [1,2,3,4,5], k = 2`:**
```
Group 1: check nodes 1,2 exist (yes) → reverse → 2 → 1
         previous chunk's tail (none yet, this is the new head) → connects later

Group 2: check nodes 3,4 exist (yes) → reverse → 4 → 3
         group 1's tail (node "1") connects to group 2's new head (node "4")
         → 2 → 1 → 4 → 3

Group 3: check node 5 exists, but k=2 needs 2 nodes, only 1 left → stop, leave as-is
         group 2's tail (node "3") connects to the untouched remainder (node "5")
         → 2 → 1 → 4 → 3 → 5

Result: [2,1,4,3,5] ✓
```

---

## Kotlin Solution

### Approach 1 — Recursive (clean, matches the mental model directly)

```kotlin
class ListNode(var `val`: Int) {
    var next: ListNode? = null
}

fun reverseKGroup(head: ListNode?, k: Int): ListNode? {
    // Step 1 — check if at least k nodes remain
    var node = head
    var count = 0
    while (node != null && count < k) {
        node = node.next
        count++
    }
    if (count < k) return head   // fewer than k nodes left — leave untouched

    // Step 2 — reverse exactly k nodes
    var prev: ListNode? = reverseKGroup(node, k)  // recurse first — this is the "rest" after this group
    var curr = head
    repeat(k) {
        val next = curr!!.next
        curr!!.next = prev
        prev = curr
        curr = next
    }

    return prev  // prev is now the head of this reversed group
}
```

### Approach 2 — Iterative with dummy head (meets O(1) extra space)

```kotlin
fun reverseKGroup(head: ListNode?, k: Int): ListNode? {
    val dummy = ListNode(0)
    dummy.next = head
    var groupPrev: ListNode? = dummy

    while (true) {
        // Check if k nodes exist ahead of groupPrev
        var kth = groupPrev
        repeat(k) {
            kth = kth?.next
            if (kth == null) return dummy.next  // fewer than k remain — done
        }

        val groupNext = kth!!.next
        var prev = groupNext
        var curr = groupPrev!!.next

        // Reverse this group of k nodes
        while (curr != groupNext) {
            val next = curr!!.next
            curr.next = prev
            prev = curr
            curr = next
        }

        val tmp = groupPrev!!.next   // the original head of this group, now the tail after reversal
        groupPrev!!.next = kth        // connect previous group's tail to this group's new head
        groupPrev = tmp                // advance groupPrev to the end of the just-reversed group
    }
}
```

Iterative version avoids recursion stack space entirely, satisfying the
follow-up's O(1) extra memory requirement (the recursive version uses
O(n/k) stack frames).

---

## Why Check-Ahead Before Reversing Matters

The recursive version's first loop **doesn't reverse anything** — it only
walks forward to confirm `k` nodes exist:

```kotlin
var node = head
var count = 0
while (node != null && count < k) {
    node = node.next
    count++
}
if (count < k) return head   // not enough nodes — bail before touching any pointers
```

If we reversed first and *then* discovered there weren't enough nodes,
we'd have to un-reverse — much messier. Checking ahead avoids ever
mutating a group that turns out to be incomplete.

**The recursive call happens before the reversal in Approach 1** — this
is intentional:
```kotlin
var prev: ListNode? = reverseKGroup(node, k)  // recurse on "the rest" FIRST
// ... then reverse this group, using the already-solved "rest" as the tail target
```

This mirrors how Reverse Linked List's recursive version works — the
problem naturally recurses from the end backward, and each level only
needs to handle wiring its own group's tail to whatever the recursive
call already solved.

**`groupNext` in the iterative version** marks the boundary so the inner
reversal loop knows exactly when to stop, without needing a separate
counter:
```kotlin
while (curr != groupNext) { ... }
// reversal naturally halts at the start of the NEXT group, no off-by-one risk
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Recursive (Approach 1) | Clarity matters most — directly mirrors the conceptual "reverse, then recurse on rest" |
| Iterative with dummy head (Approach 2) | Follow-up requires O(1) extra space, no recursion stack allowed |

---

## Complexity

| | Recursive | Iterative |
|---|---|---|
| **Time** | O(n) | O(n) |
| **Space** | O(n/k) — recursion depth | O(1) |

---

## Key Takeaway

> This is Reverse Linked List (LC 206) applied repeatedly to fixed-size
> chunks, with a crucial guard: always confirm a full group of `k` nodes
> exists *before* reversing anything, so an incomplete trailing group is
> left untouched without needing to undo work. The recursive solution
> reads naturally as "reverse this group, then plug in whatever the rest
> of the list becomes" — but the iterative dummy-head version is what
> satisfies the O(1) space follow-up, making it the strongest answer to
> have ready for this problem, the hardest pure-pointer-manipulation
> problem in this phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/reverse-nodes-in-k-group/)

---

{% include kotlin-dsa-nav.html %}
