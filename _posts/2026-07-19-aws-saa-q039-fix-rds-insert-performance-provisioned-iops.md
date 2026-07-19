---
title: "Q39: Fix Slow RDS Inserts with Provisioned IOPS Storage"
date: 2026-07-19 08:15 +0530
categories: [AWS SAA, Design High-Performing Architectures]
tags: [aws, rds, ebs, storage-performance, saa-c03]
description: "An RDS for MySQL table with millions of daily updates is suffering slow inserts because of General Purpose SSD storage limits."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design High-Performing Architectures |
| **Services** | Amazon RDS, Provisioned IOPS SSD |
| **Difficulty** | Medium |

## Question

A company maintains a searchable repository in an RDS for MySQL table with more than 10 million rows, backed by 2 TB of General Purpose SSD storage. There are millions of updates against this data every day. Some insert operations are taking 10 seconds or longer, and the company has determined the database storage performance is the problem. Which solution addresses this performance issue?

## Correct Answer

**Change the RDS storage type from General Purpose SSD to Provisioned IOPS SSD.**

## Why this is correct

General Purpose SSD (gp2/gp3) storage ties its baseline IOPS to volume size and relies on a burst-credit model — under a sustained, heavy-write workload like millions of daily updates, those burst credits can deplete, causing IOPS (and therefore insert latency) to drop sharply. That's a strong match for the symptom described: insert operations occasionally taking 10+ seconds.

Provisioned IOPS SSD (io1/io2) storage is designed specifically for I/O-intensive, latency-sensitive workloads. It lets you provision a consistent, guaranteed level of IOPS independent of volume size, so insert-heavy OLTP workloads get predictable, low-latency performance instead of being throttled by burst-credit exhaustion.

Since the company has already isolated the bottleneck to storage performance (not CPU, memory, or query design), switching the storage type directly targets the root cause without requiring any application or schema changes.

## Why the alternatives fall short

- **Scaling up to a larger DB instance class** — increases CPU and memory, but does nothing for an I/O-bound storage bottleneck if the underlying volume can't sustain the required IOPS.
- **Adding a read replica** — offloads read traffic to a separate instance, but has no effect on write/insert latency on the primary, which is the actual problem here.
- **Simply increasing the gp2 volume size for more baseline IOPS** — can help indirectly since gp2 IOPS scale with size, but it's a less direct and less reliable fix than moving to Provisioned IOPS, which guarantees a specific IOPS level regardless of size.

## Exam Tip

Sustained heavy **write/insert** latency traced to **storage**, not compute, on RDS = switch to **Provisioned IOPS SSD** — don't reach for a bigger instance class or a read replica.
