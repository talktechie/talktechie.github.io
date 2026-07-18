---
title: "Q14: Automatically Scaling a Read-Heavy Database While Keeping High Availability"
date: 2026-07-03 08:15:00 +0530
categories: [AWS SAA, Design High-Performing Architectures]
tags: [aws, aurora, rds, auto-scaling, saa-c03, database]
description: "A MySQL database on a single large EC2 instance degrades under load. The app is read-heavy and needs automatic scaling for unpredictable reads plus high availability."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design High-Performing Architectures |
| **Services** | Amazon Aurora, Aurora Auto Scaling, Aurora Replicas |
| **Difficulty** | Medium |

## Question

An ecommerce app runs on EC2 instances in an Auto Scaling group behind an ALB, storing transaction data in **MySQL 8.0 on a large EC2 instance**. Performance degrades quickly as load increases, and the app handles **far more reads than writes**. The company wants the database to **automatically scale to meet unpredictable read demand** while **maintaining high availability**.

Which solution meets these requirements?

## Correct Answer

**Migrate to Amazon Aurora with a Multi-AZ deployment. Configure Aurora Auto Scaling with Aurora Replicas.**

## Why this is correct

Running MySQL yourself on a single EC2 instance means you own all the scaling and failover work manually — and a single instance has a hard ceiling on read capacity no matter how large you make it.

**Amazon Aurora** (MySQL-compatible, so migration is straightforward) decouples storage from compute and supports up to **15 low-latency read replicas** sharing the same underlying storage volume. **Aurora Auto Scaling** monitors a target metric (typically average CPU or Aurora Replica connections) on the replicas and automatically **adds or removes replicas** as read load fluctuates — a direct match for "unpredictable read workloads."

A **Multi-AZ** Aurora deployment keeps at least one replica in a different Availability Zone, and Aurora automatically fails over to a replica within ~30 seconds if the primary writer instance fails — satisfying "maintaining high availability" without any manual intervention.

## Why the alternatives fall short

- **Vertically scale the existing EC2-hosted MySQL instance to a larger instance type** — helps temporarily, but it's manual, has a hard ceiling, doesn't distribute read load, and provides no automatic failover — doesn't meet "automatically scale" or robustly meet "high availability."
- **Add a single MySQL read replica manually on another EC2 instance** — improves read capacity somewhat, but "manually add one replica" is neither automatic nor elastic to *unpredictable* demand, and self-managed MySQL replication adds significant operational burden.
- **Use RDS for MySQL with Multi-AZ (not Aurora) and manually created read replicas** — RDS for MySQL Multi-AZ provides a standby for failover but that standby **isn't used to serve read traffic**; you'd have to create and manage read replicas yourself, and RDS for MySQL doesn't offer the automatic replica scaling that Aurora Auto Scaling provides.

## Exam Tip

**"MySQL/PostgreSQL-compatible" + "read-heavy, unpredictable" + "scale automatically" + "high availability" → Amazon Aurora with Aurora Auto Scaling + Multi-AZ.** This is one of the most frequently tested database patterns on the SAA exam — Aurora's separation of storage and compute, fast replica promotion, and native auto-scaling of replicas consistently beat plain RDS or self-managed EC2 databases whenever elasticity and HA are both required.
