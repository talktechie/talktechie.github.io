---
title: "1-D DP: Word Break — Kotlin Solution"
date: 2026-06-26
categories: [Kotlin DSA, 1-D Dynamic Programming]
tags: [word-break, dynamic-programming, string, hashset, medium, leetcode, leetcode-139]
leetcode_number: 139
leetcode_url: https://leetcode.com/problems/word-break/
difficulty: Medium
permalink: /posts/kotlin-dsa-dp-word-break/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [139 — Word Break](https://leetcode.com/problems/word-break/) |
| **Difficulty** | Medium |
| **Topic** | Dynamic Programming, String, HashSet |

> Given a string `s` and a dictionary `wordDict`, return `true` if `s`
> can be segmented into a space-separated sequence of one or more
> dictionary words (words may be reused).

**Example:**
```
Input:  s = "leetcode", wordDict = ["leet","code"]
Output: true
Explanation: "leet" + "code"

Input:  s = "catsandog", wordDict = ["cats","dog","sand","and","cat"]
Output: false
Explanation: no valid combination covers the entire string
```

**Constraints:**
- `1 <= s.length <= 300`
- `1 <= wordDict.length <= 1000`
- `1 <= wordDict[i].length <= 20`
- `s` and `wordDict[i]` consist of lowercase English letters

---

## Approach

**Key insight:** Define `dp[i]` as "can the prefix `s[0..i-1]` be fully
segmented into dictionary words?" For each position `i`, try every
possible **last word boundary** `j < i` — if `dp[j]` is already known
true, AND the substring `s[j..i-1]` is itself a valid dictionary word,
then `dp[i]` is true. This is the same "try every possible last
decision point" shape as Coin Change, applied to substring boundaries
instead of coin denominations.

**Walk through `s = "leetcode", wordDict = ["leet","code"]`:**
```
dp[0] = true   (empty prefix is trivially "segmented")

dp[1..3] = false (no single-letter/short prefixes match any dictionary word)
dp[4] ("leet"): try j=0 → dp[0]=true AND s[0..3]="leet" is in dict → dp[4]=true

dp[5..7] = false (no valid word boundary found yet for these lengths)
dp[8] ("leetcode"): try j=4 → dp[4]=true AND s[4..7]="code" is in dict → dp[8]=true

dp[8] (the full string length) = true → return true ✓
```

---

## Kotlin Solution

### Approach 1 — Bottom-up DP, try every split point (optimal, most intuitive)

```kotlin
fun wordBreak(s: String, wordDict: List<String>): Boolean {
    val wordSet = wordDict.toHashSet()
    val n = s.length
    val dp = BooleanArray(n + 1)
    dp[0] = true   // empty prefix is trivially segmentable

    for (i in 1..n) {
        for (j in 0 until i) {
            if (dp[j] && s.substring(j, i) in wordSet) {
                dp[i] = true
                break   // found one valid way — no need to check other j values
            }
        }
    }

    return dp[n]
}
```

### Approach 2 — Top-down memoization

```kotlin
fun wordBreak(s: String, wordDict: List<String>): Boolean {
    val wordSet = wordDict.toHashSet()
    val memo = HashMap<Int, Boolean>()

    fun dp(start: Int): Boolean {
        if (start == s.length) return true

        return memo.getOrPut(start) {
            for (end in start + 1..s.length) {
                if (s.substring(start, end) in wordSet && dp(end)) {
                    return@getOrPut true
                }
            }
            false
        }
    }

    return dp(0)
}
```

### Approach 3 — BFS over string positions (graph framing)

```kotlin
fun wordBreak(s: String, wordDict: List<String>): Boolean {
    val wordSet = wordDict.toHashSet()
    val n = s.length
    val visited = BooleanArray(n + 1)
    val queue: ArrayDeque<Int> = ArrayDeque()
    queue.addLast(0)
    visited[0] = true

    while (queue.isNotEmpty()) {
        val start = queue.removeFirst()

        if (start == n) return true

        for (end in start + 1..n) {
            if (!visited[end] && s.substring(start, end) in wordSet) {
                visited[end] = true
                queue.addLast(end)
            }
        }
    }

    return false
}
```

Treats each string position as a graph node, with an edge from `start`
to `end` whenever `s[start..end-1]` is a dictionary word — finding if
position `n` is reachable from position `0` answers the question, same
graph-theoretic reframing technique seen in Coin Change's BFS approach.

---

## Why Checking `dp[j] && substring in wordSet` (Both Conditions) Is Essential

```kotlin
if (dp[j] && s.substring(j, i) in wordSet) {
    dp[i] = true
    break
}
```

Both halves of this condition are independently necessary:
`s.substring(j, i)` being a valid dictionary word alone isn't enough —
the **prefix before it** (`s[0..j-1]`) must *also* already be fully
segmentable (`dp[j]` true), otherwise we'd incorrectly validate a
substring that has no way to actually connect back to the start of the
original string.

**Why `break` after finding one valid split (Approach 1):** since `dp[i]`
is a boolean (we only need to know *if* a valid segmentation exists, not
enumerate all of them), the moment we find **any** valid `j`, we can stop
checking — further `j` values can't make `dp[i]` "more true."

**Why this is structurally identical to Coin Change's "try every last
decision" pattern:** Coin Change asks "what's the best way to reach
amount `a`, trying every coin as the last one used"; Word Break asks "can
we reach position `i`, trying every dictionary word as the last one
matched." Same recurrence shape — iterate over every possible
*immediately preceding* state, and combine with whatever's already known
about reaching that state.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Bottom-up DP (Approach 1) | Always — clean and iterative |
| Top-down memoization (Approach 2) | Prefer recursive framing |
| BFS (Approach 3) | Want to see the graph-theoretic connection explicitly |

---

## Complexity

| | |
|---|---|
| **Time** | O(n² ) — n positions, up to n substring checks each, each substring check itself bounded by word length |
| **Space** | O(n) — the DP array, plus the word dictionary's HashSet |

---

## Key Takeaway

> Word Break is "Coin Change for strings" — instead of trying every coin
> denomination as the most recent addition, try every dictionary word as
> the most recent matched segment, building up from "can we segment the
> empty prefix" (trivially yes) to "can we segment the whole string."
> Recognizing problems that share this "try every valid last step,
> building on already-solved smaller prefixes" shape — regardless of
> whether the underlying domain is numbers (coins) or text (words) — is
> one of the most transferable pattern-recognition skills in this entire
> DP phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/word-break/)

---

{% include kotlin-dsa-nav.html %}
