---
title: "Linked List: Copy List With Random Pointer — Kotlin Solution"
date: 2026-06-20
categories: [Kotlin DSA, Linked List]
tags: [copy-list-with-random-pointer, linked-list, hashmap, medium, leetcode, leetcode-138]
leetcode_number: 138
leetcode_url: https://leetcode.com/problems/copy-list-with-random-pointer/
difficulty: Medium
permalink: /posts/kotlin-dsa-linked-list-copy-list-with-random-pointer/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [138 — Copy List With Random Pointer](https://leetcode.com/problems/copy-list-with-random-pointer/) |
| **Difficulty** | Medium |
| **Topic** | Linked List, HashMap |

> A linked list has each node containing an additional **random** pointer
> which could point to any node in the list, or `null`.
>
> Construct a **deep copy** of the list — the copy must consist of
> exactly `n` brand-new nodes, where each new node's `val` matches its
> original, and both `next` and `random` pointers point to new nodes in
> the copied list (never back to the original).

**Example:**
```
Input:  head = [[7,null],[13,0],[11,4],[10,2],[1,0]]
        (each pair is [val, random_index], null means no random pointer)
Output: [[7,null],[13,0],[11,4],[10,2],[1,0]]
        (a fully independent deep copy with the same structure)
```

**Constraints:**
- `0 <= n <= 1000`
- `-10⁴ <= Node.val <= 10⁴`
- `Node.random` is `null` or points to a node in the list

---

## Approach

The challenge: `random` pointers can point **forward** to nodes we haven't
created copies of yet. We need a way to reference "the copy of node X"
even before we've fully processed node X's own pointers.

**Key insight — two-pass with a HashMap:**
1. **First pass:** create a copy of every node (with `next = null`,
   `random = null` for now), and map `original node → copy node`.
2. **Second pass:** walk the original list again, and for each original
   node, set its copy's `next` and `random` pointers using the map — every
   lookup is guaranteed to already exist because pass 1 created copies
   for all nodes upfront.

**Walk through `[7,null] → [13,0] → [11,4] → [10,2] → [1,0]`** (indices
0-4, random points by index):
```
Pass 1 — create copies, build map:
  map[node0] = copy(val=7)
  map[node1] = copy(val=13)
  map[node2] = copy(val=11)
  map[node3] = copy(val=10)
  map[node4] = copy(val=1)

Pass 2 — wire up next and random using the map:
  copy(7).next = map[node1] = copy(13)     copy(7).random = null
  copy(13).next = map[node2] = copy(11)    copy(13).random = map[node0] = copy(7)
  copy(11).next = map[node3] = copy(10)    copy(11).random = map[node4] = copy(1)
  copy(10).next = map[node4] = copy(1)     copy(10).random = map[node2] = copy(11)
  copy(1).next = null                      copy(1).random = map[node0] = copy(7)

Result: a fully independent copy with identical structure ✓
```

---

## Kotlin Solution

### Approach 1 — Two-pass HashMap (optimal, easiest to reason about)

```kotlin
class Node(var `val`: Int) {
    var next: Node? = null
    var random: Node? = null
}

fun copyRandomList(head: Node?): Node? {
    if (head == null) return null

    val map = HashMap<Node, Node>()

    // Pass 1 — create all copies, no pointers wired yet
    var curr = head
    while (curr != null) {
        map[curr] = Node(curr.`val`)
        curr = curr.next
    }

    // Pass 2 — wire up next and random using the map
    curr = head
    while (curr != null) {
        map[curr]?.next = curr.next?.let { map[it] }
        map[curr]?.random = curr.random?.let { map[it] }
        curr = curr.next
    }

    return map[head]
}
```

### Approach 2 — Single pass, interleaved nodes (O(1) extra space)

```kotlin
fun copyRandomList(head: Node?): Node? {
    if (head == null) return null

    // Step 1 — interleave copies: original1 -> copy1 -> original2 -> copy2 -> ...
    var curr = head
    while (curr != null) {
        val copy = Node(curr.`val`)
        copy.next = curr.next
        curr.next = copy
        curr = copy.next
    }

    // Step 2 — wire up random pointers using the interleaved structure
    curr = head
    while (curr != null) {
        curr.next?.random = curr.random?.next
        curr = curr.next?.next
    }

    // Step 3 — unweave: separate original list from copy list
    curr = head
    val dummy = Node(0)
    var copyTail: Node? = dummy
    while (curr != null) {
        val copy = curr.next
        curr.next = copy?.next
        copyTail?.next = copy
        copyTail = copy
        curr = curr.next
    }

    return dummy.next
}
```

No HashMap needed — copies are temporarily threaded directly into the
original list (`orig → copy → orig → copy...`), which lets `curr.random.next`
serve as "the copy of curr's random target," since every original node's
`next` now points to its own copy.

---

## Why `curr.random?.next` Gives the Copy's Random Target

This is the clever trick in Approach 2. After step 1, every original
node's `next` points to *its own copy*. So if `curr.random` points to some
original node `X`, then `X.next` is `X`'s copy — exactly what we want
`copy.random` to point to.

```kotlin
curr.next?.random = curr.random?.next
// curr.next is the copy of curr
// curr.random is some original node X (or null)
// curr.random?.next is X's copy (or null if curr.random was null)
```

**Why pass 1 must fully finish before pass 2 starts (Approach 1):** if we
tried to wire up `random` pointers while still creating copies, a forward
reference (a node's `random` pointing to a node we haven't copied yet)
would fail — the map wouldn't have an entry for it yet.

```kotlin
// Pass 1 — ALL copies must exist before any wiring happens
while (curr != null) {
    map[curr] = Node(curr.`val`)
    curr = curr.next
}
// Only now is every node guaranteed to have a copy ready in the map
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Two-pass HashMap (Approach 1) | Clearer to write and explain, O(n) space is fine |
| Interleaved single-pass (Approach 2) | O(1) extra space is required, comfortable with more intricate pointer surgery |

---

## Complexity

| | Two-Pass HashMap | Interleaved |
|---|---|---|
| **Time** | O(n) | O(n) |
| **Space** | O(n) — the map | O(1) extra (beyond the n new nodes, which are unavoidable) |

---

## Key Takeaway

> When pointers can reference nodes that don't exist yet (forward
> references), separate the "create" phase from the "wire up" phase. A
> HashMap mapping original→copy makes this trivial and is the answer
> most interviewers expect. The O(1)-space version is a neat trick worth
> knowing — temporarily weaving copies into the original list turns
> "find the copy of this node" into a simple `.next` dereference, no
> map required.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/copy-list-with-random-pointer/)

---

{% include kotlin-dsa-nav.html %}
