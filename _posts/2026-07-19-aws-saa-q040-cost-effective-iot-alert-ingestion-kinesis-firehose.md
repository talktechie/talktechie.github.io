---
title: "Q40: Ingest and Archive IoT Alerts Cheaply with Kinesis Firehose"
date: 2026-07-19 08:20 +0530
categories: [AWS SAA, Design Cost-Optimized Architectures]
tags: [aws, kinesis-firehose, s3, glacier, saa-c03]
description: "Thousands of edge devices generate 1 TB of daily alerts that must be ingested, kept 14 days for immediate analysis, then archived cheaply with minimal management."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Cost-Optimized Architectures |
| **Services** | Kinesis Data Firehose, Amazon S3, Amazon S3 Glacier |
| **Difficulty** | Easy–Medium |

## Question

A company has thousands of edge devices that collectively generate 1 TB of status alerts each day (roughly 2 KB per alert). A solutions architect must ingest and store the alerts for future analysis. The company wants a highly available solution that minimizes cost and avoids managing additional infrastructure. It needs 14 days of data available for immediate analysis, with anything older archived. What is the most operationally efficient solution?

## Correct Answer

**Use Amazon Kinesis Data Firehose to ingest the alerts and deliver them to an S3 bucket, then apply an S3 Lifecycle rule to transition data older than 14 days to S3 Glacier.**

## Why this is correct

Kinesis Data Firehose is a fully managed delivery service — there are no clusters or consumers to provision or scale, and it automatically batches, compresses, and writes incoming records to a destination like S3. That directly satisfies "no additional infrastructure to manage" while comfortably handling 1 TB/day of small (2 KB) alert records from thousands of devices.

S3 provides highly available, durable storage for the 14-day "immediate analysis" window at low per-GB cost, and an S3 Lifecycle configuration automates the transition of anything older than 14 days into S3 Glacier — a much cheaper long-term archive tier — with zero ongoing operational effort once the rule is set.

Together, Firehose-to-S3 plus a Lifecycle rule to Glacier is a fully serverless, self-managing pipeline: ingestion scales automatically, storage tiers itself automatically by age, and nothing needs to be patched, sized, or babysat.

## Why the alternatives fall short

- **Kinesis Data Streams with a custom consumer application** — works, but requires building, scaling, and operating consumer logic yourself, adding exactly the infrastructure management the requirement asks to avoid.
- **Writing alerts directly into DynamoDB** — becomes costly at 1 TB/day of continuous small-item writes and doesn't offer the same built-in, low-cost tiered archival path that S3 Lifecycle + Glacier provides.
- **A self-managed EC2-based ingestion pipeline** — directly conflicts with "does not want to manage additional infrastructure" and adds unnecessary operational and cost burden for a workload Firehose handles natively.

## Exam Tip

"Ingest streaming data" + "fully managed, no infrastructure" + "keep recent data, archive the rest cheaply" is the classic signature for **Kinesis Data Firehose → S3 → Lifecycle policy to Glacier**.
