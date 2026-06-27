---
title: "Backtracking: Letter Combinations of a Phone Number — Kotlin Solution"
date: 2026-06-24
categories: [Kotlin DSA, Backtracking]
tags: [letter-combinations-phone-number, backtracking, string, hashmap, medium, leetcode, leetcode-17]
leetcode_number: 17
leetcode_url: https://leetcode.com/problems/letter-combinations-of-a-phone-number/
difficulty: Medium
permalink: /posts/kotlin-dsa-backtracking-letter-combinations-of-a-phone-number/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [17 — Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/) |
| **Difficulty** | Medium |
| **Topic** | Backtracking, String, HashMap |

> Given a string of digits `2-9`, return all possible letter combinations
> the digits could represent, based on a standard telephone keypad
> mapping (2="abc", 3="def", ..., 9="wxyz").

**Example:**
```
Input:  digits = "23"
Output: ["ad","ae","af","bd","be","bf","cd","ce","cf"]
Explanation: '2' maps to "abc", '3' maps to "def" — every combination
             of one letter from each

Input:  digits = ""
Output: []

Input:  digits = "2"
Output: ["a","b","c"]
```

**Constraints:**
- `0 <= digits.length <= 4`
- `digits[i]` is a digit in the range `['2', '9']`

---

## Approach

Each digit independently contributes a small, fixed set of letter
choices, and we need every possible combination across all digit
positions — directly analogous to Permutations' "choose from available
options at each step," except here the set of choices comes from a
**fixed mapping** rather than a shrinking pool of unused elements.

**Key insight:** At each position (each digit in the input), try every
letter that digit maps to. Recurse to the next digit position; once
we've made a choice for every digit, the accumulated string is a
complete, valid combination.

**Walk through `digits = "23"`:**
```
digitMap: {'2':"abc", '3':"def"}

backtrack(index=0, current=""):
  digit='2' → try 'a': current="a"
    backtrack(index=1, current="a"):
      digit='3' → try 'd': current="ad" → index==length → record "ad" ✓
      try 'e': current="ae" → record "ae" ✓
      try 'f': current="af" → record "af" ✓
  try 'b': current="b"
    ... produces "bd","be","bf" ✓
  try 'c': current="c"
    ... produces "cd","ce","cf" ✓

Result: ["ad","ae","af","bd","be","bf","cd","ce","cf"] ✓
```

---

## Kotlin Solution

### Approach 1 — Backtracking with a digit-to-letters map (optimal, most instructive)

```kotlin
fun letterCombinations(digits: String): List<String> {
    if (digits.isEmpty()) return emptyList()

    val digitMap = mapOf(
        '2' to "abc", '3' to "def", '4' to "ghi", '5' to "jkl",
        '6' to "mno", '7' to "pqrs", '8' to "tuv", '9' to "wxyz"
    )

    val result = mutableListOf<String>()
    val current = StringBuilder()

    fun backtrack(index: Int) {
        if (index == digits.length) {
            result.add(current.toString())
            return
        }

        val letters = digitMap[digits[index]]!!
        for (letter in letters) {
            current.append(letter)
            backtrack(index + 1)
            current.deleteCharAt(current.lastIndex)   // undo
        }
    }

    backtrack(0)
    return result
}
```

### Approach 2 — Iterative, build up the result list digit by digit

```kotlin
fun letterCombinations(digits: String): List<String> {
    if (digits.isEmpty()) return emptyList()

    val digitMap = mapOf(
        '2' to "abc", '3' to "def", '4' to "ghi", '5' to "jkl",
        '6' to "mno", '7' to "pqrs", '8' to "tuv", '9' to "wxyz"
    )

    var result = listOf("")

    for (digit in digits) {
        val letters = digitMap[digit]!!
        result = result.flatMap { prefix -> letters.map { prefix + it } }
    }

    return result
}
```

For each digit, expand every existing partial combination by appending
each possible letter for that digit — no recursion at all, just
iterative list expansion.

---

## Why the Empty-Input Check Comes First

```kotlin
if (digits.isEmpty()) return emptyList()
```

This is a specific quirk of this problem's expected behavior: an empty
input string should produce an **empty list** of combinations (`[]`), not
a list containing a single empty string (`[""]`). Without this explicit
check, the backtracking would immediately hit its base case
(`index == digits.length`, which is `0 == 0`) and incorrectly record `""`
as a "combination."

**Why the iterative approach (Approach 2) is a clean alternative for this
specific problem:** because every digit's letter choices are completely
independent of all other digits (unlike, say, Permutations, where
choices for later positions depend on what's already been used), there's
no need for explicit backtracking/undo logic — `flatMap` naturally
expresses "for every existing partial result, branch into every new
possibility":
```kotlin
result = result.flatMap { prefix -> letters.map { prefix + it } }
// For digit '3' with letters "def" and existing prefixes ["a","b","c"]:
// → "a"+"d","a"+"e","a"+"f", "b"+"d","b"+"e","b"+"f", "c"+"d","c"+"e","c"+"f"
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Backtracking (Approach 1) | Want the general template consistent with the rest of this phase |
| Iterative expansion (Approach 2) | This specific problem's structure (independent choices, no shared state) makes it a clean fit, slightly less code |

---

## Complexity

| | |
|---|---|
| **Time** | O(4ⁿ × n) — up to 4 letters per digit (e.g., '7' and '9' have 4 letters each), n digits total |
| **Space** | O(n) — recursion depth / intermediate string length |

---

## Key Takeaway

> When each position's choices come from an independent, fixed set rather
> than a shrinking shared pool, both backtracking and simple iterative
> expansion work equally well — there's no need to track "used" state
> since nothing is ever consumed or shared across positions. This problem
> is a good checkpoint for recognizing when backtracking's full
> machinery (explicit undo, pruning) is actually necessary versus when a
> simpler iterative `flatMap`-style expansion gets the same result with
> less ceremony.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)

---

{% include kotlin-dsa-nav.html %}
