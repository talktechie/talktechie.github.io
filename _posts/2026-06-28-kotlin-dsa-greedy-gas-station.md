---
title: "Greedy: Gas Station — Kotlin Solution"
date: 2026-06-28
categories: [Kotlin DSA, Greedy]
tags: [gas-station, greedy, array, medium, leetcode, leetcode-134]
leetcode_number: 134
leetcode_url: https://leetcode.com/problems/gas-station/
difficulty: Medium
permalink: /posts/kotlin-dsa-greedy-gas-station/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [134 — Gas Station](https://leetcode.com/problems/gas-station/) |
| **Difficulty** | Medium |
| **Topic** | Greedy, Array |

> There are `n` gas stations in a circle. `gas[i]` is the fuel available
> at station `i`; `cost[i]` is the fuel needed to travel from station
> `i` to station `i+1`. Starting with an empty tank, return the starting
> station index from which you can complete the full circuit, or `-1` if
> impossible. The answer is guaranteed unique if it exists.

**Example:**
```
Input:  gas = [1,2,3,4,5], cost = [3,4,5,1,2]
Output: 3
Explanation: starting at station 3: tank=4-1=3, then +5-2=6, then
             +1-3=4, then +2-4=2, then +3-5=0 — completes the circuit
             with exactly 0 left, having visited every station

Input:  gas = [2,3,4], cost = [3,4,3]
Output: -1
Explanation: total gas (9) < total cost (10) — impossible regardless
             of starting point
```

**Constraints:**
- `n == gas.length == cost.length`
- `1 <= n <= 10⁵`
- `0 <= gas[i], cost[i] <= 10⁴`

---

## Approach

The brute force — try every starting station, simulate the full circuit
— is O(n²). Two greedy insights bring this down to O(n).

**Key insight 1 — a feasible start exists if and only if total gas ≥
total cost:** if the sum of all gas collected is less than the sum of
all costs across the entire circuit, it's mathematically impossible to
complete the loop from *any* starting point — return `-1` immediately.

**Key insight 2 — if a feasible solution exists, greedily simulating
from station 0 and restarting at the first failure point finds it
directly:** track a running tank balance. The moment it goes negative at
some station `i`, **none** of the stations from the previous start point
up through `i` could have been valid starting points either (starting
anywhere in that already-failed stretch would run out of gas even
sooner) — so greedily set the **next** station (`i+1`) as the new
candidate start, and reset the tank to 0.

**Walk through `gas = [1,2,3,4,5], cost = [3,4,5,1,2]`:**
```
totalGas = 15, totalCost = 15 → 15 >= 15 → a solution exists somewhere

start=0, tank=0

i=0: tank += gas[0]-cost[0] = 1-3 = -2 → tank=-2 → NEGATIVE!
  → station 0 can't be the start, and neither can any station we'd
    have tried in between (there are none yet) → new candidate start=1, tank=0

i=1: tank += gas[1]-cost[1] = 2-4 = -2 → tank=-2 → NEGATIVE!
  → new candidate start=2, tank=0

i=2: tank += gas[2]-cost[2] = 3-5 = -2 → tank=-2 → NEGATIVE!
  → new candidate start=3, tank=0

i=3: tank += gas[3]-cost[3] = 4-1 = 3 → tank=3 (still positive, keep going)
i=4: tank += gas[4]-cost[4] = 5-2 = 3 → tank=6 (still positive)

Loop completes without going negative again → answer: start=3 ✓
```

---

## Kotlin Solution

### Approach 1 — Total feasibility check + single greedy pass (optimal)

```kotlin
fun canCompleteCircuit(gas: IntArray, cost: IntArray): Int {
    if (gas.sum() < cost.sum()) return -1   // globally impossible

    var start = 0
    var tank = 0

    for (i in gas.indices) {
        tank += gas[i] - cost[i]

        if (tank < 0) {
            start = i + 1   // this stretch failed — try starting fresh from the next station
            tank = 0
        }
    }

    return start
}
```

### Approach 2 — Brute force simulation from every station (for contrast, O(n²))

```kotlin
fun canCompleteCircuit(gas: IntArray, cost: IntArray): Int {
    val n = gas.size

    for (start in 0 until n) {
        var tank = 0
        var completed = true

        for (step in 0 until n) {
            val i = (start + step) % n
            tank += gas[i] - cost[i]
            if (tank < 0) {
                completed = false
                break
            }
        }

        if (completed) return start
    }

    return -1
}
```

Tries every possible starting station explicitly, simulating the full
circular route each time — correct, but redundantly re-derives
information the greedy approach extracts in a single pass.

---

## Why Failing at Station `i` Disqualifies Every Earlier Candidate Too

This is the subtle but crucial greedy insight that makes the single-pass
approach valid, and it's worth proving to yourself why it's airtight:

```kotlin
if (tank < 0) {
    start = i + 1
    tank = 0
}
```

Suppose we started accumulating from some candidate station `s`, and the
tank goes negative upon reaching station `i`. This means the **total**
gas collected from `s` through `i` is less than the **total** cost over
that same stretch. Now consider any station `s' ` strictly between `s`
and `i`: starting from `s'` instead would mean accumulating gas/cost only
over the **shorter** stretch from `s'` to `i` — but since the **full**
stretch from `s` to `i` was already insufficient, and the portion from
`s` to `s'` (which we're now excluding) had a **non-negative** running
total at every point up to its own failure (otherwise we'd have already
restarted earlier) — removing a non-negative-contributing prefix can
only make the remaining deficit the same or worse, never better. So `s'`
would fail too, by an equal or larger margin. This proves every station
strictly between the previous start and the failure point can be safely
skipped.

**Why checking `gas.sum() < cost.sum()` upfront matters:** the
single-pass greedy scan alone, without this check, could still terminate
having found *some* candidate `start` value — but that candidate might
not actually survive a *second* full lap (since the scan only verifies
forward through the array once, not the wraparound). The total-sum check
is what guarantees a solution exists globally before trusting the
greedy scan's specific answer.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Single greedy pass (Approach 1) | Always — O(n), the correct and intended solution |
| Brute force simulation (Approach 2) | Never for submission — useful only to validate understanding by contrast |

---

## Complexity

| | Greedy | Brute Force |
|---|---|---|
| **Time** | O(n) | O(n²) |
| **Space** | O(1) | O(1) |

---

## Key Takeaway

> Gas Station combines two greedy insights: a global feasibility
> precheck (total gas vs. total cost) and a local "skip everything up to
> and including the failure point" rule that's provably safe, not just
> heuristically convenient. The proof that skipping intermediate
> candidates never loses the true answer is what elevates this from "a
> trick that happens to work" to "a greedy algorithm with a rigorous
> correctness argument" — exactly the kind of reasoning worth applying
> whenever a greedy approach feels right but its correctness isn't
> immediately obvious.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/gas-station/)

---

{% include kotlin-dsa-nav.html %}
