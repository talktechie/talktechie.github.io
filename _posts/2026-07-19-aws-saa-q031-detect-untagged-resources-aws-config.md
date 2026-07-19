---
title: "Q31: Auto-Detect Untagged AWS Resources with Config Rules"
date: 2026-07-19 08:35 +0530
categories: [AWS SAA, Design Secure Architectures]
tags: [aws, aws-config, tagging, governance, saa-c03]
description: "A company wants to detect EC2, RDS, and Redshift resources that aren't properly tagged, with minimal ongoing operational effort."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Secure Architectures |
| **Services** | AWS Config |
| **Difficulty** | Easy |

## Question

A company hosts its web application on AWS and wants to ensure all Amazon EC2 instances, Amazon RDS DB instances, and Amazon Redshift clusters are configured with tags. The company wants to minimize the effort of configuring and operating this check. What should a solutions architect do?

## Correct Answer

**Use AWS Config rules — specifically the managed `required-tags` rule — to continuously define and detect resources that are not properly tagged.**

## Why this is correct

AWS Config continuously records the configuration state of your resources and evaluates them against rules you define. The `required-tags` managed rule is purpose-built for exactly this scenario: point it at EC2, RDS, and Redshift resource types, specify the tag keys (and optionally values) you require, and Config automatically flags any resource that's out of compliance — no custom code, no scheduling infrastructure to build.

Because it's a managed rule, there's essentially zero operational overhead: Config evaluates resources on creation and on every configuration change, and non-compliant resources show up in the Config dashboard (and can trigger EventBridge notifications or even auto-remediation actions) without the team writing or maintaining any Lambda functions or cron jobs.

This directly satisfies the "minimize effort" requirement in the question — that phrase is the signal to reach for a managed AWS Config rule rather than a custom-built compliance check.

## Why the alternatives fall short

- **A scheduled Lambda function scanning resources via the API** — works, but requires writing, testing, and maintaining custom code plus a trigger schedule, which is exactly the operational overhead the question asks to avoid.
- **AWS Trusted Advisor** — offers only limited tagging-related checks and doesn't provide a continuous, rule-based enforcement mechanism across all three services the way Config does.
- **Manual periodic tagging audits** — doesn't scale, isn't continuous, and introduces human error; it also fails the "minimize effort" bar entirely.

## Exam Tip

Whenever a question pairs "minimize effort/operational overhead" with "check resource configuration or compliance," the answer is almost always an **AWS Config managed rule**, not a custom script.
