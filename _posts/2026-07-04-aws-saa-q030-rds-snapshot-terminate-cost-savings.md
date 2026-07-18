---
title: "Q30: Cutting Costs on a Database Used Only 48 Hours a Month"
date: 2026-07-04 08:45:00 +0530
categories: [AWS SAA, Design Cost-Optimized Architectures]
tags: [aws, rds, snapshots, saa-c03, cost-optimization, database]
description: "A development team runs resource-intensive tests once a month for 48 hours on an RDS for MySQL instance, and wants to cut costs without reducing the instance's compute and memory."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Cost-Optimized Architectures |
| **Services** | Amazon RDS, RDS Snapshots |
| **Difficulty** | Easy–Medium |

## Question

A team runs resource-intensive tests **once a month for 48 hours** on a general-purpose RDS for MySQL instance — the only workload on that database. The team wants to **reduce testing costs** **without reducing** the instance's compute/memory attributes.

Which solution is **most cost-effective**?

## Correct Answer

**Take a snapshot when testing finishes, then terminate (delete) the DB instance. Restore the snapshot into a new instance the next time testing is needed.**

## Why this is correct

The core insight: an RDS instance that's actively used only **48 hours out of roughly 720 hours in a month** (under 7% utilization) is being paid for at full price the other 93%+ of the time it's simply idle.

RDS billing is primarily **per running instance-hour**. Since RDS instances **cannot** be "stopped" indefinitely the way EC2 can for long, unattended periods without limitation (RDS auto-restarts a stopped instance after 7 days), and this database sits unused for weeks at a time, the most cost-effective move is to **not have a running instance at all** between test cycles.

Taking a final **snapshot** before terminating preserves 100% of the data and configuration — snapshots are billed at a low per-GB storage rate, dramatically cheaper than a running instance. When the next monthly test cycle arrives, **restoring the snapshot** recreates a new DB instance with the **exact same engine, data, and instance class** as before — satisfying "without reducing the compute and memory attributes," since you simply restore to the same instance type used previously.

## Why the alternatives fall short

- **Use RDS Reserved Instances to lower the hourly rate** — Reserved Instances give a discount for **committing to run continuously** over a 1- or 3-year term; that's the wrong pricing model for a database that's only needed 48 hours a month — you'd be pre-paying (or committing) for capacity sitting idle almost all the time.
- **Downsize the instance class between tests, then resize back up before testing** — resizing does reduce idle-time cost somewhat, but it **does** change compute/memory attributes during the idle period (and requires manual resize operations before/after every cycle), and still leaves an instance running (and billed) continuously — not as cost-effective as having zero running instance when idle.
- **Stop the RDS instance between test cycles** — RDS supports stopping an instance to pause compute billing, but AWS **automatically restarts** a stopped RDS instance after **7 consecutive days**, meaning it would auto-restart multiple times before the next monthly test — resuming compute charges unexpectedly and requiring the team to notice and re-stop it repeatedly. A snapshot-and-terminate cycle avoids this entirely.

## Exam Tip

**"Database (or workload) used only occasionally / a few days a month" + "reduce cost without changing compute/memory specs" → snapshot, then terminate/delete; restore from snapshot when needed again.** Remember RDS's **7-day auto-restart limitation** on stopped instances — it's a frequently tested detail that rules out "just stop it" as a valid answer for month-long idle periods.
