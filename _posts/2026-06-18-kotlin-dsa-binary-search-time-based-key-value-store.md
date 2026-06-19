---
title: "Binary Search: Time Based Key-Value Store — Kotlin Solution"
date: 2026-06-18
categories: [Kotlin DSA, Binary Search]
tags: [time-based-key-value-store, binary-search, hashmap, design, medium, leetcode, leetcode-981]
leetcode_number: 981
leetcode_url: https://leetcode.com/problems/time-based-key-value-store/
difficulty: Medium
permalink: /posts/kotlin-dsa-binary-search-time-based-key-value-store/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [981 — Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store/) |
| **Difficulty** | Medium |
| **Topic** | Binary Search, HashMap, Design |

> Design a time-based key-value store that supports:
> - `set(key, value, timestamp)` — stores `key` with `value` at the given
>   `timestamp`.
> - `get(key, timestamp)` — returns the value associated with `key` at
>   the largest `timestamp_prev <= timestamp`. If no such timestamp
>   exists, return `""`.
>
> Timestamps for `set` calls on the same key are guaranteed to be
> strictly increasing.

**Example:**
```
Input:
["TimeMap","set","get","get","set","get","get"]
[[],["foo","bar",1],["foo",1],["foo",2],["foo","bar2",4],["foo",4],["foo",5]]

Output:
[null,null,"bar","bar",null,"bar2","bar2"]

Explanation:
TimeMap timeMap = new TimeMap();
timeMap.set("foo", "bar", 1);   // store key "foo" → value "bar" at timestamp 1
timeMap.get("foo", 1);          // "bar"
timeMap.get("foo", 2);          // "bar" — no entry at t=2, latest <= 2 is t=1
timeMap.set("foo", "bar2", 4);
timeMap.get("foo", 4);          // "bar2"
timeMap.get("foo", 5);          // "bar2" — latest <= 5 is t=4
```

**Constraints:**
- `1 <= key.length, value.length <= 100`
- `key`, `value` consist of lowercase English letters and digits
- `1 <= timestamp <= 10⁷`
- `set` calls are strictly increasing in timestamp per key
- At most `2 * 10⁵` calls to `set` and `get`

---

## Approach

Since `set` calls per key arrive in **strictly increasing timestamp order**,
each key's history is naturally a sorted list — no manual sorting needed.
This makes binary search the perfect tool for `get`.

**Key insight:** Store `HashMap<String, MutableList<Pair<Int, String>>>` —
one growing, already-sorted list per key. For `get(key, timestamp)`, binary
search that list for the **largest timestamp ≤ the query timestamp** (this
specific variant is sometimes called "find floor" or "upper bound minus one").

**Walk through `set("foo","bar",1)`, `set("foo","bar2",4)`, then `get("foo", 5)`:**
```
store["foo"] = [(1,"bar"), (4,"bar2")]

get("foo", 5):
  left=0, right=1
  mid=0 → timestamps[0]=1 <= 5 → candidate found, try going right → left=1
  left=1, right=1
  mid=1 → timestamps[1]=4 <= 5 → candidate found, try going right → left=2
  left=2 > right=1 → stop

  Best candidate found at index 1 → value = "bar2" ✓
```

---

## Kotlin Solution

### Approach 1 — HashMap of lists + binary search for floor (optimal)

```kotlin
class TimeMap {
    private val store = HashMap<String, MutableList<Pair<Int, String>>>()

    fun set(key: String, value: String, timestamp: Int) {
        store.getOrPut(key) { mutableListOf() }.add(timestamp to value)
        // No need to sort — set() guarantees increasing timestamps per key
    }

    fun get(key: String, timestamp: Int): String {
        val entries = store[key] ?: return ""

        var left = 0
        var right = entries.lastIndex
        var result = ""

        while (left <= right) {
            val mid = left + (right - left) / 2
            if (entries[mid].first <= timestamp) {
                result = entries[mid].second   // valid candidate — record it
                left = mid + 1                  // try for an even later timestamp
            } else {
                right = mid - 1                 // too late — search earlier
            }
        }

        return result
    }
}
```

### Approach 2 — Using Kotlin's built-in `binarySearch` with a comparator

```kotlin
class TimeMap {
    private val store = HashMap<String, MutableList<Int>>()
    private val values = HashMap<String, MutableList<String>>()

    fun set(key: String, value: String, timestamp: Int) {
        store.getOrPut(key) { mutableListOf() }.add(timestamp)
        values.getOrPut(key) { mutableListOf() }.add(value)
    }

    fun get(key: String, timestamp: Int): String {
        val timestamps = store[key] ?: return ""
        var idx = timestamps.binarySearch(timestamp)

        if (idx < 0) {
            // Not an exact match — binarySearch returns -(insertionPoint) - 1
            idx = -idx - 2   // step back to the largest value < timestamp
        }

        return if (idx >= 0) values[key]!![idx] else ""
    }
}
```

Uses the standard library's `List<Int>.binarySearch()`, which returns the
insertion point encoded as a negative number when there's no exact match.
Slightly trickier to get right, but avoids hand-rolling the binary search loop.

---

## Why "Find Floor" Needs a Tweaked Binary Search

This isn't a search for an *exact* match — we want the **largest timestamp
that doesn't exceed the query**. Standard binary search would return `-1`
the moment it doesn't find an exact value. Instead:

```kotlin
if (entries[mid].first <= timestamp) {
    result = entries[mid].second   // this is a VALID answer so far
    left = mid + 1                  // but maybe there's a later one that's still <= timestamp
} else {
    right = mid - 1                 // this timestamp is too far in the future
}
```

We keep recording the best candidate seen so far and keep searching right
for something even closer to (but not exceeding) the target — that's what
makes it a "floor" search rather than an exact-match search.

**`getOrPut`** avoids null-checking on every `set` call:
```kotlin
store.getOrPut(key) { mutableListOf() }.add(timestamp to value)
// Creates the list on first use, otherwise reuses the existing one
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Hand-rolled floor search (Approach 1) | Clearest to explain in an interview, single data structure per key |
| Built-in `binarySearch` (Approach 2) | Want to lean on the standard library, comfortable with its negative-index convention |

---

## Complexity

| | set | get |
|---|---|---|
| **Time** | O(1) amortized | O(log n) where n = number of `set` calls for that key |
| **Space** | O(n) total across all `set` calls | |

---

## Key Takeaway

> When timestamps arrive in guaranteed sorted order, you get binary search
> for free — no need to re-sort on every query. The twist here is "floor"
> search: instead of bailing out on a non-exact match, keep the best valid
> candidate and keep pushing right for something closer. This pattern —
> binary search for the largest value not exceeding a target — is common
> in any time-series or versioned-data lookup problem.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/time-based-key-value-store/)

---

{% include kotlin-dsa-nav.html %}
