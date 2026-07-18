---
title: "Q13: Rotating RDS Credentials Across Multiple Regions With the Least Overhead"
date: 2026-07-03 08:10:00 +0530
categories: [AWS SAA, Design Secure Architectures]
tags: [aws, secrets-manager, rds, multi-region, saa-c03, security]
description: "A company needs to rotate credentials for RDS for MySQL databases across multiple AWS Regions during monthly maintenance, with the least operational overhead."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Secure Architectures |
| **Services** | AWS Secrets Manager (multi-Region secrets), Amazon RDS for MySQL |
| **Difficulty** | Medium |

## Question

A company performs **monthly maintenance** on its AWS infrastructure and needs to rotate credentials for its **RDS for MySQL** databases across **multiple AWS Regions**.

Which solution meets this requirement with the **least operational overhead**?

## Correct Answer

**Store the credentials as secrets in AWS Secrets Manager. Turn on multi-Region secret replication for the required Regions. Configure Secrets Manager to rotate the secrets on a schedule.**

## Why this is correct

This is Q11's Secrets Manager pattern, extended across Regions — and Secrets Manager has a purpose-built feature for exactly that: **multi-Region secrets**.

Rather than creating and managing a separate secret (and separate rotation schedule) per Region, you create one secret and enable **replication** to the other required Regions. Secrets Manager keeps the replicas in sync automatically. Layer on **scheduled rotation**, and every Region's RDS credential gets rotated on the defined cadence (e.g., monthly, matching the maintenance window) without anyone touching each Region by hand.

This satisfies "least operational overhead" precisely because there's no per-Region manual process — one configuration, replicated and rotated automatically everywhere it's needed.

## Why the alternatives fall short

- **Manually rotate credentials in each Region's RDS instance during the maintenance window** — exactly the manual, repetitive, error-prone process the question is trying to eliminate; doesn't scale as Regions or databases grow.
- **Create a separate secret per Region in Secrets Manager with independent rotation schedules** — an improvement over fully manual rotation, but still means managing N separate secrets and N rotation configurations, more overhead than one replicated secret.
- **Use Systems Manager Parameter Store with a custom Lambda rotation function per Region** — Parameter Store has no native cross-Region replication or built-in rotation the way Secrets Manager does; you'd be building and maintaining custom automation per Region, the opposite of "least overhead."

## Exam Tip

Whenever a Secrets Manager question adds **"multiple Regions"** to the mix, look specifically for **multi-Region secret replication** as part of the answer — it's a distinct, exam-testable feature from basic single-Region rotation (as in Q11). The pairing "replicate the secret + rotate on schedule" is the signature of the correct answer for cross-Region credential management questions.
