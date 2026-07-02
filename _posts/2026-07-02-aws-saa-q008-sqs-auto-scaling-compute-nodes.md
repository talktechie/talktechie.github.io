---
title: "Q8: Modernizing a Primary/Worker-Node Batch Platform for Resiliency and Scalability"
date: 2026-07-02 08:35:00 +0530
categories: [AWS SAA, Design Resilient Architectures]
tags: [aws, sqs, auto-scaling, ec2, saa-c03, decoupling]
description: "A legacy application with a primary server coordinating jobs across compute nodes needs a modern, resilient, and scalable AWS architecture."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Resilient Architectures |
| **Services** | Amazon SQS, EC2 Auto Scaling |
| **Difficulty** | Medium |

## Question

A company is migrating a distributed application to AWS. The legacy platform has a **primary server that coordinates jobs across multiple compute nodes**, and workload is **variable**. The company wants to **maximize resiliency and scalability**.

How should the architecture be designed?

## Correct Answer

**Send jobs to an SQS queue. Run the compute nodes as EC2 instances in an Auto Scaling group, and scale the group based on the SQS queue depth (ApproximateNumberOfMessages).**

## Why this is correct

The legacy design has a fatal flaw for the cloud: the **primary server is a single point of failure and a fixed-capacity bottleneck** — if it goes down, job coordination stops; if load spikes, there's no elastic way to add capacity.

Replacing the primary's coordination role with **Amazon SQS** removes that single point of failure: the queue is a highly durable, managed buffer that jobs are submitted to, and it doesn't care how many producers or consumers touch it. Running compute nodes as **EC2 instances in an Auto Scaling group**, scaled on the **queue's backlog size**, means:

- **Resiliency**: no single "primary" to lose — if any worker instance fails, Auto Scaling replaces it, and in-flight messages simply become visible again in the queue for another worker to pick up (via SQS visibility timeout).
- **Scalability**: as the queue depth grows during bursts, a scaling policy adds more worker instances; as it drains, instances scale back in — directly matching "variable workloads."

This is the canonical **"decouple with a queue + scale workers off queue depth"** pattern that appears throughout the SAA exam.

## Why the alternatives fall short

- **Replace the primary server with a larger EC2 instance (vertical scaling only)** — still a single point of failure, and vertical scaling has a ceiling; it doesn't address resiliency at all.
- **Use an Application Load Balancer in front of the compute nodes, no queue** — an ALB is designed for request/response HTTP traffic, not for durably buffering discrete jobs; if a burst arrives faster than nodes can process, requests are dropped or time out rather than safely queued for later processing.
- **Keep a primary server but run it across two AZs with a failover standby** — improves the primary's own availability, but doesn't fix the scalability problem of a fixed-capacity coordinator, and is more operationally complex than removing the primary role entirely.

## Exam Tip

**"Coordinator/primary node distributing jobs to workers" + "variable/bursty workload" + "resiliency and scalability" → SQS queue + EC2 Auto Scaling group scaled on queue depth.** This pattern generalizes: whenever you see a legacy master–worker batch system being modernized, look for the answer that (1) removes the single coordinating node and (2) ties scaling directly to a queue metric rather than CPU alone.
