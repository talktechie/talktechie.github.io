---
title: "1-D DP: Decode Ways — Kotlin Solution"
date: 2026-06-26
categories: [Kotlin DSA, 1-D Dynamic Programming]
tags: [decode-ways, dynamic-programming, string, medium, leetcode, leetcode-91]
leetcode_number: 91
leetcode_url: https://leetcode.com/problems/decode-ways/
difficulty: Medium
permalink: /posts/kotlin-dsa-dp-decode-ways/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [91 — Decode Ways](https://leetcode.com/problems/decode-ways/) |
| **Difficulty** | Medium |
| **Topic** | Dynamic Programming, String |

> A message of digits can be decoded using `'A'`=1, `'B'`=2, ...,
> `'Z'`=26. Given a string `s` of digits, return the number of ways to
> **decode** it.

**Example:**
```
Input:  s = "12"
Output: 2
Explanation: "AB" (1,2) or "L" (12)

Input:  s = "226"
Output: 3
Explanation: "BZ" (2,26), "VF" (22,6), "BBF" (2,2,6)

Input:  s = "06"
Output: 0
Explanation: "06" is not a valid encoding — no leading zeros allowed
             for two-digit codes, and "0" alone isn't a valid letter
```

**Constraints:**
- `1 <= s.length <= 100`
- `s` consists of digits, may contain leading zero

---

## Approach

This is Climbing Stairs' recurrence wearing yet another costume — but
with added **validity conditions** that determine whether each
transition is even allowed.

**Key insight:** At position `i`, the number of ways to decode the
prefix `s[0..i]` depends on: can the **last single digit** (`s[i]`)
stand alone as a valid letter (1-9, never 0)? And can the **last two
digits** (`s[i-1..i]`) form a valid two-digit letter (10-26)? Each `yes`
contributes the ways count from further back, summed together — exactly
like Climbing Stairs' `dp[i] = dp[i-1] + dp[i-2]`, except each term is
only included **if** the corresponding decoding choice is valid.

**Walk through `s = "226"`:**
```
dp[0] (empty prefix) = 1     (one way to decode nothing — the base case)
dp[1] (prefix "2") = 1       ('2' is valid alone → inherits dp[0])

dp[2] (prefix "22"):
  single digit '2' (s[1]) valid alone → += dp[1] = 1
  two digits "22" (s[0..1]) valid (10-26) → += dp[0] = 1
  dp[2] = 1 + 1 = 2

dp[3] (prefix "226"):
  single digit '6' (s[2]) valid alone → += dp[2] = 2
  two digits "26" (s[1..2]) valid (10-26) → += dp[1] = 1
  dp[3] = 2 + 1 = 3

Answer: 3 ✓
```

---

## Kotlin Solution

### Approach 1 — Bottom-up DP, O(1) space, two rolling variables (optimal)

```kotlin
fun numDecodings(s: String): Int {
    if (s.isEmpty() || s[0] == '0') return 0

    var prev2 = 1   // dp[i-2] equivalent — represents the empty-prefix base case
    var prev1 = 1   // dp[i-1] equivalent — represents the first character processed

    for (i in 1 until s.length) {
        var current = 0

        if (s[i] != '0') current += prev1                       // single digit s[i] is valid alone
        val twoDigit = s.substring(i - 1, i + 1).toInt()
        if (twoDigit in 10..26) current += prev2                 // two digits s[i-1..i] valid

        if (current == 0) return 0   // no valid decoding continues from here — dead end

        prev2 = prev1
        prev1 = current
    }

    return prev1
}
```

### Approach 2 — Top-down memoization

```kotlin
fun numDecodings(s: String): Int {
    val memo = HashMap<Int, Int>()

    fun dp(i: Int): Int {
        if (i == s.length) return 1   // successfully decoded the entire string
        if (s[i] == '0') return 0      // a leading zero can never be decoded

        return memo.getOrPut(i) {
            var ways = dp(i + 1)   // take just this one digit

            if (i + 1 < s.length) {
                val twoDigit = s.substring(i, i + 2).toInt()
                if (twoDigit in 10..26) ways += dp(i + 2)   // take two digits together
            }

            ways
        }
    }

    return dp(0)
}
```

---

## Why the Early-Exit Check (`if (current == 0) return 0`) Matters

```kotlin
if (current == 0) return 0   // no valid decoding continues from here — dead end
```

If, at some position, **neither** the single-digit nor two-digit
interpretation is valid (e.g., the current digit is `'0'` and the
two-digit combination also doesn't fall in `10-26`), then there's no way
to extend any valid decoding past this point — the whole string is
undecodable, and we can return `0` immediately rather than continuing to
compute meaningless values.

**Why checking `s[0] == '0'` upfront is necessary:** a leading `'0'` can
never be the start of any valid decoding, since no letter maps to `0`
and a two-digit code can't start with `0` either (`"0X"` isn't a valid
range 10-26) — without this check, the rest of the algorithm could
silently produce an incorrect non-zero answer for an actually-invalid
input.

**Why the two-digit check uses `in 10..26`, not `<= 26`:** two-digit
codes must be at least `10` (since `01`-`09` would just be redundant
with — and conflict with — their single-digit `'0'`-leading-zero-invalid
counterparts) and at most `26` (since the alphabet only has 26 letters).

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Rolling variables (Approach 1) | Always — O(1) space, fastest |
| Top-down memoization (Approach 2) | Prefer recursive framing, fine with O(n) space |

---

## Complexity

| | |
|---|---|
| **Time** | O(n) |
| **Space** | O(1) rolling variables / O(n) memoization |

---

## Key Takeaway

> Decode Ways extends the "sum of the previous two states" recurrence
> from Climbing Stairs by adding **validity gates** on each transition —
> you only add `dp[i-1]` if the single-digit interpretation is legal, and
> only add `dp[i-2]` if the two-digit interpretation is legal. This
> pattern — a Fibonacci-shaped recurrence where each term is
> conditionally included based on a validity check — is common whenever
> a DP problem involves "ways to partition/decode a sequence" under
> constraints on what each partition/token is allowed to look like.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/decode-ways/)

---

{% include kotlin-dsa-nav.html %}
