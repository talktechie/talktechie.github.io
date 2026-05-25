---
layout: post
title: "I Wrote Raw SQL for Years. Then JPA Surprised Me."
date: 2026-05-25 00:00:00 +0000
# categories: [Spring Data JPA, Kotlin]
# tags: [jpa, kotlin, spring-boot, n+1, hibernate]
author: talktechie
---

For most of my career, I wrote raw SQL. Not because I was stubborn about it — I just understood it. I knew exactly what was hitting the database. I could read the query, predict the result, and tune it when something was slow. There was no mystery.

Then I moved to a project using Spring Data JPA.

And I discovered that when you hand query control to a framework, the surprises come fast — especially when you *think* you already understand what's happening.

This is the first thing that caught me off guard.

---

## The Setup (Table Names Used Here)

I had an `Orders` entity with a `@ManyToOne` relationship to a `Customer`. I needed the customer name whenever I fetched orders. In raw SQL, I'd just write a JOIN. Simple.

But in JPA, I'd read that `FetchType.EAGER` is how you tell the framework to load associated data. So I used it:

```kotlin
@Entity
class Orders(
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0,

    val amount: Double,

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "customer_id")
    val customer: Customer
)

@Entity
class Customer(
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0,

    val name: String
)
```

Then I wrote a repository query and looped over the results:

```kotlin
interface OrderRepository : JpaRepository<Orders, Long> {
    @Query("select o from Orders o")
    fun findAllOrders(): List<Orders>
}
```

```kotlin
fun printAllOrders() {
    val orders = orderRepository.findAllOrders()
    for (order in orders) {
        println("Order #${order.id} — Customer: ${order.customer.name}")
    }
}
```

In my head, this was basically:

```sql
SELECT o.*, c.* FROM orders o JOIN customer c ON o.customer_id = c.id
```

I enabled SQL logging and ran it.

---

## What Actually Hit the Database


![N+1 vs JOIN FETCH — how queries fire differently]({{ "/assets/img/n-plus-one-diagram.svg" | relative_url }})


```sql
-- One query for all the orders
SELECT o.id, o.amount, o.customer_id FROM orders o

-- Then one query per order, for the customer
SELECT c.id, c.name FROM customer c WHERE c.id = 1
SELECT c.id, c.name FROM customer c WHERE c.id = 2
SELECT c.id, c.name FROM customer c WHERE c.id = 3
-- ... one for every single row
```

I had 10 orders. I got 11 queries.

Coming from raw SQL, this felt wrong in my gut immediately. I would *never* write this. No sane developer queries inside a loop. But JPA did it for me — quietly, automatically — and I hadn't noticed until I looked at the logs.

This is the N+1 problem. One query to get N rows, then N more queries to get the related data. And I had just invited it in by trusting that `EAGER` meant what I thought it meant.

---

## The Assumption I Got Wrong

In raw SQL, if I want related data, I write a JOIN. That's the contract — explicit, readable, predictable.

I assumed `FetchType.EAGER` was JPA's version of that. Load the customer data. Now. With the order. Like a JOIN would.

It isn't.

`FetchType.EAGER` only controls *when* the data is fetched — immediately, not when you first touch it. It says nothing about *how*. And when you don't say how, JPA defaults to separate queries. One per row. Right away. Eagerly. But separately.

This is the distinction that raw SQL never taught me, because in raw SQL it doesn't exist. You either JOIN or you don't. There's no timing vs. strategy split to think about.

| Concern | Controlled by |
|---|---|
| **When** to load the association | `FetchType` — EAGER loads immediately, LAZY loads on access |
| **How** to load it (JOIN vs. separate queries) | Your JPQL query |

JPA separates these two concerns. I had only solved for one of them.

---

## The Fix

Once you understand the distinction, the fix is obvious. You tell JPA explicitly to JOIN:

```kotlin
interface OrderRepository : JpaRepository<Orders, Long> {
    @Query("select o from Orders o join fetch o.customer")
    fun findAllOrders(): List<Orders>
}
```

`join fetch` is JPQL — not plain SQL, but close enough to read naturally. It tells Hibernate: don't run a separate query for the customer. Bring it with this one, in a JOIN.

The resulting SQL:

```sql
SELECT o.id, o.amount, o.customer_id, c.id, c.name
FROM orders o
INNER JOIN customer c ON o.customer_id = c.id
```

One query. Exactly what I would have written by hand.

---

## What This Taught Me About JPA

Coming from raw SQL, I expected JPA to be an abstraction over the queries I already knew. And it is — but the abstraction has its own vocabulary, its own mental model, and its own failure modes.

`FetchType` is not about JOINs. It's about lazy vs. eager initialization. The query is still your responsibility.

When I wrote `@Query("select o from Orders o")`, I gave JPA a complete query with no JOIN in it. JPA executed that query faithfully. Then it saw `EAGER` and dutifully fetched the customers — separately, per row, just like the spec allows.

It did exactly what I told it to. I just didn't realize what I was telling it.

The raw SQL habit that would've saved me: *if you need related data, you write the JOIN*. That instinct still applies in JPA. It just lives in JPQL now, with `join fetch` instead of `JOIN`.

---

## The Rule I'm Keeping

> Write a `@Query`? Check every associated entity you'll access.  
> Need that data? Add `join fetch`. Don't trust `FetchType` to handle it.

It's one extra phrase in the query. Two words. But without them, you're quietly paying N database round trips every time that method runs.

In raw SQL, this kind of thing is visible by nature — you wrote the query, you see what it does. In JPA, you have to go looking for it. Turn on SQL logging, at least in development. Read what's actually hitting your database.

The framework isn't hiding things from you. It's doing what you said. You just have to learn to speak its language precisely.
