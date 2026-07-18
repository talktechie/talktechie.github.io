---
title: "Q23: Keeping Backup Files Forever at the Lowest Possible Cost"
date: 2026-07-04 08:10:00 +0530
categories: [AWS SAA, Design Cost-Optimized Architectures]
tags: [aws, s3, glacier-deep-archive, lifecycle-policy, saa-c03, storage]
description: "Backup files are accessed frequently for the first month, then never again, but must be kept indefinitely. The company wants the most cost-effective storage solution."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Cost-Optimized Architectures |
| **Services** | Amazon S3 Standard, S3 Glacier Deep Archive, S3 Lifecycle policies |
| **Difficulty** | Easy |

## Question

Backup files are stored in **S3 Standard**, accessed frequently for the **first month**, then **never accessed again** — but must be **kept indefinitely**.

Which storage solution meets this **most cost-effectively**?

## Correct Answer

**Create an S3 Lifecycle configuration to transition objects from S3 Standard to S3 Glacier Deep Archive after 1 month.**

## Why this is correct

This question has a clean, known access pattern — frequent for exactly 30 days, then permanently cold — which is precisely what an **S3 Lifecycle transition rule** is designed to automate.

**S3 Glacier Deep Archive** is AWS's **lowest-cost** storage class, built for data that is rarely (if ever) accessed again and can tolerate a retrieval time measured in hours. Since these backups are described as never accessed after month one, and must be retained **indefinitely**, Deep Archive's ultra-low per-GB storage price minimizes long-term cost far more than any "warmer" tier would, and the multi-hour retrieval time is an acceptable trade-off since the files aren't expected to be pulled back on demand.

Setting a **Lifecycle rule** to transition objects automatically at the 1-month mark means the files get full-speed **S3 Standard** access during the period they're actually needed, then drop to the cheapest possible storage the moment that access pattern ends — with zero manual intervention going forward.

## Why the alternatives fall short

- **Leave everything in S3 Standard indefinitely** — correct for accessibility, but by far the most expensive option for data that's never touched again after month one — fails "most cost-effective."
- **Transition to S3 Standard-IA after 1 month instead of Glacier Deep Archive** — cheaper than Standard, but Standard-IA is priced for data accessed **occasionally**, not data that's *never* accessed again. Since these backups have zero further access, Deep Archive's much lower storage price wins decisively for a "keep indefinitely, never read again" pattern.
- **Delete the files after 1 month to save cost** — violates the explicit requirement to **keep the files indefinitely**; not a valid option regardless of cost.

## Exam Tip

When you see a clear, described time-based transition ("accessed frequently for X, then never again, must be kept forever/long-term"), that's a **Lifecycle policy** question, and the final tier should match how "cold" the data truly is: **rarely accessed but might need it back reasonably fast → Standard-IA/Glacier Instant Retrieval**; **essentially never accessed again, retention/compliance driven, retrieval-time-tolerant → Glacier Deep Archive**, the cheapest class AWS offers.
