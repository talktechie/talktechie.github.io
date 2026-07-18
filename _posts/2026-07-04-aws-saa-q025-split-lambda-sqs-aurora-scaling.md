---
title: "Q25: Fixing a Lambda Function That Keeps Hitting Its Quotas Under Load"
date: 2026-07-04 08:20:00 +0530
categories: [AWS SAA, Design High-Performing Architectures]
tags: [aws, lambda, sqs, api-gateway, aurora, saa-c03, scalability]
description: "A Lambda function receiving data via API Gateway and writing to Aurora PostgreSQL needed significantly increased quotas during proof-of-concept. The architect needs better scalability with minimal configuration effort."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design High-Performing Architectures |
| **Services** | AWS Lambda, Amazon SQS, API Gateway, Aurora PostgreSQL |
| **Difficulty** | Medium |

## Question

An app uses a **Lambda function** to receive data via **API Gateway** and store it in **Aurora PostgreSQL**. During proof-of-concept, the company had to **significantly increase Lambda quotas** to handle high data volumes. A solutions architect must improve **scalability** and **minimize configuration effort**.

Which solution meets these requirements?

## Correct Answer

**Split the work into two Lambda functions — one to receive the information, one to load it into the database — integrated via an SQS queue.**

## Why this is correct

The root problem: a **single Lambda function doing both intake and database writes** is bottlenecked by the *slower* of the two operations. Aurora writes have finite throughput and connection limits, but incoming requests via API Gateway can burst far faster than the database can absorb writes — forcing high Lambda concurrency (hence needing quota increases) just to keep up with intake, while many of those concurrent invocations sit blocked waiting on database connections.

**Splitting into two functions connected by an SQS queue** decouples the two rates:

- **Receiver Lambda**: triggered by API Gateway, does almost no work beyond validating and dropping the message onto an SQS queue — returns fast, scales easily with incoming request bursts, and needs comparatively few concurrent executions.
- **Loader Lambda**: triggered by the SQS queue at a **controlled, database-appropriate pace** (Lambda's SQS event source scales consumption based on queue depth, but you can also tune batch size and reserved concurrency to match what Aurora can handle), writing to Aurora without being flooded by intake spikes.

This requires minimal configuration — just an SQS queue and a second, smaller Lambda function — rather than negotiating ever-higher Lambda quotas or redesigning the database layer.

## Why the alternatives fall short

- **Keep raising Lambda concurrency quotas as volume grows** — treats the symptom, not the cause; you'll keep needing quota increases as traffic grows, and it doesn't fix the underlying bottleneck of Aurora connection/write limits — more operational effort over time, not less.
- **Add RDS Proxy in front of Aurora, keep one Lambda function** — RDS Proxy helps manage database connection pooling efficiently and is a legitimate scaling aid, but by itself it doesn't decouple the *rate* of intake from the *rate* of processing the way an SQS buffer does, and the single function is still handling both concerns.
- **Switch to a larger Aurora instance class to handle more concurrent writes** — vertical scaling raises the ceiling somewhat, but doesn't fundamentally decouple bursty intake from steady processing, and is a more expensive lever than simply buffering with SQS.

## Exam Tip

**"One Lambda doing intake + processing" + "had to increase Lambda quotas" + "improve scalability, minimal configuration" → split into two functions with SQS as the buffer between them.** This is the serverless equivalent of the classic "queue between producer and consumer" pattern (see Q7, Q8, Q18) — recognizing it in a Lambda-specific costume is a recurring exam skill.
