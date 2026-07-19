---
title: "Q33: Stream Millions of Transactions in Near Real Time with Kinesis"
date: 2026-07-19 08:45 +0530
categories: [AWS SAA, Design High-Performing Architectures]
tags: [aws, kinesis, lambda, dynamodb, saa-c03]
description: "An online marketplace needs a scalable, near-real-time way to share millions of transactions with multiple internal apps while stripping sensitive data before low-latency storage."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design High-Performing Architectures |
| **Services** | Kinesis Data Streams, AWS Lambda, DynamoDB |
| **Difficulty** | Medium |

## Question

A company runs an online marketplace serving hundreds of thousands of users at peak. It needs a scalable, near-real-time solution to share millions of financial transactions with several internal applications, while removing sensitive data before storing transactions in a document database for low-latency retrieval. What should a solutions architect recommend?

## Correct Answer

**Stream transactions into Amazon Kinesis Data Streams, use an AWS Lambda integration to strip sensitive data and write clean records to Amazon DynamoDB, while other applications consume directly from the Kinesis stream.**

## Why this is correct

Kinesis Data Streams is built for exactly this "one producer, many consumers" pattern: it retains data in shards for a configurable window and allows multiple independent applications to read the same stream concurrently and in near real time, without one consumer's pace affecting another's. That directly satisfies the requirement to share the same transaction feed with several internal applications simultaneously.

The Lambda integration handles the sensitive-data removal in-flight — Lambda can be configured as a stream consumer that processes each batch of records, redacts or removes sensitive fields, and writes the sanitized result into DynamoDB. DynamoDB then provides the single-digit-millisecond, low-latency document-style retrieval the requirement calls for.

Both Kinesis and DynamoDB scale elastically to handle the "hundreds of thousands of users" and "millions of transactions" volumes described, without the team managing servers or clusters — keeping the architecture both performant and operationally light.

## Why the alternatives fall short

- **Amazon SQS as the transport layer** — doesn't support multiple independent consumers reading the same messages at true stream scale the way Kinesis does; a message is typically consumed once and removed.
- **Kinesis Data Firehose alone** — is near-real-time and can transform records via Lambda, but it delivers to a single destination (like S3 or Redshift) rather than allowing several applications to independently consume the live feed.
- **Writing directly from the application to DynamoDB with no streaming layer** — solves storage but gives other applications no way to consume the ongoing transaction feed in near real time.

## Exam Tip

When a question needs the **same real-time data feed shared with several separate consuming applications**, think **Kinesis Data Streams** (multi-consumer), not Firehose (single delivery destination) or SQS (single-consume).
