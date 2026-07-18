---
title: "Q21: A One-Deal-a-Day Site That Handles Millions of Requests With Millisecond Latency"
date: 2026-07-04 08:00:00 +0530
categories: [AWS SAA, Design High-Performing Architectures]
tags: [aws, s3, cloudfront, api-gateway, lambda, dynamodb, saa-c03, serverless]
description: "An ecommerce flash-sale site featuring one product per 24-hour window needs to handle millions of requests per hour with millisecond latency, at the lowest possible operational overhead."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design High-Performing Architectures |
| **Services** | Amazon S3, CloudFront, API Gateway, Lambda, DynamoDB |
| **Difficulty** | Medium |

## Question

An ecommerce company is launching a **one-deal-a-day** website — one product on sale every 24 hours. It must handle **millions of requests per hour** with **millisecond latency** during peak traffic.

Which solution meets these requirements with the **least operational overhead**?

## Correct Answer

**Host static content in an S3 bucket, front it with a CloudFront distribution (S3 as origin). Build backend APIs with API Gateway + Lambda. Store data in DynamoDB.**

## Why this is correct

This is the SAA exam's canonical **fully serverless, fully managed** stack, and every layer is chosen specifically to avoid capacity planning:

- **S3 + CloudFront**: the marketing page, images, and other static assets are served from CloudFront's edge caches — capable of absorbing massive concurrent read traffic with millisecond edge latency, without provisioning a single server.
- **API Gateway + Lambda**: the backend (e.g., "buy now," inventory checks) scales automatically with incoming request volume — Lambda spins up as many concurrent executions as needed for a burst, then scales back to zero when idle. No fleet of application servers to size or Auto Scale manually.
- **DynamoDB**: a fully managed, single-digit-millisecond NoSQL database that scales throughput automatically (with on-demand capacity mode), matching the unpredictable, spiky traffic pattern of a flash-sale product going live.

Every component here is **serverless and auto-scaling by default**, which is exactly what "least operational overhead" is testing for — no instances, no capacity planning, no patching.

## Why the alternatives fall short

- **EC2 instances in an Auto Scaling group behind an ALB, with RDS as the database** — can technically scale, but Auto Scaling reacts to metrics with some lag, isn't instantaneous like Lambda's concurrency model, and requires ongoing instance/AMI/patch management — real operational overhead compared to a fully serverless stack.
- **Amazon ElastiCache in front of RDS for read scaling** — helps read performance for a relational database, but doesn't address the core requirement of the backend API layer scaling itself, and still leaves you managing RDS instance sizing and Multi-AZ failover manually.
- **Host the entire site on EC2 with pre-warmed instances for the expected traffic spike** — "pre-warming" implies manual capacity prediction ahead of each day's sale, which is itself significant operational effort, and risks under- or over-provisioning if the actual spike differs from the forecast.

## Exam Tip

**"Millions of requests, unpredictable/spiky traffic" + "millisecond latency" + "least operational overhead" → the fully serverless stack: S3 + CloudFront (static) + API Gateway + Lambda (compute) + DynamoDB (data).** This exact five-service combination is one of the most frequently recurring "correct answer shapes" across the entire SAA-C03 exam — recognize it on sight.
