---
title: "Stack: Evaluate Reverse Polish Notation — Kotlin Solution"
date: 2026-06-17
categories: [Kotlin DSA, Stack]
tags: [evaluate-reverse-polish-notation, stack, array, math, medium, leetcode, leetcode-150]
leetcode_number: 150
leetcode_url: https://leetcode.com/problems/evaluate-reverse-polish-notation/
difficulty: Medium
permalink: /posts/kotlin-dsa-stack-evaluate-reverse-polish-notation/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [150 — Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/) |
| **Difficulty** | Medium |
| **Topic** | Stack, Array, Math |

> Evaluate the value of an arithmetic expression given in **Reverse Polish
> Notation** (postfix notation).
>
> Valid operators are `+`, `-`, `*`, `/`. Each operand may be an integer
> or another expression. Division between two integers should truncate
> toward zero.

**Example:**
```
Input:  tokens = ["2","1","+","3","*"]
Output: 9
Explanation: ((2 + 1) * 3) = 9

Input:  tokens = ["4","13","5","/","+"]
Output: 6
Explanation: (4 + (13 / 5)) = 6

Input:  tokens = ["10","6","9","3","+","-11","*","/","*","17","+","5","+"]
Output: 22
```

**Constraints:**
- `1 <= tokens.length <= 10⁴`
- Each token is an operator (`+`, `-`, `*`, `/`) or an integer in `[-200, 200]`
- The expression is always valid

---

## Approach

In postfix notation, an operator always comes **after** its two operands.
This is exactly what a stack is built for — push operands, and when an
operator arrives, pop the two most recent operands, apply it, push the result.

**Key insight:** No need to parse precedence or parentheses at all — the
order tokens appear in already encodes the evaluation order. Stack handles
the rest.

**Walk through `["2","1","+","3","*"]`:**
```
"2" → number → push  → stack: [2]
"1" → number → push  → stack: [2, 1]
"+" → operator → pop 1, pop 2 → 2 + 1 = 3 → push → stack: [3]
"3" → number → push  → stack: [3, 3]
"*" → operator → pop 3, pop 3 → 3 * 3 = 9 → push → stack: [9]

Result: stack.last() = 9 ✓
```

**Order matters for subtraction/division** — always `firstPopped op secondPopped`
... actually the reverse: the **second** popped value is the left operand:

```
stack: [4, 13, 5]   token = "/"
pop → b = 5   (top)
pop → a = 13  (next)
result = a / b = 13 / 5 = 2   ← order matters! b is divisor, not a
```

---

## Kotlin Solution

### Approach 1 — Stack with when expression (optimal)

```kotlin
fun evalRPN(tokens: Array<String>): Int {
    val stack = ArrayDeque<Int>()

    for (token in tokens) {
        when (token) {
            "+" -> {
                val b = stack.removeLast()
                val a = stack.removeLast()
                stack.addLast(a + b)
            }
            "-" -> {
                val b = stack.removeLast()
                val a = stack.removeLast()
                stack.addLast(a - b)
            }
            "*" -> {
                val b = stack.removeLast()
                val a = stack.removeLast()
                stack.addLast(a * b)
            }
            "/" -> {
                val b = stack.removeLast()
                val a = stack.removeLast()
                stack.addLast(a / b)  // Kotlin Int division truncates toward zero
            }
            else -> stack.addLast(token.toInt())
        }
    }

    return stack.last()
}
```

### Approach 2 — Lambda map for operators (less repetition)

```kotlin
fun evalRPN(tokens: Array<String>): Int {
    val stack = ArrayDeque<Int>()
    val operators = mapOf<String, (Int, Int) -> Int>(
        "+" to { a, b -> a + b },
        "-" to { a, b -> a - b },
        "*" to { a, b -> a * b },
        "/" to { a, b -> a / b }
    )

    for (token in tokens) {
        val op = operators[token]
        if (op != null) {
            val b = stack.removeLast()
            val a = stack.removeLast()
            stack.addLast(op(a, b))
        } else {
            stack.addLast(token.toInt())
        }
    }

    return stack.last()
}
```

Reduces duplicated pop logic — each operator is just a 2-argument lambda.

---

## Why Pop Order Matters

The most common bug in this problem is reversing the operand order for `-` and `/`:

```kotlin
val b = stack.removeLast()   // most recently pushed → RIGHT operand
val a = stack.removeLast()   // second most recent → LEFT operand
stack.addLast(a - b)         // a is left, b is right: a - b, NOT b - a
```

For `+` and `*` order doesn't matter (commutative), but for `-` and `/` it's
critical — getting it backward silently produces wrong answers that are easy
to miss in testing.

**Kotlin's `/` on Int already truncates toward zero**, matching the problem's
requirement with zero extra code:
```kotlin
stack.addLast(a / b)
// 13 / 5 = 2     (matches expected)
// -7 / 2 = -3    (Kotlin truncates toward zero, same as Java/C++)
```

**`when (token)` matching on String** — clean dispatch without a chain of `if/else`:
```kotlin
when (token) {
    "+" -> { ... }
    else -> stack.addLast(token.toInt())
}
// Falls through to the numeric case for any non-operator token
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| `when` expression (Approach 1) | Straightforward, no extra allocations |
| Lambda map (Approach 2) | Want to reduce repeated pop/push boilerplate |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) — one pass through tokens |
| **Space** | O(n) — worst case, all tokens are operands before any operator |

---

## Key Takeaway

> Postfix notation removes the need for precedence rules or parentheses —
> the token order already encodes evaluation order. Push operands, and on
> every operator, pop two, apply, push the result. The one gotcha: for
> non-commutative operators, the **second** popped value is the left operand.
> `a = pop(); b = pop()` is wrong — it should be `b = pop(); a = pop()`.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/evaluate-reverse-polish-notation/)

---

{% include kotlin-dsa-nav.html %}
