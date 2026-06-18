---
title: "Stack: Min Stack — Kotlin Solution"
date: 2026-06-17
categories: [Kotlin DSA, Stack]
tags: [min-stack, stack, design, medium, leetcode, leetcode-155]
leetcode_number: 155
leetcode_url: https://leetcode.com/problems/min-stack/
difficulty: Medium
permalink: /posts/kotlin-dsa-stack-min-stack/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [155 — Min Stack](https://leetcode.com/problems/min-stack/) |
| **Difficulty** | Medium |
| **Topic** | Stack, Design |

> Design a stack that supports `push`, `pop`, `top`, and retrieving the
> minimum element — all in **O(1) time**.
>
> Implement the `MinStack` class:
> - `push(val)` — push the element onto the stack
> - `pop()` — remove the top element
> - `top()` — get the top element
> - `getMin()` — retrieve the minimum element in the stack

**Example:**
```
Input:
["MinStack","push","push","push","getMin","pop","top","getMin"]
[[],[-2],[0],[-3],[],[],[],[]]

Output:
[null,null,null,null,-3,null,0,-2]

Explanation:
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.getMin(); // return -3
minStack.pop();
minStack.top();    // return 0
minStack.getMin(); // return -2
```

**Constraints:**
- `-2³¹ <= val <= 2³¹ - 1`
- `pop`, `top`, `getMin` are only called on non-empty stacks
- At most `3 * 10⁴` calls to push, pop, top, and getMin

---

## Approach

The naive idea — scan the whole stack for the minimum on every `getMin()`
call — is O(n), not O(1). We need the minimum available instantly at any point.

**Key insight — track the running minimum alongside each element:**
Use a second stack that stores, at each position, the **minimum seen so far
up to and including that push**. When we pop the main stack, we pop the min
stack too — the new top of the min stack is automatically the correct minimum
for the remaining elements.

**Walk through push(-2), push(0), push(-3):**
```
push(-2): mainStack = [-2]        minStack = [-2]            (min so far: -2)
push(0):  mainStack = [-2, 0]     minStack = [-2, -2]         (min so far: still -2)
push(-3): mainStack = [-2, 0, -3] minStack = [-2, -2, -3]     (min so far: -3)

getMin() → minStack.last() = -3 ✓

pop(): mainStack = [-2, 0]  minStack = [-2, -2]
top()  → mainStack.last() = 0 ✓
getMin() → minStack.last() = -2 ✓
```

---

## Kotlin Solution

### Approach 1 — Two stacks, track running min (optimal)

```kotlin
class MinStack {
    private val stack = ArrayDeque<Int>()
    private val minStack = ArrayDeque<Int>()

    fun push(val_: Int) {
        stack.addLast(val_)
        val currentMin = if (minStack.isEmpty()) val_ else minOf(val_, minStack.last())
        minStack.addLast(currentMin)
    }

    fun pop() {
        stack.removeLast()
        minStack.removeLast()
    }

    fun top(): Int = stack.last()

    fun getMin(): Int = minStack.last()
}
```

Every operation is O(1) — no scanning, no recomputation. Both stacks stay
in sync, always the same size.

### Approach 2 — Single stack storing (value, minSoFar) pairs

```kotlin
class MinStack {
    private val stack = ArrayDeque<Pair<Int, Int>>()  // (value, minSoFar)

    fun push(val_: Int) {
        val currentMin = if (stack.isEmpty()) val_ else minOf(val_, stack.last().second)
        stack.addLast(val_ to currentMin)
    }

    fun pop() {
        stack.removeLast()
    }

    fun top(): Int = stack.last().first

    fun getMin(): Int = stack.last().second
}
```

Same idea, one stack instead of two — uses Kotlin's `Pair` and infix `to`.
Slightly less memory-friendly due to boxing pairs, but a single data structure
to reason about.

---

## Why This Achieves O(1) for getMin

The trick is **redundant storage** — instead of computing the minimum on
demand, we precompute and store it at the time of every push.

```kotlin
val currentMin = if (minStack.isEmpty()) val_ else minOf(val_, minStack.last())
minStack.addLast(currentMin)
// minStack[i] always holds: min(stack[0], stack[1], ..., stack[i])
// So minStack.last() is always the minimum of everything currently in the stack
```

When we pop, the min stack pops too — its new top reflects the minimum of
whatever's left, with zero recomputation:

```kotlin
fun pop() {
    stack.removeLast()
    minStack.removeLast()  // automatically "restores" the previous minimum
}
```

**Trailing underscore in `val_`** — Kotlin reserves `val` as a keyword:
```kotlin
fun push(val_: Int) { ... }
// Can't name a parameter "val" — Kotlin convention is to append underscore
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Two stacks (Approach 1) | Cleaner separation, easier to debug each independently |
| Single stack of pairs (Approach 2) | Want one data structure, comfortable with Pair overhead |

---

## Complexity

| | Push | Pop | Top | getMin |
|---|---|---|---|---|
| **Time** | O(1) | O(1) | O(1) | O(1) |
| **Space** | O(n) total across all stored elements | | | |

---

## Key Takeaway

> When an operation needs O(1) lookup but naturally requires O(n) recomputation,
> ask: can I precompute and store the answer at write time instead of read time?
> Here, storing "minimum so far" alongside every push turns getMin() from an
> O(n) scan into an O(1) stack peek. This redundant-storage pattern shows up
> in many stack-design problems beyond this one.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/min-stack/)

---

{% include kotlin-dsa-nav.html %}
