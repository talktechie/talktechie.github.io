---
title: "Q18: Building a Durable, Stateless Image-Compression Microservice (Choose Two)"
date: 2026-07-03 08:35:00 +0530
categories: [AWS SAA, Design Resilient Architectures]
tags: [aws, sqs, lambda, s3, saa-c03, decoupling, serverless]
description: "A microservice must compress uploaded images automatically using durable, stateless components — moving them from an intake S3 bucket to a compressed-output S3 bucket."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Resilient Architectures |
| **Services** | Amazon S3, Amazon SQS, AWS Lambda |
| **Difficulty** | Medium |
| **Question Type** | Choose TWO |

## Question

A microservice converts large images to smaller, compressed images. When a user uploads an image via the web interface, the service should: store the image in an S3 bucket, process/compress it with a **Lambda** function, and store the compressed result in a **different** S3 bucket. The design must use **durable, stateless components** to process images automatically.

Which two actions meet these requirements?

## Correct Answers

**A. Create an SQS queue. Configure the source S3 bucket to send a notification to the SQS queue when an image is uploaded.**

**B. Configure the Lambda function to use the SQS queue as its invocation source. After a message is successfully processed, delete it from the queue.**

## Why this is correct

The requirement's key phrase is **"durable, stateless components."** An S3 event triggering Lambda directly *can* work for simple cases, but adding **SQS as an intermediary buffer** between the upload event and the processing function is what makes the pipeline genuinely **durable**:

- **S3 → SQS notification (Answer A):** When an image lands in the source bucket, S3 sends an event notification into an SQS queue rather than invoking Lambda directly. The message — durably stored, replicated across multiple AZs by SQS — now exists independently of whether a Lambda function happens to be available or briefly throttled at that exact moment. If processing fails or Lambda concurrency is temporarily exhausted, the message simply waits safely in the queue instead of being dropped.
- **SQS → Lambda, delete on success (Answer B):** Lambda polls (or is triggered by) the SQS queue as its event source. Each invocation is **stateless** — it processes exactly one message/image with no dependency on prior invocations — a natural fit for Lambda's execution model. Only after the image is successfully compressed and written to the destination bucket does the function delete the message from the queue. If processing fails partway through, the message becomes visible again after its visibility timeout expires, so another invocation retries it automatically — no manual recovery, and no message silently lost.

Together, this builds a self-healing pipeline: durable at the messaging layer, stateless at the compute layer.

## Why the alternatives fall short

- **Configure the S3 bucket to invoke the Lambda function directly on upload, with no queue** — works for simple, low-volume cases, but removes the durable buffer: if Lambda throttles under a burst of uploads or a transient failure occurs, S3 event notifications delivered directly to Lambda don't have the same built-in retry/backlog behavior as an SQS-backed pipeline, making the pipeline less resilient to load spikes or processing failures.
- **Store processing state (e.g., "in progress") in the Lambda function's local `/tmp` storage across invocations** — violates "stateless": Lambda execution environments are ephemeral and not guaranteed to be reused, so relying on local state between invocations is unreliable and defeats the stateless design goal.
- **Use a single long-running EC2 instance to poll the source bucket for new images** — introduces a persistent server that must be kept running and patched, the opposite of a stateless, event-driven microservice architecture, and a single instance is also a scaling and availability bottleneck.

## Exam Tip

When a question explicitly asks for **"durable" AND "stateless"** components together, look for the combination of **a durable buffer (SQS or S3 itself) feeding a stateless compute layer (Lambda)** — this pattern (S3 event → SQS → Lambda) is one of the most heavily tested serverless architectures on the SAA exam, precisely because it demonstrates decoupling, automatic retry, and elastic scaling all at once.
