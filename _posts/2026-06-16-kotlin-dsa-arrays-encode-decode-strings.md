---
title: "Arrays: Encode and Decode Strings — Kotlin Solution"
date: 2026-06-16
categories: [Kotlin DSA, Arrays]
tags: [encode-decode-strings, string, design, length-prefix, medium, leetcode, leetcode-271]
leetcode_number: 271
leetcode_url: https://leetcode.com/problems/encode-and-decode-strings/
difficulty: Medium
permalink: /posts/kotlin-dsa-arrays-encode-decode-strings/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [271 — Encode and Decode Strings](https://leetcode.com/problems/encode-and-decode-strings/) |
| **Difficulty** | Medium |
| **Topic** | Arrays, String, Design |

> Design an algorithm to encode a list of strings to a single string.
> The encoded string is then sent over the network and decoded back to
> the original list of strings.
>
> Implement `encode` and `decode` — no separator character is safe because
> any character could appear in the input strings.

**Example:**
```
Input:  ["neet","code","love","you"]
Output: ["neet","code","love","you"]

Input:  ["we","say",":","yes"]
Output: ["we","say",":","yes"]
```

**Constraints:**
- `0 <= strs.length < 100`
- `0 <= strs[i].length < 200`
- `strs[i]` contains any UTF-8 characters (including `#`, `|`, delimiters)

---

## Approach

You cannot use a simple delimiter like `","` or `"|"` — those characters can appear
inside the strings themselves, making decoding ambiguous.

**Key insight — Length-prefix encoding:** Prepend each string with its length
followed by a separator character (`#`). On the decode side, read the number
before `#` to know exactly how many characters to consume. No ambiguity possible.

**Walk through `["neet","code","love","you"]`:**
```
Encode:
  "neet" → "4#neet"
  "code" → "4#code"
  "love" → "4#love"
  "you"  → "3#you"
  result → "4#neet4#code4#love3#you"

Decode "4#neet4#code4#love3#you":
  i=0 → j finds '#' at 1 → length=4 → read s[2..5]="neet" → i=6
  i=6 → j finds '#' at 7 → length=4 → read s[8..11]="code" → i=12
  i=12 → j finds '#' at 13 → length=4 → read s[14..17]="love" → i=18
  i=18 → j finds '#' at 19 → length=3 → read s[20..22]="you"  → i=23
  result → ["neet","code","love","you"] ✓
```

Why `#` as separator? It marks where the length number ends — the number itself
tells us where the string ends, so `#` never appears in an ambiguous position.

---

## Kotlin Solution

### Approach 1 — Length-prefix with `#` delimiter (optimal)

```kotlin
fun encode(strs: List<String>): String {
    return buildString {
        for (s in strs) {
            append(s.length)
            append('#')
            append(s)
        }
    }
}

fun decode(s: String): List<String> {
    val result = mutableListOf<String>()
    var i = 0

    while (i < s.length) {
        var j = i
        while (s[j] != '#') j++          // scan to find '#'
        val length = s.substring(i, j).toInt()
        result.add(s.substring(j + 1, j + 1 + length))
        i = j + 1 + length               // advance past this segment
    }

    return result
}
```

### Approach 2 — Fixed 4-byte length header

```kotlin
fun encode(strs: List<String>): String {
    return buildString {
        for (s in strs) {
            append(s.length.toString().padStart(4, '0'))
            append(s)
        }
    }
}

fun decode(s: String): List<String> {
    val result = mutableListOf<String>()
    var i = 0

    while (i < s.length) {
        val length = s.substring(i, i + 4).toInt()
        result.add(s.substring(i + 4, i + 4 + length))
        i += 4 + length
    }

    return result
}
```

No `#` needed — the header is always exactly 4 chars. Assumes string length ≤ 9999.

---

## Why a Simple Delimiter Doesn't Work

The tempting approach — join with `","` and split on decode — breaks immediately:

```kotlin
// BROKEN — what if a string contains ","?
encode(["a,b", "c"]) → "a,b,c"
decode("a,b,c")      → ["a", "b", "c"]  // wrong! expected ["a,b", "c"]
```

Any character you pick as a delimiter could appear in the input. The only safe
approach is to encode the **length** of each string explicitly, so the decoder
never has to guess where one string ends.

**`buildString { }`** — idiomatic Kotlin string building:
```kotlin
return buildString {
    for (s in strs) {
        append(s.length)   // auto-converts Int to String
        append('#')
        append(s)
    }
}
// vs Java: StringBuilder sb = new StringBuilder();
//          for (String s : strs) sb.append(s.length()).append('#').append(s);
//          return sb.toString();
```

**Two-pointer decode** — `i` tracks position, inner `j` scans for `#`:
```kotlin
var j = i
while (s[j] != '#') j++              // j stops at '#'
val length = s.substring(i, j).toInt()
result.add(s.substring(j + 1, j + 1 + length))
i = j + 1 + length                   // jump exactly past this word
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Length-prefix + `#` (Approach 1) | General case — handles any string length, clean to implement |
| Fixed 4-byte header (Approach 2) | String length bounded (≤ 9999), slightly faster decode (no scan for `#`) |

---

## Follow-up — What about empty strings or empty list?

Both approaches handle these naturally:
- Empty string `""` encodes as `"0#"` — length is 0, nothing follows
- Empty list produces `""` on encode and `[]` on decode

---

## Complexity

| | Encode | Decode |
|---|---|---|
| **Time** | O(n) | O(n) |
| **Space** | O(n) | O(n) |

Where `n` = total number of characters across all strings.

---

## Key Takeaway

> When no delimiter character is safe, encode the length. Prepend each string
> with `length#` — the decoder reads the number, skips `#`, then consumes
> exactly that many characters. Zero ambiguity, any character set supported.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/encode-and-decode-strings/)

---

{% include kotlin-dsa-nav.html %}
