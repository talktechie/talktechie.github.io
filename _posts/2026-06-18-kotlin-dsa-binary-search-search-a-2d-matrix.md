---
title: "Binary Search: Search a 2D Matrix — Kotlin Solution"
date: 2026-06-18
categories: [Kotlin DSA, Binary Search]
tags: [search-a-2d-matrix, binary-search, matrix, array, medium, leetcode, leetcode-74]
leetcode_number: 74
leetcode_url: https://leetcode.com/problems/search-a-2d-matrix/
difficulty: Medium
permalink: /posts/kotlin-dsa-binary-search-search-a-2d-matrix/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [74 — Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/) |
| **Difficulty** | Medium |
| **Topic** | Binary Search, Matrix, Array |

> You are given an `m x n` integer matrix with the following properties:
> - Each row is sorted in non-decreasing order.
> - The first integer of each row is greater than the last integer of
>   the previous row.
>
> Given an integer `target`, return `true` if it exists in the matrix.

**Example:**
```
Input:  matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 3
Output: true

Input:  matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 13
Output: false
```

**Constraints:**
- `m == matrix.length`
- `n == matrix[i].length`
- `1 <= m, n <= 100`
- `-10⁴ <= matrix[i][j], target <= 10⁴`

---

## Approach

The matrix's two properties combined mean: **if you read it row by row, left
to right, the entire matrix is one giant sorted sequence**. That's the
unlock — we don't need two separate binary searches (one for the row, one
for the column). We can binary search the whole thing as if it were a
flattened 1D array.

**Key insight — index mapping:** For a flattened index `i` in a matrix with
`n` columns:
```
row = i / n
col = i % n
```

**Walk through `matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 16`:**
```
Flattened view: [1,3,5,7,10,11,16,20,23,30,34,60]  (m=3, n=4, total=12)

left=0, right=11 → mid=5 → row=5/4=1, col=5%4=1 → matrix[1][1]=11 → 11<16 → left=6
left=6, right=11 → mid=8 → row=8/4=2, col=8%4=0 → matrix[2][0]=23 → 23>16 → right=7
left=6, right=7  → mid=6 → row=6/4=1, col=6%4=2 → matrix[1][2]=16 → 16==16 → found! ✓
```

---

## Kotlin Solution

### Approach 1 — Treat as flattened 1D array (optimal)

```kotlin
fun searchMatrix(matrix: Array<IntArray>, target: Int): Boolean {
    val rows = matrix.size
    val cols = matrix[0].size

    var left = 0
    var right = rows * cols - 1

    while (left <= right) {
        val mid = left + (right - left) / 2
        val midValue = matrix[mid / cols][mid % cols]

        when {
            midValue == target -> return true
            midValue < target  -> left = mid + 1
            else                -> right = mid - 1
        }
    }

    return false
}
```

### Approach 2 — Two-step binary search (find row, then column)

```kotlin
fun searchMatrix(matrix: Array<IntArray>, target: Int): Boolean {
    val rows = matrix.size
    val cols = matrix[0].size

    // Step 1 — find the candidate row: the last row whose first element <= target
    var top = 0
    var bottom = rows - 1
    while (top < bottom) {
        val mid = top + (bottom - top + 1) / 2
        if (matrix[mid][0] <= target) top = mid else bottom = mid - 1
    }
    val row = top

    // Step 2 — binary search within that row
    var left = 0
    var right = cols - 1
    while (left <= right) {
        val mid = left + (right - left) / 2
        when {
            matrix[row][mid] == target -> return true
            matrix[row][mid] < target  -> left = mid + 1
            else                        -> right = mid - 1
        }
    }

    return false
}
```

Two binary searches instead of one — same overall O(log(m·n)) complexity
since `log(m) + log(n) = log(m·n)`, but more code to maintain.

---

## Why the Flattened Approach Is Simpler

Both approaches are O(log(m × n)), but Approach 1 is strictly less code and
less room for off-by-one mistakes, because it reduces the 2D problem to the
exact same template as LC 704.

**The index mapping is the entire trick:**
```kotlin
val midValue = matrix[mid / cols][mid % cols]
// mid=6, cols=4 → row=6/4=1 (integer division), col=6%4=2
// This works because rows are laid out end-to-end: row 0 occupies
// indices 0..cols-1, row 1 occupies cols..2*cols-1, and so on.
```

**`rows * cols - 1`** as the initial right boundary — same as treating the
matrix as `Array<Int>` of size `rows * cols`:
```kotlin
var right = rows * cols - 1
```

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Flattened 1D search (Approach 1) | Always — simpler, same complexity, fewer bugs |
| Two-step search (Approach 2) | Educational — shows the row/column decomposition explicitly |

---

## Complexity

| | |
|---|---|
| **Time** | O(log(m × n)) |
| **Space** | O(1) |

---

## Key Takeaway

> When a 2D structure has a global ordering (every row continues where the
> previous one ended), treat it as a 1D array using index mapping
> `row = i / cols`, `col = i % cols`. This collapses a seemingly 2D search
> problem into the exact same binary search template from LC 704 — no new
> algorithm needed, just a coordinate transform.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/search-a-2d-matrix/)

---

{% include kotlin-dsa-nav.html %}
