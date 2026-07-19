---
title: "Q34: Track Config Changes and API Calls with Config + CloudTrail"
date: 2026-07-19 08:50 +0530
categories: [AWS SAA, Design Secure Architectures]
tags: [aws, aws-config, cloudtrail, compliance, saa-c03]
description: "A company needs to track configuration changes on its AWS resources and record a history of API calls for compliance, governance, and security."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Secure Architectures |
| **Services** | AWS Config, AWS CloudTrail |
| **Difficulty** | Easy |

## Question

A company hosts multi-tier applications on AWS. For compliance, governance, auditing, and security, it must track configuration changes on its AWS resources and record a history of API calls made to those resources. What should a solutions architect do?

## Correct Answer

**Use AWS Config to track configuration changes over time, and AWS CloudTrail to record the history of API calls made to AWS resources.**

## Why this is correct

These two services are complementary and cover different halves of the requirement. AWS Config continuously records the configuration state of resources and maintains a timeline of how each resource's configuration has changed, which is exactly what "track configuration changes" calls for — it's central to compliance and governance auditing.

AWS CloudTrail, by contrast, is focused on the identity and activity layer: it logs every API call made against your account, capturing who made the call, when, from where, and what action was taken. That satisfies the "record a history of API calls" requirement and is essential for security investigations and audit trails.

Using both together gives a complete governance picture — Config answers "what changed and when," while CloudTrail answers "who made it happen" — which is why compliance-flavored questions on the exam frequently expect both services named together.

## Why the alternatives fall short

- **CloudTrail alone** — captures API activity but doesn't maintain a structured history of resource configuration state, so it can't answer "what did this resource's configuration look like last week."
- **AWS Config alone** — tracks configuration history well but doesn't provide the identity-level API call audit trail needed for security and access accountability.
- **Amazon CloudWatch Logs/Events** — useful for general monitoring and alerting, but it isn't purpose-built for configuration history or a comprehensive API-call audit log the way Config and CloudTrail are.

## Exam Tip

"Who did what" points to **CloudTrail**; "what changed on a resource over time" points to **AWS Config**. Compliance/audit questions almost always want **both named together**.
