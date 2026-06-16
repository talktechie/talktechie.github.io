---
title: "Arrays: Valid Sudoku — Kotlin Solution"
date: 2026-06-16
categories: [Kotlin DSA, Arrays]
tags: [valid-sudoku, hashset, array, matrix, medium, leetcode, leetcode-36]
leetcode_number: 36
leetcode_url: https://leetcode.com/problems/valid-sudoku/
difficulty: Medium
permalink: /posts/kotlin-dsa-arrays-valid-sudoku/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [36 — Valid Sudoku](https://leetcode.com/problems/valid-sudoku/) |
| **Difficulty** | Medium |
| **Topic** | Arrays, HashSet, Matrix |

> Determine if a 9 × 9 Sudoku board is valid. Only the filled cells need
> to be validated according to the following rules:
>
> 1. Each **row** must contain the digits `1–9` without repetition.
> 2. Each **column** must contain the digits `1–9` without repetition.
> 3. Each of the nine **3 × 3 sub-boxes** must contain the digits `1–9` without repetition.
>
> Note: the board may be partially filled. Empty cells are marked `'.'`.
> A valid board is not necessarily solvable.

**Example:**
```
Input: board =
[["5","3",".",".","7",".",".",".","."],
 ["6",".",".","1","9","5",".",".","."],
 [".","9","8",".",".",".",".","6","."],
 ["8",".",".",".","6",".",".",".","3"],
 ["4",".",".","8",".","3",".",".","1"],
 ["7",".",".",".","2",".",".",".","6"],
 [".","6",".",".",".",".","2","8","."],
 [".",".",".","4","1","9",".",".","5"],
 [".",".",".",".","8",".",".","7","9"]]
Output: true

Input: board (same but top-left changed to "8"):
Output: false  // column 0 now has two 8s
```

**Constraints:**
- `board.length == 9`
- `board[i].length == 9`
- `board[i][j]` is a digit `'1'–'9'` or `'.'`

---

## Approach

The board has three independent constraints — rows, columns, and 3×3 boxes.
Each cell participates in exactly one of each. A single pass over all 81 cells
can check all three simultaneously by maintaining three sets of sets.

**Key insight — box index formula:**
```
boxRow = row / 3      (maps rows 0-2 → box 0, rows 3-5 → box 1, rows 6-8 → box 2)
boxCol = col / 3      (same for columns)
boxIndex = boxRow * 3 + boxCol   (maps the 9 boxes to indices 0–8)
```

**Walk through cell `(0, 0) = '5'`:**
```
rows[0]     → add '5' → ok
cols[0]     → add '5' → ok
boxes[0*3 + 0 = 0] → add '5' → ok

Cell (3, 0) = '8':
rows[3]     → add '8' → ok
cols[0]     → add '8' → ok (col 0 had only '5','6' so far)
boxes[1*3 + 0 = 3] → add '8' → ok

If top-left were also '8':
Cell (0, 0) = '8' and cell (3, 0) = '8':
cols[0] already contains '8' → return false ✓
```

---

## Kotlin Solution

### Approach 1 — Single pass, three sets of HashSets (optimal)

```kotlin
fun isValidSudoku(board: Array<CharArray>): Boolean {
    val rows  = Array(9) { HashSet<Char>() }
    val cols  = Array(9) { HashSet<Char>() }
    val boxes = Array(9) { HashSet<Char>() }

    for (r in 0..8) {
        for (c in 0..8) {
            val cell = board[r][c]
            if (cell == '.') continue

            val boxIndex = (r / 3) * 3 + (c / 3)

            if (!rows[r].add(cell))     return false
            if (!cols[c].add(cell))     return false
            if (!boxes[boxIndex].add(cell)) return false
        }
    }

    return true
}
```

`HashSet.add()` returns `false` if the element was already present — perfect
for duplicate detection with no extra `contains` check needed.

### Approach 2 — Encode seen state as strings (no nested sets)

```kotlin
fun isValidSudoku(board: Array<CharArray>): Boolean {
    val seen = HashSet<String>()

    for (r in 0..8) {
        for (c in 0..8) {
            val cell = board[r][c]
            if (cell == '.') continue

            val boxIndex = (r / 3) * 3 + (c / 3)

            // Each key is unique to its constraint type + position + value
            if (!seen.add("r$r:$cell"))   return false
            if (!seen.add("c$c:$cell"))   return false
            if (!seen.add("b$boxIndex:$cell")) return false
        }
    }

    return true
}
```

One `HashSet<String>` instead of three — slightly more allocations but
simpler to write and easy to reason about in an interview.

---

## Why the Box Index Formula Works

```
(r / 3) * 3 + (c / 3)

r=0,c=0 → 0*3+0 = 0   (top-left box)
r=0,c=3 → 0*3+1 = 1   (top-centre box)
r=0,c=6 → 0*3+2 = 2   (top-right box)
r=3,c=0 → 1*3+0 = 3   (middle-left box)
r=6,c=6 → 2*3+2 = 8   (bottom-right box)
```

Integer division naturally groups rows/columns into threes, and multiplying by
3 then adding offsets them into unique indices 0–8.

**`!set.add(element)`** — clean duplicate check in one expression:
```kotlin
if (!rows[r].add(cell)) return false
// add() inserts and returns true if new, false if already present
// negating it → "if already seen, return false"
// vs:  if (rows[r].contains(cell)) return false; else rows[r].add(cell)
```

**`Array(9) { HashSet<Char>() }`** — Kotlin array initializer with lambda:
```kotlin
val rows = Array(9) { HashSet<Char>() }
// Creates 9 distinct HashSet instances — not 9 references to the same one
// vs Java: HashSet<Character>[] rows = new HashSet[9];
//          for (int i=0; i<9; i++) rows[i] = new HashSet<>();
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Three `HashSet` arrays (Approach 1) | Best performance, most readable structure |
| Single `HashSet<String>` (Approach 2) | Interview — fewer variables, easy to explain |

---

## Complexity

| | |
|---|---|
| **Time** | O(1) — always 81 cells, fixed board size |
| **Space** | O(1) — at most 9 × 9 = 81 entries across all sets (fixed bound) |

---

## Key Takeaway

> Three constraints, one pass. For each non-empty cell, check its row set,
> column set, and box set simultaneously. The box index formula
> `(r/3)*3 + (c/3)` is the trick worth memorising — it maps any cell to
> its 3×3 box index in O(1). `HashSet.add()` returning false is cleaner
> than a separate `contains` check.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/valid-sudoku/)

---

{% include kotlin-dsa-nav.html %}
