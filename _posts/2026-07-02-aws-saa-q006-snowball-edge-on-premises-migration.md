---
title: "Q6: Migrating 70 TB of NFS Video Files to S3 With Minimal Bandwidth"
date: 2026-07-02 08:25:00 +0530
categories: [AWS SAA, Design Cost-Optimized Architectures]
tags: [aws, snowball, s3, migration, saa-c03, storage]
description: "A company's on-premises NFS storage holds 70 TB of large video files that must move to S3 as fast as possible while using the least network bandwidth."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Cost-Optimized Architectures |
| **Services** | AWS Snowball Edge, Amazon S3 |
| **Difficulty** | Easy–Medium |

## Question

A company stores large video files (1 MB to 500 GB each) on on-premises NFS storage. Total data is **70 TB and no longer growing**. The company wants to migrate everything to S3 **as soon as possible**, while using the **least possible network bandwidth**.

Which solution meets these requirements?

## Correct Answer

**Order an AWS Snowball Edge device. Receive it on-premises, use the Snowball Edge client to copy the data locally, then ship the device back so AWS can import the data into S3.**

## Why this is correct

Two requirements pull in what looks like opposite directions — "as soon as possible" and "least network bandwidth" — and Snowball Edge is the one AWS service built to satisfy both simultaneously for a **large, one-time, finite** dataset.

Because the 70 TB is **static (not growing)**, this is a one-shot bulk migration, not an ongoing feed — exactly Snowball's use case. Data transfer to the device happens over a local, high-speed connection (up to ~100 Gbps), so copying 70 TB takes roughly the low thousands of seconds — well under a couple of hours. Zero bytes cross the internet or corporate WAN link during that copy: **bandwidth usage is effectively zero.** The only "slow" part is physical shipping (a handful of business days each way), which is unavoidable for physically moving a device, but is still far faster and cheaper than saturating a corporate internet link for days or weeks trying to push 70 TB over the wire.

## Why the alternatives fall short

- **S3 Transfer Acceleration over the existing internet connection** — great for ongoing, moderate-sized transfers (see Q1), but pushing 70 TB over any internet link — even an accelerated one — consumes significant bandwidth and could take a long time depending on available uplink speed, directly violating "least possible network bandwidth."
- **AWS DataSync** — efficient for online, incremental, or ongoing NFS-to-S3 transfers, but it still moves data across the network. For a large one-time static dataset with a bandwidth constraint, it's not the leanest choice compared to physically shipping a device.
- **Set up Direct Connect** — solves ongoing high-throughput, low-latency connectivity, but provisioning it takes weeks and is overkill (and costly) for a single 70 TB one-time migration.

## Exam Tip

Pattern to memorize: **large + one-time + limited/no bandwidth + on-premises → Snowball family.** Scale further with data size:
- **Snowcone** — tens of terabytes, small/rugged form factor.
- **Snowball Edge** — up to ~80 TB usable per device (as in this question).
- **Snowmobile** — exabyte-scale, literally a shipping container on a truck.

If the dataset is **actively growing / needs to sync continuously**, that's a signal for **AWS DataSync** instead, not Snowball.
