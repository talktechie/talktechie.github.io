---
title: "Linked List: LRU Cache — Kotlin Solution"
date: 2026-06-20
categories: [Kotlin DSA, Linked List]
tags: [lru-cache, linked-list, hashmap, doubly-linked-list, design, medium, leetcode, leetcode-146]
leetcode_number: 146
leetcode_url: https://leetcode.com/problems/lru-cache/
difficulty: Medium
permalink: /posts/kotlin-dsa-linked-list-lru-cache/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [146 — LRU Cache](https://leetcode.com/problems/lru-cache/) |
| **Difficulty** | Medium |
| **Topic** | Linked List, HashMap, Doubly Linked List, Design |

> Design a data structure that follows a **Least Recently Used (LRU)**
> cache with fixed `capacity`.
>
> - `get(key)` — return the value if it exists, else `-1`. Counts as a use.
> - `put(key, value)` — insert or update. If capacity is exceeded, evict
>   the **least recently used** entry first.
>
> Both `get` and `put` must run in **O(1) average time**.

**Example:**
```
Input:
["LRUCache","put","put","get","put","get","put","get","get","get"]
[[2],[1,1],[2,2],[1],[3,3],[2],[4,4],[1],[3],[4]]

Output:
[null,null,null,1,null,-1,null,-1,3,4]

Explanation:
LRUCache cache = new LRUCache(2);
cache.put(1, 1);   // cache: {1=1}
cache.put(2, 2);   // cache: {1=1, 2=2}
cache.get(1);      // returns 1, 1 becomes most recently used
cache.put(3, 3);   // capacity exceeded → evict 2 (LRU) → cache: {1=1, 3=3}
cache.get(2);      // returns -1 (evicted)
cache.put(4, 4);   // capacity exceeded → evict 1 (LRU) → cache: {3=3, 4=4}
cache.get(1);      // returns -1 (evicted)
cache.get(3);      // returns 3
cache.get(4);      // returns 4
```

**Constraints:**
- `1 <= capacity <= 3000`
- `0 <= key, value <= 10⁴`
- At most `2 * 10⁵` calls to `get` and `put`

---

## Approach

O(1) lookup screams **HashMap**. O(1) "move to most-recently-used position"
and O(1) "remove the least-recently-used item" screams **doubly linked
list** — arrays and singly linked lists can't remove an arbitrary middle
node in O(1) without already having a direct reference to it.

**Key insight — combine both:** Maintain a doubly linked list ordered by
recency (head = least recently used, tail = most recently used). The
HashMap stores `key → node reference`, giving instant access to any
node's position in the list without scanning.

Every `get` or `put` that touches an existing key **moves that node to the
tail** (most recently used). Every `put` that exceeds capacity **removes
the head node** (least recently used) — both O(1) with a doubly linked
list, since we have direct pointers to the node's neighbors.

**Walk through `capacity=2`, the example above:**
```
put(1,1): list: [1] (head=tail=1)         map={1: node1}
put(2,2): list: [1, 2]                     map={1: node1, 2: node2}
get(1):   move node1 to tail → list: [2, 1]   return 1
put(3,3): capacity full → evict head (2)
          list: [1, 3]   map={1: node1, 3: node3}
get(2):   not in map → return -1
put(4,4): capacity full → evict head (1)
          list: [3, 4]   map={3: node3, 4: node4}
get(1):   not in map → return -1
get(3):   move node3 to tail → list: [4, 3]   return 3
get(4):   move node4 to tail → list: [3, 4]   return 4
```

---

## Kotlin Solution

### Approach 1 — HashMap + hand-rolled doubly linked list (optimal)

```kotlin
class LRUCache(private val capacity: Int) {

    private class Node(val key: Int, var value: Int) {
        var prev: Node? = null
        var next: Node? = null
    }

    private val map = HashMap<Int, Node>()
    private val head = Node(-1, -1)  // dummy — least recently used end
    private val tail = Node(-1, -1)  // dummy — most recently used end

    init {
        head.next = tail
        tail.prev = head
    }

    private fun remove(node: Node) {
        node.prev?.next = node.next
        node.next?.prev = node.prev
    }

    private fun insertAtTail(node: Node) {
        val prevTail = tail.prev
        prevTail?.next = node
        node.prev = prevTail
        node.next = tail
        tail.prev = node
    }

    fun get(key: Int): Int {
        val node = map[key] ?: return -1
        remove(node)
        insertAtTail(node)   // mark as most recently used
        return node.value
    }

    fun put(key: Int, value: Int) {
        map[key]?.let { remove(it) }

        val node = Node(key, value)
        map[key] = node
        insertAtTail(node)

        if (map.size > capacity) {
            val lru = head.next!!
            remove(lru)
            map.remove(lru.key)
        }
    }
}
```

### Approach 2 — Kotlin's `LinkedHashMap` (built-in, leans on the standard library)

```kotlin
class LRUCache(private val capacity: Int) {
    private val map = object : LinkedHashMap<Int, Int>(capacity, 0.75f, true) {
        override fun removeEldestEntry(eldest: MutableMap.MutableEntry<Int, Int>?): Boolean {
            return size > capacity
        }
    }

    fun get(key: Int): Int = map[key] ?: -1

    fun put(key: Int, value: Int) {
        map[key] = value
    }
}
```

`LinkedHashMap`'s `accessOrder = true` constructor flag automatically
reorders entries by access time, and `removeEldestEntry` is a hook called
right after every insertion — exactly the eviction behavior we need, with
zero manual pointer management.

---

## Why Sentinel Head/Tail Nodes Simplify Everything

Using **dummy** head and tail nodes (rather than tracking nullable
`headNode`/`tailNode` references directly) eliminates almost every edge
case in this problem — empty list, single-element list, removing the only
node, etc.

```kotlin
init {
    head.next = tail
    tail.prev = head
}
// head and tail always exist, even when the cache is empty
// "the actual LRU node" is simply head.next — never head itself
// "the actual MRU node" is simply tail.prev — never tail itself
```

**Why `remove` then `insertAtTail`, not a single "move" operation:**
splitting it into two small, reusable functions makes both `get` (move
existing node to tail) and `put` (insert new node, or move existing node)
share the exact same building blocks:

```kotlin
fun get(key: Int): Int {
    val node = map[key] ?: return -1
    remove(node)
    insertAtTail(node)   // same two functions used in put() for existing keys
    return node.value
}
```

**Why check `map.size > capacity` after inserting**, not before — it's
simpler to always insert first, then evict if needed, rather than trying
to pre-check whether eviction is necessary:
```kotlin
insertAtTail(node)
if (map.size > capacity) {
    val lru = head.next!!
    remove(lru)
    map.remove(lru.key)
}
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Hand-rolled doubly linked list (Approach 1) | Interview — demonstrates understanding of the underlying mechanics |
| `LinkedHashMap` (Approach 2) | Production code, or when allowed to use language built-ins |

---

## Complexity

| | get | put |
|---|---|---|
| **Time** | O(1) | O(1) |
| **Space** | O(capacity) | |

---

## Key Takeaway

> O(1) lookup needs a HashMap. O(1) reordering and O(1) eviction from an
> arbitrary position needs a doubly linked list — arrays can't remove from
> the middle in O(1), and singly linked lists can't remove a node without
> already holding a reference to its predecessor. Combining both data
> structures, with the HashMap storing direct node references, is the
> classic "design a cache" answer, and dummy head/tail sentinels eliminate
> nearly every edge case you'd otherwise have to special-case.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/lru-cache/)

---

{% include kotlin-dsa-nav.html %}
