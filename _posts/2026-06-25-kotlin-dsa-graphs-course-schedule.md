---
title: "Graphs: Course Schedule — Kotlin Solution"
date: 2026-06-25
categories: [Kotlin DSA, Graphs]
tags: [course-schedule, graph, dfs, bfs, topological-sort, cycle-detection, medium, leetcode, leetcode-207]
leetcode_number: 207
leetcode_url: https://leetcode.com/problems/course-schedule/
difficulty: Medium
permalink: /posts/kotlin-dsa-graphs-course-schedule/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [207 — Course Schedule](https://leetcode.com/problems/course-schedule/) |
| **Difficulty** | Medium |
| **Topic** | Graph, DFS, BFS, Topological Sort, Cycle Detection |

> There are `numCourses` courses labeled `0` to `numCourses-1`. Given
> `prerequisites[i] = [a, b]` meaning you must take course `b` before
> course `a`, return `true` if it's possible to finish **all** courses.

**Example:**
```
Input:  numCourses = 2, prerequisites = [[1,0]]
Output: true
Explanation: take course 0, then course 1

Input:  numCourses = 2, prerequisites = [[1,0],[0,1]]
Output: false
Explanation: course 1 needs course 0, but course 0 needs course 1 — a
             cycle, impossible to satisfy
```

**Constraints:**
- `1 <= numCourses <= 2000`
- `0 <= prerequisites.length <= 5000`
- `prerequisites[i].length == 2`
- All pairs are unique

---

## Approach

This is fundamentally a **cycle detection** problem in a directed graph.
If course `a` depends on `b`, and (transitively) `b` depends back on
`a`, finishing all courses is impossible — that's a **cycle**. The
question "can all courses be completed?" is exactly "does this directed
graph have a cycle?"

**Key insight — DFS with three states:** track each course as
**unvisited**, **visiting** (currently on the DFS call stack — part of
the current exploration path), or **visited** (fully processed, known
to be cycle-free). If DFS ever encounters a node that's currently
**visiting** (not just visited), that means we've looped back onto our
own current path — a cycle.

**Walk through `numCourses = 2, prerequisites = [[1,0],[0,1]]`** (course
1 requires 0, course 0 requires 1):
```
dfs(course=0): mark VISITING
  prerequisite of 0 is... wait, build the graph correctly:
  prerequisites[i]=[a,b] means b → a (take b first). So edges: 0→1, 1→0

dfs(0): mark 0 as VISITING
  neighbor 1: dfs(1): mark 1 as VISITING
    neighbor 0: dfs(0) → 0 is currently VISITING → CYCLE DETECTED → return false

Result: false ✓ (correctly identifies the circular dependency)
```

---

## Kotlin Solution

### Approach 1 — DFS with 3-state cycle detection (optimal, most intuitive)

```kotlin
fun canFinish(numCourses: Int, prerequisites: Array<IntArray>): Boolean {
    val graph = Array(numCourses) { mutableListOf<Int>() }
    for ((course, prereq) in prerequisites) {
        graph[prereq].add(course)   // prereq must come before course
    }

    val UNVISITED = 0; val VISITING = 1; val VISITED = 2
    val state = IntArray(numCourses)

    fun hasCycle(node: Int): Boolean {
        if (state[node] == VISITING) return true   // back edge — cycle!
        if (state[node] == VISITED) return false    // already cleared, no cycle here

        state[node] = VISITING
        for (neighbor in graph[node]) {
            if (hasCycle(neighbor)) return true
        }
        state[node] = VISITED

        return false
    }

    for (course in 0 until numCourses) {
        if (hasCycle(course)) return false
    }

    return true
}
```

### Approach 2 — BFS, Kahn's algorithm (topological sort via in-degree)

```kotlin
fun canFinish(numCourses: Int, prerequisites: Array<IntArray>): Boolean {
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

    var processedCount = 0

    while (queue.isNotEmpty()) {
        val course = queue.removeFirst()
        processedCount++

        for (next in graph[course]) {
            inDegree[next]--
            if (inDegree[next] == 0) queue.addLast(next)
        }
    }

    return processedCount == numCourses
}
```

If every course can eventually reach in-degree 0 and get processed, there
was no cycle. If some courses are stuck with in-degree > 0 forever (their
processing count never reaches `numCourses`), a cycle exists among them.

---

## Why the "Visiting" State (Not Just Visited/Unvisited) Is Necessary

A naive 2-state visited tracker (just `true`/`false`) **cannot**
distinguish a cycle from a diamond-shaped dependency (where two different
paths legitimately converge on the same already-fully-processed node):

```kotlin
if (state[node] == VISITING) return true   // currently on OUR current path → cycle
if (state[node] == VISITED) return false    // fully explored already, safe to skip
```

`VISITING` specifically means "this node is an ancestor of the node we're
currently examining, still on the active call stack." Encountering a
`VISITING` node means we've looped back onto our own exploration path —
a genuine cycle. Encountering a `VISITED` node, by contrast, means we
already fully explored it from some **other** branch and confirmed it's
cycle-free — perfectly fine to revisit without re-exploring or treating
it as a problem.

**Why Kahn's algorithm (Approach 2) reaches the same conclusion from a
totally different angle:** if a cycle exists, every node in that cycle
permanently depends on another node within the same cycle, so none of
them can ever reach in-degree 0 — they're "stuck" forever, and
`processedCount` will fall short of `numCourses`.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| DFS 3-state (Approach 1) | Most intuitive for cycle detection specifically |
| BFS / Kahn's algorithm (Approach 2) | Want the actual topological order as a byproduct (used directly in Course Schedule II) |

---

## Complexity

| | |
|---|---|
| **Time** | O(V + E) — V courses, E prerequisite pairs |
| **Space** | O(V + E) — adjacency list, plus state tracking |

---

## Key Takeaway

> "Can all tasks be completed given dependencies" is always a cycle
> detection question on a directed graph. DFS with three explicit states
> (unvisited/visiting/visited) is the standard way to distinguish a true
> cycle from a harmless re-convergence of paths. Kahn's algorithm
> (BFS-based, using in-degree counts) answers the same question from the
> opposite direction — and, as a bonus, naturally produces a valid
> execution order when no cycle exists, which is exactly what Course
> Schedule II needs next.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/course-schedule/)

---

{% include kotlin-dsa-nav.html %}
