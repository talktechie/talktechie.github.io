---
title: "Heap: Design Twitter — Kotlin Solution"
date: 2026-06-23
categories: [Kotlin DSA, Heap]
tags: [design-twitter, heap, priority-queue, hashmap, design, medium, leetcode, leetcode-355]
leetcode_number: 355
leetcode_url: https://leetcode.com/problems/design-twitter/
difficulty: Medium
permalink: /posts/kotlin-dsa-heap-design-twitter/
hidden: true
---

## Problem Info

| | |
|---|---|
| **LeetCode #** | [355 — Design Twitter](https://leetcode.com/problems/design-twitter/) |
| **Difficulty** | Medium |
| **Topic** | Heap, Priority Queue, HashMap, Design |

> Design a simplified Twitter with:
> - `postTweet(userId, tweetId)` — composes a new tweet.
> - `getNewsFeed(userId)` — retrieves the **10 most recent** tweet IDs
>   from the user and everyone they follow, most recent first.
> - `follow(followerId, followeeId)` — follower follows followee.
> - `unfollow(followerId, followeeId)` — follower unfollows followee.

**Example:**
```
Input:
["Twitter","postTweet","getNewsFeed","follow","postTweet","getNewsFeed","unfollow","getNewsFeed"]
[[],[1,5],[1],[1,2],[2,6],[1],[1,2],[1]]

Output:
[null,null,[5],null,null,[6,5],null,[5]]

Explanation:
Twitter twitter = new Twitter();
twitter.postTweet(1, 5);     // user 1 posts tweet 5
twitter.getNewsFeed(1);      // [5] — just their own tweet
twitter.follow(1, 2);        // user 1 follows user 2
twitter.postTweet(2, 6);     // user 2 posts tweet 6
twitter.getNewsFeed(1);      // [6,5] — tweet 6 (user 2's) is more recent
twitter.unfollow(1, 2);
twitter.getNewsFeed(1);      // [5] — back to just their own tweet
```

**Constraints:**
- `1 <= userId, followerId, followeeId <= 500`
- `0 <= tweetId <= 10⁴`
- Each user posts at most 100 tweets total
- At most `3 * 10⁴` calls total across all methods

---

## Approach

This combines **HashMaps** (for the social graph and per-user tweet
history) with a **heap-based k-way merge** — the exact same idea as
Merge K Sorted Lists from the Linked List phase, just applied to tweet
timestamps instead of linked list nodes.

**Key insight:** Track a global, ever-increasing `timestamp` counter so
tweets can be ordered without relying on wall-clock time. Each user's
tweets are naturally stored in chronological order (since they're always
appended). To build a news feed, we need to merge the **k** relevant
tweet lists (the user's own + each followee's) and take the top 10 most
recent — exactly a k-way merge, solvable with a max-heap the same way as
Merge K Sorted Lists.

**Walk through the example above, focusing on `getNewsFeed(1)` after
following user 2:**
```
user 1's tweets: [(5, time=0)]
user 2's tweets: [(6, time=1)]

Push the most recent tweet from each relevant user onto a max-heap
(ordered by timestamp):
  heap: [(time=1, tweet=6, user=2), (time=0, tweet=5, user=1)]

Pop (time=1, tweet=6) → add 6 to result → no earlier tweet from user 2 to push
Pop (time=0, tweet=5) → add 5 to result → no earlier tweet from user 1 to push

Result: [6, 5] ✓ (most recent first)
```

---

## Kotlin Solution

### Approach 1 — HashMap of tweet lists + max-heap k-way merge (optimal)

```kotlin
class Twitter {
    private var timestamp = 0
    private val tweets = HashMap<Int, MutableList<Pair<Int, Int>>>()   // userId -> [(time, tweetId)]
    private val following = HashMap<Int, MutableSet<Int>>()            // userId -> set of followeeIds

    fun postTweet(userId: Int, tweetId: Int) {
        tweets.getOrPut(userId) { mutableListOf() }.add(timestamp to tweetId)
        timestamp++
    }

    fun getNewsFeed(userId: Int): List<Int> {
        val maxHeap = PriorityQueue<IntArray>(compareByDescending { it[0] })  // [time, tweetId, listIndex, userIdx]

        val relevantUsers = (following[userId] ?: emptySet()) + userId

        // For each relevant user, push their MOST RECENT tweet (last in their list)
        val indices = HashMap<Int, Int>()
        for (uid in relevantUsers) {
            val userTweets = tweets[uid] ?: continue
            if (userTweets.isEmpty()) continue
            val idx = userTweets.lastIndex
            indices[uid] = idx
            maxHeap.add(intArrayOf(userTweets[idx].first, userTweets[idx].second, uid))
        }

        val result = mutableListOf<Int>()

        while (maxHeap.isNotEmpty() && result.size < 10) {
            val (time, tweetId, uid) = maxHeap.poll()
            result.add(tweetId)

            val nextIdx = indices[uid]!! - 1
            if (nextIdx >= 0) {
                indices[uid] = nextIdx
                val userTweets = tweets[uid]!!
                maxHeap.add(intArrayOf(userTweets[nextIdx].first, userTweets[nextIdx].second, uid))
            }
        }

        return result
    }

    fun follow(followerId: Int, followeeId: Int) {
        if (followerId == followeeId) return
        following.getOrPut(followerId) { mutableSetOf() }.add(followeeId)
    }

    fun unfollow(followerId: Int, followeeId: Int) {
        following[followerId]?.remove(followeeId)
    }
}
```

### Approach 2 — Collect all relevant tweets, sort, take top 10 (simpler)

```kotlin
class Twitter {
    private var timestamp = 0
    private val tweets = HashMap<Int, MutableList<Pair<Int, Int>>>()
    private val following = HashMap<Int, MutableSet<Int>>()

    fun postTweet(userId: Int, tweetId: Int) {
        tweets.getOrPut(userId) { mutableListOf() }.add(timestamp to tweetId)
        timestamp++
    }

    fun getNewsFeed(userId: Int): List<Int> {
        val relevantUsers = (following[userId] ?: emptySet()) + userId
        val allTweets = mutableListOf<Pair<Int, Int>>()

        for (uid in relevantUsers) {
            tweets[uid]?.let { allTweets.addAll(it) }
        }

        return allTweets
            .sortedByDescending { it.first }
            .take(10)
            .map { it.second }
    }

    fun follow(followerId: Int, followeeId: Int) {
        if (followerId == followeeId) return
        following.getOrPut(followerId) { mutableSetOf() }.add(followeeId)
    }

    fun unfollow(followerId: Int, followeeId: Int) {
        following[followerId]?.remove(followeeId)
    }
}
```

Simpler to write and reason about — gathers every tweet from every
relevant user, then sorts the entire combined list. Given the problem's
small constraints (≤100 tweets per user, ≤500 users), this is more than
fast enough in practice, even though the heap approach is asymptotically
better when each user follows many people with long tweet histories.

---

## Why This Reuses the Merge K Sorted Lists Pattern

Each user's tweet list is already internally sorted by time (tweets are
appended in chronological order). Fetching a news feed is exactly "merge
k sorted lists and take the top 10" — the same shape of problem as
Merge K Sorted Lists, just bounded to the first 10 results instead of
fully merging everything:

```kotlin
while (maxHeap.isNotEmpty() && result.size < 10) {
    val (time, tweetId, uid) = maxHeap.poll()
    result.add(tweetId)
    // push that same user's NEXT most recent tweet, if any — exactly
    // like pushing a linked list node's "next" back onto the heap
}
```

**Why we push tweets in reverse (most recent first) per user:** since we
want the news feed in descending time order, walking each user's tweet
list from the end backward (most recent to oldest) means the heap always
offers the next-most-recent candidate from any given user, without
needing to look ahead.

---

## When to Use Which Approach

| Approach | Use When |
|---|---|
| Max-heap k-way merge (Approach 1) | Want the asymptotically optimal approach, mirrors real-world feed generation systems |
| Collect-and-sort (Approach 2) | Simpler to implement correctly, fine given this problem's small constraints |

---

## Complexity

| | postTweet | getNewsFeed | follow/unfollow |
|---|---|---|---|
| **Time (Heap)** | O(1) | O(k log k) — k = number of followees, bounded heap work | O(1) |
| **Time (Sort)** | O(1) | O(m log m) — m = total relevant tweets | O(1) |

---

## Key Takeaway

> "Merge several already-sorted sequences and take the top results" is a
> recurring shape across very different-looking problems — Merge K
> Sorted Lists' linked-list nodes and Design Twitter's per-user tweet
> timelines are solved by the exact same heap-based k-way merge
> technique. Recognizing when a new problem is secretly this same merge
> pattern, just with a different underlying data source, is one of the
> most valuable transfer skills built throughout this Heap phase.

🔗 [Solve it on LeetCode →](https://leetcode.com/problems/design-twitter/)

---

{% include kotlin-dsa-nav.html %}
