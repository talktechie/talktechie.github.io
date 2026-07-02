---
title: "Q10: Guaranteeing Ecommerce Orders Are Processed in the Exact Order Received"
date: 2026-07-02 08:45:00 +0530
categories: [AWS SAA, Design Resilient Architectures]
tags: [aws, api-gateway, sqs, sqs-fifo, lambda, saa-c03]
description: "An ecommerce app sends new order events to API Gateway and must guarantee orders are processed in the same order they were received."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Resilient Architectures |
| **Services** | Amazon API Gateway, Amazon SQS FIFO, AWS Lambda |
| **Difficulty** | Medium |

## Question

An ecommerce web application sends new-order information to an **Amazon API Gateway REST API** for processing. The company wants to guarantee orders are **processed in the exact order they were received**.

Which solution meets this requirement?

## Correct Answer

**Use an API Gateway integration to send each order as a message to an SQS FIFO queue. Configure the FIFO queue to invoke a Lambda function for processing.**

## Why this is correct

The one non-negotiable requirement here is **strict ordering** — and that single word is the whole question. Standard SQS queues make a **best-effort** attempt at ordering but explicitly **do not guarantee** it, and can even deliver a message more than once. **SQS FIFO (First-In-First-Out) queues** are the AWS primitive specifically built to guarantee:

1. **Exactly the order messages were sent** is the order they're delivered (within a given `MessageGroupId`).
2. **Exactly-once processing** — no duplicate delivery, unlike standard queues.

API Gateway can integrate directly with SQS (using a **service integration**, bypassing the need for a separate Lambda just to enqueue the message), pushing each incoming order straight onto the FIFO queue as soon as it's received — preserving arrival order at the entry point. The FIFO queue then triggers a **Lambda function** as consumer, processing messages strictly in sequence.

## Why the alternatives fall short

- **API Gateway → standard SQS queue → Lambda** — standard queues explicitly do **not guarantee** ordering; under load, messages can be delivered out of sequence, directly violating the requirement.
- **API Gateway → Lambda → DynamoDB, invoked directly per request (no queue)** — with concurrent Lambda invocations processing requests in parallel, there is **no mechanism to preserve order** — two orders that arrive close together could easily finish processing in reverse order.
- **API Gateway → Kinesis Data Streams → Lambda** — Kinesis *can* preserve order per shard (per partition key), and is a valid tool for high-throughput ordered streaming, but for a straightforward "process orders in the order received" requirement without extreme throughput needs, **SQS FIFO** is the simpler, more directly-applicable, and more commonly correct SAA-exam answer. Kinesis becomes the better answer specifically when the scenario also demands very high throughput and/or multiple independent consumers replaying the same ordered stream.

## Exam Tip

The single keyword **"order" / "sequence" / "exact order received"** in a messaging question should immediately make you think **SQS FIFO**, not standard SQS. Remember the trade-off: FIFO queues cap throughput lower than standard queues (though batching and high-throughput mode raise this significantly), so if a question demands *both* strict ordering *and* massive throughput with many independent consumers, that's your cue to consider **Kinesis Data Streams** instead.
