---
title: "Q7: Decoupling a Message Ingestion System That Spikes to 100,000 Messages/Second"
date: 2026-07-02 08:30:00 +0530
categories: [AWS SAA, Design Resilient Architectures]
tags: [aws, sns, sqs, saa-c03, decoupling, messaging]
description: "Dozens of microservices need to consume the same bursty stream of ingested messages, which can spike to 100,000 messages per second."
hidden: true
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Resilient Architectures |
| **Services** | Amazon SNS, Amazon SQS |
| **Difficulty** | Medium |

## Question

An application ingests incoming messages. **Dozens of other applications and microservices** need to quickly consume these messages. Volume varies drastically and can spike to **100,000 messages per second**. The company wants to **decouple** the system and **increase scalability**.

Which solution meets these requirements?

## Correct Answer

**Publish messages to an SNS topic with multiple SQS queue subscriptions ("fan-out"). Each consumer application processes messages from its own queue.**

## Why this is correct

The phrase **"dozens of consumers need the same messages"** is the giveaway for the classic **SNS fan-out pattern**. A single SNS topic can broadcast each published message to **many SQS queue subscribers simultaneously** — one queue per consumer application (or per microservice family). This gives every consumer its own independent, durable buffer:

- **Decoupling**: the publisher only knows about the SNS topic, not about any of the dozens of downstream consumers. Consumers can be added or removed without touching the publisher.
- **Scalability**: each SQS queue absorbs bursts independently. If one consumer is slow, its queue simply grows — it doesn't block or throttle any other consumer's queue.
- **Message filtering**: SQS subscriptions to an SNS topic can attach a **filter policy**, so each queue only receives the subset of messages that specific consumer actually cares about, instead of every consumer processing every message.

For raw throughput: a single standard SQS queue can handle **very high throughput** (Standard queues effectively scale near-unlimited transactions per second; even the more constrained FIFO queues can reach up to 3,000 messages/sec by default, or more with batching, and can be raised further via AWS Support) — comfortably supporting 100,000 messages/sec spread across multiple standard queues fanned out from SNS.

## Why the alternatives fall short

- **A single SQS queue with dozens of consumers polling it** — messages are consumed *once* and removed from the queue; you can't have every one of dozens of microservices see every message that way. That's a work-queue pattern (one message → one worker), not a broadcast pattern.
- **Direct HTTP/API calls from the producer to every consumer** — tightly couples the producer to every consumer's availability and address; a burst to 100,000/sec would need to fan out synchronously to dozens of endpoints, which doesn't scale and isn't resilient to any single consumer being slow or down.
- **Kinesis Data Streams with a single shard reading model** — Kinesis is a strong alternative for *ordered, replayable* streaming with independent consumer checkpoints, but for "dozens of independent microservices that each need a durable, simple queue," SNS+SQS fan-out is the more natural, lower-overhead fit and is the textbook SAA answer for this exact phrasing.

## Exam Tip

**"One producer, many independent consumers, each needs every message" → SNS fan-out to multiple SQS queues.** **"One producer, many workers, each message processed once" → a single SQS queue with consumers in an Auto Scaling group** (see Q8). Recognizing which of these two shapes the question describes is one of the highest-value pattern recognitions on the whole exam.
