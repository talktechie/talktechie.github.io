---
title: "Q26: Catching Unauthorized Configuration Changes on S3 Buckets"
date: 2026-07-04 08:25:00 +0530
categories: [AWS SAA, Design Secure Architectures]
tags: [aws, config, s3, saa-c03, security, compliance]
description: "A company needs to ensure its S3 buckets do not have unauthorized configuration changes, as part of reviewing its AWS Cloud deployment."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Secure Architectures |
| **Services** | AWS Config |
| **Difficulty** | Easy |

## Question

A company needs to review its AWS deployment to ensure its **S3 buckets do not have unauthorized configuration changes**.

What should a solutions architect do?

## Correct Answer

**Turn on AWS Config with the appropriate rules.**

## Why this is correct

**AWS Config** continuously records the configuration state of your AWS resources — including S3 bucket settings like encryption, public access block settings, versioning, and bucket policies — and tracks how that configuration changes over time.

Layer on **Config Rules** (either AWS-managed, like `s3-bucket-public-read-prohibited` or `s3-bucket-server-side-encryption-enabled`, or custom rules), and Config continuously evaluates whether each bucket's actual configuration **complies** with your defined desired state. If someone changes a bucket setting in a way that violates a rule — say, disabling encryption or opening public access — Config flags that resource as **non-compliant** and can trigger notifications (via EventBridge/SNS) so the team is alerted to the unauthorized change almost immediately, along with a full history of exactly what changed and when.

This is precisely "ensure S3 buckets do not have unauthorized configuration changes" — continuous, automated, auditable configuration monitoring.

## Why the alternatives fall short

- **Enable AWS CloudTrail only** — CloudTrail logs *API calls* (who did what, when), which is valuable for investigating an incident after the fact, but it doesn't natively evaluate whether current resource configurations are compliant with a defined policy, or flag non-compliant resources — it's an audit log, not a continuous compliance/config-drift detector.
- **Use Amazon Macie** — Macie focuses on discovering and protecting **sensitive data** in S3 (e.g., PII detection), not on monitoring bucket-level configuration changes.
- **Manually review S3 bucket settings periodically via the console** — possible, but entirely manual, doesn't catch changes in near-real-time, and scales poorly as the number of buckets grows — the opposite of an automated compliance solution.

## Exam Tip

**"Detect/ensure no unauthorized configuration changes to AWS resources" → AWS Config (with rules).** Contrast with **CloudTrail**, which answers "who did what and when" (API activity auditing), and **Config**, which answers "is this resource's *current configuration* compliant, and how has it changed over time." Questions often pair the two service names as answer choices specifically to test that you know this distinction.
