---
title: "Heap: Task Scheduler — Kotlin Solution"
date: 2026-06-23
categories: [Kotlin DSA, Heap]
tags: [task-scheduler, heap, priority-queue, greedy, array, medium, leetcode, leetcode-621]
leetcode_number: 621
leetcode_url: https://leetcode.com/problems/task-scheduler/
difficulty: Medium
permalink: /posts/kotlin-dsa-heap-task-scheduler/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [621 — Task Scheduler](https://leetcode.com/problems/task-scheduler/) |
| **Difficulty** | Medium |
| **Topic** | Heap, Priority Queue, Greedy, Array |

> Given a list of CPU `tasks` (each represented by a character) and a
> cooldown period `n`, where the **same** task type must wait at least
> `n` intervals before running again, return the minimum number of time
> units needed to finish all tasks (idle slots count toward the total).

**Example:**
```
Input:  tasks = ["A","A","A","B","B","B"], n = 2
Output: 8
Explanation: A -> B -> idle -> A -> B -> idle -> A -> B
             (A and B each need 2 slots of cooldown before repeating)

Input:  tasks = ["A","A","A","B","B","B"], n = 0
Output: 6
Explanation: no cooldown needed — just run them in any order
```

**Constraints:**
- `1 <= tasks.length <= 10⁴`
- `tasks[i]` is an uppercase English letter
- `0 <= n <= 100`

---

## Approach

**Key insight — greedy, always run the most frequent remaining task:** to
minimize idle time, always schedule whichever task currently has the
**most remaining occurrences** — this spreads out the most "constrained"
tasks as early as possible, maximizing the chance that other tasks fill
in the cooldown gaps.

A **max-heap** of task counts naturally gives "most frequent remaining
task" in O(log 26) ≈ O(1) per step (at most 26 distinct task types, since
they're uppercase letters).

**The mechanics — track a cooldown queue:** after running a task, it
can't run again for `n` intervals. Use a small queue to hold
`(remainingCount, availableAgainAtTime)` pairs, and only push a task back
into the max-heap once its cooldown has expired.

**Walk through `tasks = ["A","A","A","B","B","B"], n = 2`:**
```
counts: {A:3, B:3}   max-heap: [3(A), 3(B)]

time=0: pop A(3) → run A, push (A,2) to cooldown queue (available at time 0+2+1=3)
        heap: [3(B)]
time=1: pop B(3) → run B, push (B,2) to cooldown queue (available at time 1+2+1=4)
        heap: []
time=2: heap empty, check cooldown queue: A available at time 3? not yet (2<3) → idle
time=3: A's cooldown expired → push A(2) back to heap → pop A(2) → run A
        push (A,1) available at time 3+2+1=6
time=4: B's cooldown expired → push B(2) back → pop B(2) → run B
        push (B,1) available at time 4+2+1=7
time=5: heap empty (both on cooldown) → idle
time=6: A available → run A (last one, count becomes 0, don't requeue)
time=7: B available → run B (last one, done)

Total time units used: 8 (indices 0-7) ✓
```

---

## Kotlin Solution

### Approach 1 — Max-heap + cooldown queue simulation (optimal, intuitive)

```kotlin
fun leastInterval(tasks: CharArray, n: Int): Int {
    val counts = IntArray(26)
    for (t in tasks) counts[t - 'A']++

    val maxHeap = PriorityQueue<Int>(compareByDescending { it })
    for (c in counts) if (c > 0) maxHeap.add(c)

    val cooldownQueue = ArrayDeque<Pair<Int, Int>>()  // (remainingCount, availableAtTime)
    var time = 0

    while (maxHeap.isNotEmpty() || cooldownQueue.isNotEmpty()) {
        time++

        if (maxHeap.isNotEmpty()) {
            val count = maxHeap.poll() - 1
            if (count > 0) cooldownQueue.addLast(count to (time + n))
        }

        if (cooldownQueue.isNotEmpty() && cooldownQueue.first().second == time) {
            val (count, _) = cooldownQueue.removeFirst()
            maxHeap.add(count)
        }
    }

    return time
}
```

### Approach 2 — Math formula (no simulation, O(1) after counting)

```kotlin
fun leastInterval(tasks: CharArray, n: Int): Int {
    val counts = IntArray(26)
    for (t in tasks) counts[t - 'A']++

    val maxCount = counts.max()
    val maxCountFrequency = counts.count { it == maxCount }

    val intervalLength = (maxCount - 1) * (n + 1) + maxCountFrequency

    return maxOf(tasks.size, intervalLength)
}
```

A closed-form formula: the most frequent task creates `(maxCount - 1)`
full cooldown cycles of length `(n+1)`, plus however many other tasks
share that same maximum frequency (they fill the very last slots).
If there are enough other distinct tasks to fill every gap, the answer
is simply `tasks.size` — no idle time needed at all, which is why we
take the max of both quantities.

---

## Why the Heap + Cooldown Queue Simulation Mirrors Real Scheduling

This approach is worth understanding even though the formula
(Approach 2) is faster, because it generalizes to scheduling variants
that don't have a clean closed-form solution.

```kotlin
val (count, _) = cooldownQueue.removeFirst()
maxHeap.add(count)
// A task becomes "available again" exactly n+1 time units after it last
// ran — at that point, re-enter it into contention for being chosen next
```

**Why we check `cooldownQueue.first().second == time` every tick:** the
queue is naturally ordered by availability time (since tasks are added in
the order they go on cooldown, and cooldown duration is uniform), so the
front of the queue is always the next task to become available, if any.

**Why the formula's `maxCountFrequency` term matters:** if multiple tasks
tie for the highest frequency, they fill in each other's cooldown gaps
during the final cycle, meaning we don't actually need additional idle
time for those — that's the `+ maxCountFrequency` term, replacing what
would otherwise need to be more idle slots.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Heap + cooldown simulation (Approach 1) | Want to understand the mechanics, or facing variants without a clean formula |
| Math formula (Approach 2) | Fastest, most concise — the ideal answer once you've derived/memorized it |

---

## Complexity

| | Heap Simulation | Formula |
|---|---|---|
| **Time** | O(tasks.length) — at most 26 distinct counts in the heap | O(tasks.length) — single counting pass |
| **Space** | O(1) — bounded by 26 task types | O(1) |

---

## Key Takeaway

> Greedily running the most frequent remaining task first, using a
> max-heap to always know which task that is, naturally minimizes idle
> time by spreading out the most constrained tasks early. The cooldown
> queue models exactly when a task becomes "available again," mirroring
> real-world job scheduling. The closed-form formula is a nice shortcut
> once derived, but the heap-based simulation is the more broadly
> applicable technique, and demonstrates greedy-plus-heap combined with a
> time-based eligibility queue — a pattern useful well beyond this
> specific problem.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/task-scheduler/)

---

{% include kotlin-dsa-nav.html %}
