---
title: "Q4: Give an EC2 Instance Private Access to S3 With No Internet Connectivity"
date: 2026-07-02 08:15:00 +0530
categories: [AWS SAA, Design Secure Architectures]
tags: [aws, vpc, s3, vpc-endpoint, saa-c03, networking]
description: "An EC2 instance in a VPC needs to reach an S3 bucket to process logs, without any route to the public internet."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Secure Architectures |
| **Services** | Amazon VPC, Gateway VPC Endpoint, Amazon S3 |
| **Difficulty** | Easy–Medium |

## Question

An application runs on an EC2 instance inside a VPC and processes logs stored in an S3 bucket. The instance **must reach S3 without any internet connectivity** (no internet gateway, no NAT).

Which solution provides private network connectivity to S3?

## Correct Answer

**Create a Gateway VPC Endpoint to Amazon S3.**

## Why this is correct

A **VPC endpoint** lets resources inside a VPC talk to supported AWS services **over Amazon's private network**, entirely bypassing the public internet — no internet gateway, no NAT gateway, no additional cost for the endpoint itself.

S3 (along with DynamoDB) uses the **Gateway endpoint** type specifically. You add a route in the VPC's route table pointing traffic destined for S3's IP ranges to the gateway endpoint's prefix list. From the EC2 instance's point of view, it just calls `s3.amazonaws.com` as normal — routing quietly keeps that traffic inside AWS's network instead of going out to the internet.

This directly satisfies "access S3 without connectivity to the internet."

## Why the alternatives fall short

- **NAT Gateway + Internet Gateway** — this *does* let a private instance reach S3, but the traffic still transits the public internet path (via NAT out through an IGW), which contradicts "without connectivity to the internet." It also costs more and adds complexity.
- **Interface VPC Endpoint (powered by AWS PrivateLink)** — also private, but S3 primarily uses the simpler, no-cost **Gateway** endpoint type; Interface endpoints are used for services that don't support Gateway endpoints (e.g., most other AWS APIs, or when you specifically need on-premises/peered-VPC access to S3, which Gateway endpoints don't support).
- **VPN or Direct Connect to AWS** — solves on-premises-to-AWS connectivity, irrelevant here since the EC2 instance is already inside the VPC.

## Exam Tip

Memorize this distinction cold — it shows up repeatedly on the SAA exam:

| | Gateway Endpoint | Interface Endpoint |
|---|---|---|
| Services | S3, DynamoDB only | Most other AWS services |
| Cost | Free | Hourly + data charge |
| Mechanism | Route table entry | ENI with private IP (PrivateLink) |
| Reachable from on-prem / peered VPC | ❌ No | ✅ Yes |

"EC2 in a VPC needs S3, no internet" → **Gateway VPC Endpoint**, every time.
