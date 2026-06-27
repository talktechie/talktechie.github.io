---
title: "Backtracking: Generate Parentheses — Kotlin Solution"
date: 2026-06-24
categories: [Kotlin DSA, Backtracking]
tags: [generate-parentheses, backtracking, string, medium, leetcode, leetcode-22]
leetcode_number: 22
leetcode_url: https://leetcode.com/problems/generate-parentheses/
difficulty: Medium
permalink: /posts/kotlin-dsa-backtracking-generate-parentheses/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [22 — Generate Parentheses](https://leetcode.com/problems/generate-parentheses/) |
| **Difficulty** | Medium |
| **Topic** | Backtracking, String |

> Given `n` pairs of parentheses, generate all combinations of
> **well-formed** (valid) parentheses strings.

**Example:**
```
Input:  n = 3
Output: ["((()))","(()())","(())()","()(())","()()()"]

Input:  n = 1
Output: ["()"]
```

**Constraints:**
- `1 <= n <= 8`

---

## Approach

This connects directly back to Valid Parentheses (LC 20, from the Stack
phase) — we already know what makes a string *valid*. Now we need to
**generate** every valid string, using backtracking with constraints
that prune invalid paths before they're ever fully built.

**Key insight — track open and close counts as constraints, not just
characters used:** at each step, we can add `'('` as long as we haven't
used all `n` of them yet. We can only add `')'` if there are currently
**more open parens than close parens** in the string so far (otherwise
the string would become invalid — a closing paren with no matching
open). This is a direct application of the rule from Valid Parentheses,
used here as a generation constraint rather than a validation check.

**Walk through `n = 2`:**
```
backtrack(current="", open=0, close=0):
  open(0) < n(2) → try adding '(' → current="(", open=1
    open(1) < n(2) → try adding '(' → current="((", open=2
      open(2) == n(2) → can't add more '('
      close(0) < open(2) → try adding ')' → current="((,)", close=1
        close(1) < open(2) → try adding ')' → current="(())", close=2
          length == 2n(4) → record "(())" ✓
    close(0) < open(1) → try adding ')' → current="()", close=1
      open(1) < n(2) → try adding '(' → current="()(", open=2
        close(1) < open(2) → try adding ')' → current="()()" → record ✓

Result: ["(())", "()()"] ✓
```

---

## Kotlin Solution

### Approach 1 — Backtracking with open/close counters as pruning constraints (optimal)

```kotlin
fun generateParenthesis(n: Int): List<String> {
    val result = mutableListOf<String>()
    val current = StringBuilder()

    fun backtrack(open: Int, close: Int) {
        if (current.length == 2 * n) {
            result.add(current.toString())
            return
        }

        if (open < n) {
            current.append('(')
            backtrack(open + 1, close)
            current.deleteCharAt(current.lastIndex)   // undo
        }

        if (close < open) {
            current.append(')')
            backtrack(open, close + 1)
            current.deleteCharAt(current.lastIndex)   // undo
        }
    }

    backtrack(0, 0)
    return result
}
```

### Approach 2 — Generate all 2^(2n) strings, filter with Valid Parentheses check

```kotlin
fun generateParenthesis(n: Int): List<String> {
    val result = mutableListOf<String>()
    val current = CharArray(2 * n)

    fun isValid(s: CharArray): Boolean {
        var balance = 0
        for (c in s) {
            if (c == '(') balance++ else balance--
            if (balance < 0) return false
        }
        return balance == 0
    }

    fun backtrack(index: Int) {
        if (index == 2 * n) {
            if (isValid(current)) result.add(String(current))
            return
        }

        current[index] = '('
        backtrack(index + 1)

        current[index] = ')'
        backtrack(index + 1)
    }

    backtrack(0)
    return result
}
```

Generates **every** possible arrangement of `n` opens and `n` closes
(`2^(2n)` total), then filters for validity at the end — correct, but
wastes enormous effort building and checking strings that could have been
pruned far earlier. Shown only to highlight what the constrained approach
avoids.

---

## Why Pruning During Generation Beats Generate-Then-Filter

This is one of the clearest illustrations in the whole Backtracking phase
of **why** early pruning matters so much. Approach 2 explores all
`2^(2n)` binary strings — for `n=8`, that's `2^16 = 65536` strings, most
of which are invalid and get discarded. Approach 1 only ever builds
strings that **remain valid at every single step**:

```kotlin
if (open < n) { ... }      // never add more '(' than allowed
if (close < open) { ... }  // never add ')' that would create an imbalance
```

These two conditions mean every recursive call maintains a **guaranteed
valid prefix** — by the time we reach `current.length == 2*n`, the result
is automatically valid, with no separate validation pass needed at all.

**Why `close < open` (not `close < n`)** is the precise constraint:
adding a closing paren is fine as long as there's a previously-unmatched
opening paren to pair it with — checking against the **current open
count**, not the total target `n`, is what ensures the string never
becomes invalid mid-construction.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Constrained backtracking (Approach 1) | Always — generates only valid strings, far more efficient |
| Generate-then-filter (Approach 2) | Never for submission; included purely to illustrate the contrast |

---

## Complexity

| | Approach 1 | Approach 2 |
|---|---|---|
| **Time** | O(4ⁿ / √n) — bounded by the nth Catalan number, the true count of valid combinations | O(2^(2n) × n) |
| **Space** | O(n) — recursion depth | O(2n) per string during generation |

---

## Key Takeaway

> The single biggest lesson from this problem: prune **during**
> generation, not after. Tracking simple counters (`open` and `close`)
> as explicit backtracking constraints means we only ever construct
> strings that are valid at every intermediate step — directly reusing
> the "balance never goes negative" insight from Valid Parentheses, but
> applying it proactively as a generation rule instead of reactively as
> a validation check. This same "build a count-based invariant into the
> recursion's branching conditions" idea is the most transferable
> takeaway from this problem.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/generate-parentheses/)

---

{% include kotlin-dsa-nav.html %}
