---
title: "Stack: Valid Parentheses — Kotlin Solution"
date: 2026-06-17
categories: [Kotlin DSA, Stack]
tags: [valid-parentheses, stack, string, easy, leetcode, leetcode-20]
leetcode_number: 20
leetcode_url: https://leetcode.com/problems/valid-parentheses/
difficulty: Easy
permalink: /posts/kotlin-dsa-stack-valid-parentheses/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [20 — Valid Parentheses](https://leetcode.com/problems/valid-parentheses/) |
| **Difficulty** | Easy |
| **Topic** | Stack, String |

> Given a string `s` containing just the characters `'('`, `')'`, `'{'`,
> `'}'`, `'['`, and `']'`, determine if the input string is valid.
>
> A string is valid if:
> 1. Open brackets are closed by the same type of bracket.
> 2. Open brackets are closed in the correct order.
> 3. Every closing bracket has a corresponding open bracket of the same type.

**Example:**
```
Input:  s = "()"
Output: true

Input:  s = "()[]{}"
Output: true

Input:  s = "(]"
Output: false

Input:  s = "([])"
Output: true
```

**Constraints:**
- `1 <= s.length <= 10⁴`
- `s` consists only of the characters `'()[]{}'`

---

## Approach

This is the textbook problem that introduces the Stack data structure to most
people learning DSA — and for good reason. The "closed in the correct order"
rule is exactly what a stack enforces naturally: **last opened, first closed**.

**Key insight:** Push every opening bracket onto a stack. When a closing bracket
arrives, it must match the bracket on **top** of the stack — if it doesn't, or
the stack is empty, the string is invalid. At the end, the stack must be empty
(every open bracket got closed).

**Walk through `"([)]"`:**
```
'(' → push → stack: ['(']
'[' → push → stack: ['(', '[']
')' → closing — top of stack is '[' but ')' needs '(' → mismatch → false ✓
```

**Walk through `"([])"`:**
```
'(' → push → stack: ['(']
'[' → push → stack: ['(', '[']
']' → closing — top is '[' → matches → pop → stack: ['(']
')' → closing — top is '(' → matches → pop → stack: []
End → stack empty → true ✓
```

---

## Kotlin Solution

### Approach 1 — Stack with HashMap lookup (optimal)

```kotlin
fun isValid(s: String): Boolean {
    val stack = ArrayDeque<Char>()
    val pairs = mapOf(')' to '(', ']' to '[', '}' to '{')

    for (c in s) {
        if (c in pairs) {
            // Closing bracket — must match the top of the stack
            if (stack.isEmpty() || stack.removeLast() != pairs[c]) return false
        } else {
            // Opening bracket — push it
            stack.addLast(c)
        }
    }

    return stack.isEmpty()
}
```

### Approach 2 — Stack with explicit when (no map, slightly faster)

```kotlin
fun isValid(s: String): Boolean {
    val stack = ArrayDeque<Char>()

    for (c in s) {
        when (c) {
            '(' , '[', '{' -> stack.addLast(c)
            ')' -> if (stack.isEmpty() || stack.removeLast() != '(') return false
            ']' -> if (stack.isEmpty() || stack.removeLast() != '[') return false
            '}' -> if (stack.isEmpty() || stack.removeLast() != '{') return false
        }
    }

    return stack.isEmpty()
}
```

Avoids the HashMap allocation entirely — marginal speed gain, more verbose.

---

## Why `ArrayDeque` Instead of a `Stack` Class

Kotlin/Java has a legacy `Stack` class, but it's synchronized (thread-safety
overhead you don't need) and considered outdated.

```kotlin
val stack = ArrayDeque<Char>()
stack.addLast(c)        // push
stack.removeLast()      // pop
stack.isEmpty()          // check empty
// ArrayDeque is the idiomatic, unsynchronized choice for stack behavior in Kotlin
```

**The empty-stack short-circuit** is the detail most people miss:
```kotlin
if (stack.isEmpty() || stack.removeLast() != pairs[c]) return false
// Short-circuit && / || — removeLast() is never called on an empty stack
// because isEmpty() check comes first
```

**Why check `stack.isEmpty()` at the end** — a string like `"((("` pushes
three opens but never closes any, leaving a non-empty stack:
```kotlin
return stack.isEmpty()
// "(()" → after processing: stack = ['('] → not empty → false (correctly invalid)
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| HashMap lookup (Approach 1) | Cleaner code, easy to extend to more bracket types |
| Explicit `when` (Approach 2) | Marginal performance, no allocation, only 3 bracket types |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — single pass through the string |
| **Space** | O(n) — worst case, all characters are opening brackets |

---

## Key Takeaway

> Stacks naturally enforce "last opened, first closed." Push opens, and on
> every close, check it matches the top of the stack — pop if it does,
> return false if it doesn't (or the stack is empty). A non-empty stack at
> the end means unclosed brackets remain. This pattern — push on open,
> validate-and-pop on close — is the foundation for every other Stack
> problem in this series.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/valid-parentheses/)

---

{% include kotlin-dsa-nav.html %}
