---
title: "Q2: Least-Overhead Way to Run Ad-Hoc SQL on JSON Logs in S3"
date: 2026-07-02 08:05:00 +0530
categories: [AWS SAA, Design High-Performing Architectures]
tags: [aws, athena, s3, saa-c03, analytics]
description: "A company needs to run simple, on-demand SQL queries against JSON application logs stored in S3 with the least operational overhead."
hidden: true
---

## Problem Info

| | |
|---|---|
| **Domain** | Design High-Performing Architectures |
| **Services** | Amazon Athena, Amazon S3 |
| **Difficulty** | Easy |

## Question

A company needs to analyze the log files of its proprietary application. The logs are stored in **JSON format** in an Amazon S3 bucket. Queries will be **simple** and run **on-demand**. A solutions architect must perform the analysis with **minimal changes to the existing architecture** and the **least operational overhead**.

What should the solutions architect do?

## Correct Answer

**Use Amazon Athena directly against the S3 bucket to run the queries as needed.**

## Why this is correct

The two loudest signals in this question are **"on-demand / simple queries"** and **"least operational overhead."** That combination almost always points to **Amazon Athena** on the exam.

Athena is a **serverless, interactive query service** that runs standard SQL directly against data sitting in S3 — including JSON. There's no cluster to provision, patch, or scale, and no data movement required: you point Athena at the bucket (optionally defining a table via the Glue Data Catalog or an inline schema), and you're querying within minutes. You pay only for the data scanned per query, which fits "on-demand" perfectly — there's no always-on infrastructure sitting idle between queries.

Because the logs stay exactly where they are in S3, this also satisfies **"minimal changes to the existing architecture."**

## Why the alternatives fall short

- **Amazon Redshift / Redshift Spectrum** — powerful, but provisioning a data warehouse cluster (even Spectrum needs a Redshift cluster) is significant operational overhead for "simple, on-demand" queries. Redshift shines for complex, high-concurrency BI workloads, not ad-hoc log checks.
- **Amazon EMR with Hive/Presto** — requires standing up and managing a Hadoop cluster. Far too much operational weight for occasional simple queries.
- **Loading logs into a relational database (RDS) first** — adds an ETL pipeline and a new piece of infrastructure to maintain, directly violating "minimal changes" and "least overhead."

## Exam Tip

Pattern-match this phrase combo instantly: **"data already in S3" + "SQL" + "ad-hoc / on-demand" + "least operational overhead" → Amazon Athena.** If the question instead emphasizes **recurring, complex joins across many large datasets for dashboards/BI**, lean toward **Redshift**. If it mentions **custom big-data processing frameworks (Spark, Hive)**, think **EMR**.
