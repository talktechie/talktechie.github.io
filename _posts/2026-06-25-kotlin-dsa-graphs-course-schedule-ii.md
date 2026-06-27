---
title: "Graphs: Course Schedule II — Kotlin Solution"
date: 2026-06-25
categories: [Kotlin DSA, Graphs]
tags: [course-schedule-ii, graph, dfs, bfs, topological-sort, medium, leetcode, leetcode-210]
leetcode_number: 210
leetcode_url: https://leetcode.com/problems/course-schedule-ii/
difficulty: Medium
permalink: /posts/kotlin-dsa-graphs-course-schedule-ii/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [210 — Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) |
| **Difficulty** | Medium |
| **Topic** | Graph, DFS, BFS, Topological Sort |

> Same setup as Course Schedule — return **a valid ordering** of courses
> that satisfies every prerequisite, or an empty array if no valid
> ordering exists (a cycle is present).

**Example:**
```
Input:  numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]
Output: [0,1,2,3] or [0,2,1,3]
Explanation: 0 must come first; 1 and 2 both need 0; 3 needs both 1 and 2

Input:  numCourses = 2, prerequisites = [[1,0],[0,1]]
Output: []
Explanation: a cycle — no valid ordering exists
```

**Constraints:**
- `1 <= numCourses <= 2000`
- `0 <= prerequisites.length <= numCourses × (numCourses - 1)`
- All pairs are unique

---

## Approach

This extends Course Schedule (LC 207) from "does a valid order exist"
(boolean) to "what **is** that order" (the actual sequence) — both
algorithms from LC 207 generalize naturally, since they were already
computing an ordering-related structure internally.

**Key insight (Kahn's / BFS):** Process courses in order of their
in-degree reaching 0 — exactly the BFS approach from Course Schedule.
The **order in which courses get dequeued** IS a valid topological
ordering. If a cycle exists, some courses never reach in-degree 0 and
never get dequeued, so the final result list will be shorter than
`numCourses` — signaling failure.

**Key insight (DFS):** Run DFS cycle detection as before, but
**additionally** record each node in a result list the moment it
finishes being fully explored (`VISITED`). Since DFS finishes a node's
descendants before the node itself, this naturally produces a **reverse
topological order** — reverse the final list to get the correct order.

**Walk through `numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]`
using BFS:**
```
graph: 0→1, 0→2, 1→3, 2→3
inDegree: course0=0, course1=1, course2=1, course3=2

queue starts with in-degree-0 courses: [0]

dequeue 0 → add to order=[0] → decrement inDegree of 1,2 → both become 0
  → enqueue 1, 2 → queue=[1,2]
dequeue 1 → order=[0,1] → decrement inDegree of 3 → becomes 1 (not yet 0)
dequeue 2 → order=[0,1,2] → decrement inDegree of 3 → becomes 0 → enqueue 3
dequeue 3 → order=[0,1,2,3]

order.size == numCourses(4) → valid! return [0,1,2,3] ✓
```

---

## Kotlin Solution

### Approach 1 — BFS / Kahn's algorithm, dequeue order IS the answer (optimal)

```kotlin
fun findOrder(numCourses: Int, prerequisites: Array<IntArray>): IntArray {
    val graph = Array(numCourses) { mutableListOf<Int>() }
    val inDegree = IntArray(numCourses)

    for ((course, prereq) in prerequisites) {
        graph[prereq].add(course)
        inDegree[course]++
    }

    val queue: ArrayDeque<Int> = ArrayDeque()
    for (course in 0 until numCourses) {
        if (inDegree[course] == 0) queue.addLast(course)
    }

    val order = mutableListOf<Int>()

    while (queue.isNotEmpty()) {
        val course = queue.removeFirst()
        order.add(course)

        for (next in graph[course]) {
            inDegree[next]--
            if (inDegree[next] == 0) queue.addLast(next)
        }
    }

    return if (order.size == numCourses) order.toIntArray() else IntArray(0)
}
```

### Approach 2 — DFS, append to result on finishing a node, then reverse

```kotlin
fun findOrder(numCourses: Int, prerequisites: Array<IntArray>): IntArray {
    val graph = Array(numCourses) { mutableListOf<Int>() }
    for ((course, prereq) in prerequisites) {
        graph[prereq].add(course)
    }

    val UNVISITED = 0; val VISITING = 1; val VISITED = 2
    val state = IntArray(numCourses)
    val order = mutableListOf<Int>()
    var hasCycle = false

    fun dfs(node: Int) {
        if (hasCycle || state[node] == VISITED) return
        if (state[node] == VISITING) { hasCycle = true; return }

        state[node] = VISITING
        for (neighbor in graph[node]) {
            dfs(neighbor)
        }
        state[node] = VISITED

        order.add(node)   // record AFTER all descendants are fully processed
    }

    for (course in 0 until numCourses) {
        dfs(course)
    }

    if (hasCycle) return IntArray(0)

    return order.reversed().toIntArray()   // reverse — DFS finishes leaves first
}
```

---

## Why BFS's Dequeue Order Is Automatically a Valid Topological Order

```kotlin
val course = queue.removeFirst()
order.add(course)
```

A course is only ever enqueued once **every** one of its prerequisites
has already been dequeued (its in-degree reaches 0 only after all
incoming edges have been "consumed" by earlier dequeues). This is
precisely the definition of a valid ordering — every course appears
after all of its prerequisites.

**Why DFS's finish order must be reversed:**
```kotlin
order.add(node)   // happens AFTER recursing into all neighbors (descendants)
// ...
return order.reversed().toIntArray()
```
DFS naturally finishes (marks `VISITED`) a node's descendants **before**
the node itself returns — so the raw `order` list ends up with
"dependencies" appearing *after* the things that depend on them
(reverse of what we want). Reversing the list at the end corrects this
into proper dependency-respecting order.

**Why checking `order.size == numCourses` (BFS) or `hasCycle` (DFS) is
the correct failure signal:** both mirror Course Schedule's cycle
detection exactly — if a cycle exists, some courses can never be
"finished" by either algorithm, so the count falls short or the cycle
flag gets set.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| BFS / Kahn's (Approach 1) | Most direct — the dequeue order IS the answer with zero post-processing |
| DFS (Approach 2) | Comfortable with recursion, fine with the extra reversal step |

---

## Complexity

| | |
|---|---|
| **Time** | O(V + E) |
| **Space** | O(V + E) |

---

## Key Takeaway

> Course Schedule II shows that "does a valid topological order exist"
> and "what is that order" are solved by the **same** algorithms, just
> reading off the answer differently — BFS's dequeue sequence directly
> IS the order, while DFS's finish sequence needs one reversal to
> become correct. Recognizing that a Yes/No graph algorithm often
> already contains the richer answer as a byproduct (here, the actual
> ordering) is a valuable habit: before reaching for a different
> algorithm, check whether the one you already know naturally produces
> more information than you initially needed.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/course-schedule-ii/)

---

{% include kotlin-dsa-nav.html %}
