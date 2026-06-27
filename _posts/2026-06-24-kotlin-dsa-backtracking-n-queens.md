---
title: "Backtracking: N Queens — Kotlin Solution"
date: 2026-06-24
categories: [Kotlin DSA, Backtracking]
tags: [n-queens, backtracking, matrix, hard, leetcode, leetcode-51]
leetcode_number: 51
leetcode_url: https://leetcode.com/problems/n-queens/
difficulty: Hard
permalink: /posts/kotlin-dsa-backtracking-n-queens/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [51 — N Queens](https://leetcode.com/problems/n-queens/) |
| **Difficulty** | Hard |
| **Topic** | Backtracking, Matrix |

> The n-queens puzzle: place `n` queens on an `n × n` chessboard so that
> no two queens attack each other (no shared row, column, or diagonal).
> Return all distinct solutions, each as a board configuration using
> `'Q'` for a queen and `'.'` for empty.

**Example:**
```
Input:  n = 4
Output: [[".Q..","...Q","Q...","..Q."],
          ["..Q.","Q...","...Q",".Q.."]]
Explanation: 2 distinct ways to place 4 non-attacking queens on a 4x4 board

Input:  n = 1
Output: [["Q"]]
```

**Constraints:**
- `1 <= n <= 9`

---

## Approach

This is the capstone problem of the Backtracking phase — it combines
multiple constraints simultaneously (row, column, and **two** diagonal
directions), and demonstrates how clever **O(1) constraint tracking**
turns an otherwise expensive validity check into instant lookups.

**Key insight — place one queen per row, track occupied columns and
diagonals with sets:** since no two queens can share a row, we can
process row by row, placing exactly one queen per row and trying every
column in that row. For each candidate column, check three things in
O(1): is this column already used? Is this "/"-diagonal already used?
Is this "\"-diagonal already used?

The diagonal identities are the elegant trick: every cell on the same
**"\"-diagonal** (top-left to bottom-right) shares the same `row - col`
value. Every cell on the same **"/"-diagonal** (top-right to bottom-left)
shares the same `row + col` value.

**Walk through placing queens for `n = 4`, partial trace:**
```
row=0: try col=0 → cols={0}, diag1(r-c)={0}, diag2(r+c)={0} → place Q at (0,0)
  row=1: try col=0 → cols already has 0 → skip
         try col=1 → diag1(1-1=0) already used → skip
         try col=2 → cols ok, diag1(1-2=-1) ok, diag2(1+2=3) ok → place Q at (1,2)
    row=2: try col=0 → diag2(2+0=2)? ok; diag1(2-0=2)? ok; col? ok → place Q at (2,0)
       Hold on — need col not used: cols={0,2}, so col=0 fails (already used)
       try col=1 → diag1(2-1=1) ok, diag2(2+1=3) ALREADY USED (from row1) → skip
       try col=3 → cols ok, diag1(2-3=-1) ALREADY USED (from row1) → skip
       → no valid column in row 2 → BACKTRACK, undo row1's placement, try col=3 instead

... (continues exploring) ...

Eventually finds both valid 4-queens arrangements ✓
```

---

## Kotlin Solution

### Approach 1 — Row-by-row placement, O(1) constraint sets for diagonals (optimal)

```kotlin
fun solveNQueens(n: Int): List<List<String>> {
    val result = mutableListOf<List<String>>()
    val cols = HashSet<Int>()
    val diag1 = HashSet<Int>()   // row - col (constant along "\" diagonals)
    val diag2 = HashSet<Int>()   // row + col (constant along "/" diagonals)
    val queenCol = IntArray(n)   // queenCol[row] = column of the queen in that row

    fun backtrack(row: Int) {
        if (row == n) {
            val board = (0 until n).map { r ->
                (0 until n).map { c -> if (c == queenCol[r]) 'Q' else '.' }
                    .joinToString("")
            }
            result.add(board)
            return
        }

        for (col in 0 until n) {
            if (col in cols || (row - col) in diag1 || (row + col) in diag2) continue

            cols.add(col); diag1.add(row - col); diag2.add(row + col)
            queenCol[row] = col

            backtrack(row + 1)

            cols.remove(col); diag1.remove(row - col); diag2.remove(row + col)
        }
    }

    backtrack(0)
    return result
}
```

### Approach 2 — Boolean array board, check column + both diagonals by scanning

```kotlin
fun solveNQueens(n: Int): List<List<String>> {
    val result = mutableListOf<List<String>>()
    val board = Array(n) { CharArray(n) { '.' } }

    fun isSafe(row: Int, col: Int): Boolean {
        for (r in 0 until row) {
            val c = board[r].indexOf('Q')
            if (c == col || Math.abs(row - r) == Math.abs(col - c)) return false
        }
        return true
    }

    fun backtrack(row: Int) {
        if (row == n) {
            result.add(board.map { String(it) })
            return
        }

        for (col in 0 until n) {
            if (isSafe(row, col)) {
                board[row][col] = 'Q'
                backtrack(row + 1)
                board[row][col] = '.'
            }
        }
    }

    backtrack(0)
    return result
}
```

`isSafe` re-scans all previously placed queens for every candidate
column — O(n) per check instead of Approach 1's O(1) set lookups, making
this version slower overall, though arguably more visually intuitive
since it mirrors directly looking at the board.

---

## Why `row - col` and `row + col` Uniquely Identify Diagonals

This is the single most valuable trick in this problem, and it's worth
internalizing the geometric intuition behind it:

```kotlin
diag1.add(row - col)   // "\" diagonal: as you move down-right, both row
                         // and col increase by 1 each step → row - col
                         // stays CONSTANT along this diagonal
diag2.add(row + col)   // "/" diagonal: as you move down-left, row
                         // increases while col decreases → row + col
                         // stays CONSTANT along this diagonal
```

Any two cells `(r1,c1)` and `(r2,c2)` lie on the same "\" diagonal if and
only if `r1 - c1 == r2 - c2`, and on the same "/" diagonal if and only if
`r1 + c1 == r2 + c2`. Tracking these two derived values in `HashSet`s
turns "is this diagonal already occupied?" into an O(1) lookup, instead
of Approach 2's O(n) scan through all previously placed queens.

**Why we only need ONE set per constraint type, not a 2D structure:**
since we place exactly one queen per row, there's no need to track
*which* row a column/diagonal conflict came from — simply knowing "this
column/diagonal value is taken by *some* queen" is sufficient to reject
the placement.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| O(1) constraint sets (Approach 1) | Always — significantly faster, the standard optimized solution |
| Boolean board scan (Approach 2) | Want a more visually direct mapping to the actual chessboard; acceptable for small `n` |

---

## Complexity

| | Approach 1 | Approach 2 |
|---|---|---|
| **Time** | O(n!) worst case for the search itself, O(1) per validity check | O(n!) search, O(n) per validity check |
| **Space** | O(n) — sets and recursion depth | O(n²) — board storage |

---

## Key Takeaway

> N-Queens closes out the Backtracking phase by combining everything:
> the "choose → recurse → undo" backbone from Subsets, the "track
> multiple simultaneous constraints" idea, and a genuinely clever
> mathematical trick — collapsing two diagonal-conflict checks into O(1)
> set lookups via the `row-col` / `row+col` invariants. Whenever a
> backtracking problem involves geometric or positional constraints,
> it's worth asking: is there a derived value that stays constant along
> the constraint I care about? If so, tracking that value directly (as
> done here for diagonals) can turn an O(n) validity check into O(1).

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/n-queens/)

---

{% include kotlin-dsa-nav.html %}
