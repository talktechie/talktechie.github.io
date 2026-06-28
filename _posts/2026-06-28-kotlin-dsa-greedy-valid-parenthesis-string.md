---
title: "Greedy: Valid Parenthesis String — Kotlin Solution"
date: 2026-06-28
categories: [Kotlin DSA, Greedy]
tags: [valid-parenthesis-string, greedy, stack, dynamic-programming, string, medium, leetcode, leetcode-678]
leetcode_number: 678
leetcode_url: https://leetcode.com/problems/valid-parenthesis-string/
difficulty: Medium
permalink: /posts/kotlin-dsa-greedy-valid-parenthesis-string/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [678 — Valid Parenthesis String](https://leetcode.com/problems/valid-parenthesis-string/) |
| **Difficulty** | Medium |
| **Topic** | Greedy, Stack, Dynamic Programming, String |

> Given a string `s` containing only `'('`, `')'`, and `'*'` (a wildcard
> that can act as `'('`, `')'`, or an empty string), return `true` if `s`
> could be a valid parentheses string under **some** interpretation of
> every `'*'`.

**Example:**
```
Input:  s = "()"
Output: true

Input:  s = "(*)"
Output: true
Explanation: treat '*' as empty string → "()"

Input:  s = "(*))"
Output: true
Explanation: treat '*' as '(' → "(()) "... wait, that's 4 chars matching
             4 chars: "((" + "))" — actually treat '*' as empty →
             "())" is invalid; treat '*' as ')' → "()))" invalid;
             treat '*' as '(' → "(())" valid! ✓
```

**Constraints:**
- `1 <= s.length <= 100`
- `s[i]` is `'('`, `')'`, or `'*'`

---

## Approach

This closes out the Greedy phase with its most delicate reasoning yet —
because `'*'` is genuinely ambiguous, we can't commit to a single
interpretation upfront. Instead, we track a **range** of possible
"currently open paren counts," collapsing what looks like exponential
branching into a simple two-variable greedy scan.

**Key insight — track the MINIMUM and MAXIMUM possible number of
currently-open (unmatched) `'('`:** at every character, update both
bounds:
- `'('`: both min and max increase (definitely one more open paren).
- `')'`: both min and max decrease (definitely one fewer open paren) —
  but min can't go below 0 conceptually (clamp it, since we'd simply
  choose not to let this `')'` close anything if min would go negative).
- `'*'`: it could be `'('` (max increases), `')'` (min decreases), or
  empty (neither changes) — so **min decreases** (assuming the most
  "closing" interpretation) and **max increases** (assuming the most
  "opening" interpretation), capturing the full range of possibilities
  simultaneously.

If at any point `maxOpen < 0`, even the most generous interpretation
can't avoid a "too many closes" failure — return `false` immediately.
At the end, `s` is valid if `minOpen` can be exactly `0` — i.e., **0 is
within the achievable range** `[minOpen, maxOpen]`.

**Walk through `s = "(*))"`:**
```
minOpen=0, maxOpen=0

'(' : minOpen=1, maxOpen=1
'*' : minOpen=max(0,1-1)=0, maxOpen=1+1=2
')' : minOpen=max(0,0-1)=0, maxOpen=2-1=1
')' : minOpen=max(0,0-1)=0, maxOpen=1-1=0

maxOpen never went negative ✓
Final: minOpen=0, maxOpen=0 → 0 is within [0,0] → true ✓
```

---

## Kotlin Solution

### Approach 1 — Greedy, track min/max possible open-paren count (optimal)

```kotlin
fun checkValidString(s: String): Boolean {
    var minOpen = 0
    var maxOpen = 0

    for (c in s) {
        when (c) {
            '(' -> {
                minOpen++
                maxOpen++
            }
            ')' -> {
                minOpen--
                maxOpen--
            }
            else -> {   // '*'
                minOpen--   // most "closing" interpretation
                maxOpen++   // most "opening" interpretation
            }
        }

        if (maxOpen < 0) return false   // even the best case has too many closes
        minOpen = maxOf(minOpen, 0)      // can't have negative open count — clamp
    }

    return minOpen == 0
}
```

### Approach 2 — DP, track all achievable open-counts as a Set (clearer, less efficient)

```kotlin
fun checkValidString(s: String): Boolean {
    var possibleCounts = mutableSetOf(0)

    for (c in s) {
        val next = mutableSetOf<Int>()

        for (count in possibleCounts) {
            when (c) {
                '(' -> next.add(count + 1)
                ')' -> if (count > 0) next.add(count - 1)
                else -> {   // '*'
                    next.add(count + 1)
                    if (count > 0) next.add(count - 1)
                    next.add(count)
                }
            }
        }

        if (next.isEmpty()) return false
        possibleCounts = next
    }

    return 0 in possibleCounts
}
```

Tracks every individually achievable open-paren count as a `Set<Int>`
rather than compressing the range into just two bounds — conceptually
clearer (it's directly simulating all branches), but O(n²) worst case
since the set can grow up to O(n) in size, processed over n characters.

---

## Why Tracking a MIN/MAX Range (Not Every Possible Value) Is Sufficient

This is the crucial compression that makes the greedy approach O(n)
instead of exponential or even the DP approach's O(n²):

```kotlin
minOpen--   // most "closing" interpretation
maxOpen++   // most "opening" interpretation
```

The key insight is that the set of all achievable open-paren counts at
any point is always a **contiguous range** — there are no "gaps." This
is because every transition (`+1`, `-1`, or `0` for a `'*'`) shifts
possible values by at most 1 in either direction, and starting from a
single point (`0`), the reachable set after any number of such steps
remains an unbroken interval. Since it's always contiguous, we only ever
need to track its two endpoints — `minOpen` and `maxOpen` — rather than
every individual value within it.

**Why we clamp `minOpen` to 0 (`minOpen = maxOf(minOpen, 0)`):** a
negative "open paren count" isn't a real, achievable state — it would
mean more closing parens were used than opens, which is invalid by
definition. Clamping reflects that we'd simply choose **not** to let an
ambiguous `'*'` (or a forced `)`) push the open-count below what's
actually possible; the true minimum achievable value can never be less
than 0.

**Why checking `maxOpen < 0` triggers immediate failure:** if even the
most generous, opening-everywhere interpretation of every `'*'` still
results in more closes than opens, there is truly no valid
interpretation — this is an unrecoverable failure, not just a tight
squeeze.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Min/max range (Approach 1) | Always — O(n) time, O(1) space, exploits the contiguous-range property |
| Set of all counts (Approach 2) | Want to see the literal "track every branch" simulation before compressing it |

---

## Complexity

| | Min/Max Range | Set of Counts |
|---|---|---|
| **Time** | O(n) | O(n²) worst case |
| **Space** | O(1) | O(n) — the set of possible counts |

---

## Key Takeaway

> Valid Parenthesis String closes the Greedy phase by showing how
> tracking a **range** of possibilities — rather than every individual
> possibility — can collapse what looks like genuine ambiguity into an
> O(1)-space greedy scan. The key enabling fact is that the achievable
> set of states is always contiguous, so only its two endpoints matter.
> This "compress a set of branching possibilities into a min/max range"
> technique is a powerful generalization beyond greedy algorithms
> specifically — whenever a problem's reachable states form an interval
> rather than a scattered set, tracking just the boundaries is often
> sufficient.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/valid-parenthesis-string/)

---

{% include kotlin-dsa-nav.html %}
