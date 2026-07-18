---
title: "Q22: Storing Media Files With Unpredictable Access Patterns at Minimum Cost"
date: 2026-07-04 08:05:00 +0530
categories: [AWS SAA, Design Cost-Optimized Architectures]
tags: [aws, s3, intelligent-tiering, saa-c03, storage, cost-optimization]
description: "A digital media app's files must survive the loss of an Availability Zone, with some files accessed frequently and others rarely in an unpredictable pattern, minimizing storage and retrieval costs."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Cost-Optimized Architectures |
| **Services** | Amazon S3, S3 Intelligent-Tiering |
| **Difficulty** | Easy–Medium |

## Question

A digital media app stores files in S3. Files must be **resilient to the loss of an Availability Zone**. Some files are accessed frequently; others are rarely accessed in an **unpredictable pattern**. The architect must **minimize storage and retrieval costs**.

Which storage option meets these requirements?

## Correct Answer

**S3 Intelligent-Tiering.**

## Why this is correct

The phrase **"unpredictable access pattern"** is the giveaway here — it's the exact scenario S3 Intelligent-Tiering was built to solve.

Every S3 storage class (Standard, Standard-IA, Intelligent-Tiering, One Zone-IA, Glacier tiers) that spans **multiple Availability Zones** by default satisfies "resilient to the loss of an AZ" — that rules out One Zone-IA immediately, but leaves several candidates. What separates Intelligent-Tiering is its **automatic** cost optimization: it monitors each object's access pattern and automatically moves objects between a **frequent access tier** (priced like S3 Standard) and an **infrequent access tier** (priced lower), with **no retrieval fees** for moving between these tiers, and no operational effort required to classify files yourself.

Because the files in this scenario are accessed unpredictably — sometimes frequently, sometimes not, with no clear schedule — manually choosing between Standard and Standard-IA (or writing custom Lifecycle rules based on a fixed number of days) risks either overpaying for Standard when a file goes cold, or incurring early-retrieval penalty fees on Standard-IA if a "cold" file is accessed sooner than expected. Intelligent-Tiering removes that guesswork and those retrieval-fee risks entirely.

## Why the alternatives fall short

- **S3 Standard for everything** — resilient to AZ loss, but you pay the highest per-GB rate even for files that go untouched for months — fails to minimize cost.
- **S3 Standard-IA with a Lifecycle policy based on a fixed access window** — cheaper for genuinely infrequent files, but Standard-IA charges a **retrieval fee** every time a file is read; with an *unpredictable* pattern, some "infrequent" files may be reread often, generating unexpected retrieval costs. Lifecycle rules also assume a known time-based pattern, which doesn't fit "unpredictable."
- **S3 One Zone-IA** — cheaper than Standard-IA, but stores data in only a **single Availability Zone**, directly violating "resilient to the loss of an Availability Zone."

## Exam Tip

**"Unpredictable / unknown / changing access patterns" + "minimize cost" → S3 Intelligent-Tiering**, almost without exception on this exam. Reserve **Lifecycle policies to Standard-IA/Glacier tiers** for scenarios where the access pattern is **known and predictable** (e.g., "accessed frequently for 30 days, then rarely" — see Q23), and always remember **One Zone-IA sacrifices AZ resilience** for a lower price.
