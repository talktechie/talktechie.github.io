---
title: "Q16: Visualizing a Data Lake With Tiered Access for Management vs Everyone Else"
date: 2026-07-03 08:25:00 +0530
categories: [AWS SAA, Design Secure Architectures]
tags: [aws, quicksight, s3, rds, saa-c03, analytics, security]
description: "A data lake spans S3 and RDS for PostgreSQL. The company needs data visualization across all sources, with management getting full access and everyone else limited access."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Secure Architectures |
| **Services** | Amazon QuickSight, Amazon S3, Amazon RDS for PostgreSQL |
| **Difficulty** | Easy–Medium |

## Question

A data lake spans **S3** and **RDS for PostgreSQL**. The company needs a reporting solution with **data visualization** that includes all data sources. **Only management should have full access** to all visualizations; everyone else gets **limited access**.

Which solution meets these requirements?

## Correct Answer

**Create an analysis in Amazon QuickSight. Connect all data sources and build datasets. Publish dashboards, and share them with the appropriate specific users and groups.**

## Why this is correct

**Amazon QuickSight** is AWS's managed business intelligence and visualization service, and it directly supports both halves of this requirement.

First, connecting sources: QuickSight can connect natively to both **Amazon S3** (including via Athena for querying) and **Amazon RDS for PostgreSQL**, letting you build unified datasets and dashboards that blend data from across the data lake — satisfying "includes all the data sources."

Second, and just as important for this question, is the access control model: published QuickSight **dashboards** can be **shared with specific IAM users or QuickSight groups**, with fine-grained control over who sees what. You can grant the management group full access to every dashboard, while sharing a restricted subset (or a filtered view via row-level security) with everyone else — matching "management gets full access, the rest gets limited access" without building a separate access-control system.

## Why the alternatives fall short

- **Export data to spreadsheets and share via S3 with bucket policies controlling access** — technically enforces access tiers, but loses interactive visualization entirely and requires manual export/refresh cycles — poor fit for "data visualization," and not scalable.
- **Build a custom web dashboard on EC2 that queries S3 and RDS directly** — could visualize the data, but you'd be building and maintaining your own authentication/authorization layer, database connectors, and rendering — far more operational overhead than a managed BI tool built for exactly this.
- **Use Amazon Athena alone for querying, with IAM policies restricting query access per user** — Athena provides SQL query access, not interactive dashboards/visualizations; it doesn't natively address "data visualization," which is the primary requirement here.

## Exam Tip

**"Data visualization" + "multiple data sources (S3, RDS, Redshift, etc.)" + "different access levels for different user groups" → Amazon QuickSight.** Remember QuickSight's access control operates at the level of **dashboards shared with users/groups**, and can go further with **row-level security** for restricting specific data within a shared dashboard — useful when a question asks for partial rather than all-or-nothing visibility.
