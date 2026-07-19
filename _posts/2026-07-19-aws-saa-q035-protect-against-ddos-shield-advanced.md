---
title: "Q35: Shield Your Public App from Large-Scale DDoS Attacks"
date: 2026-07-19 08:55 +0530
categories: [AWS SAA, Design Secure Architectures]
tags: [aws, shield, ddos, elb, saa-c03]
description: "A public-facing web app behind an ELB, using third-party DNS, needs detection and protection against large-scale DDoS attacks."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Secure Architectures |
| **Services** | AWS Shield Advanced, Elastic Load Balancer |
| **Difficulty** | Easy–Medium |

## Question

A company is preparing to launch a public-facing web application. The architecture consists of EC2 instances within a VPC behind an Elastic Load Balancer, with DNS managed by a third-party service. The solutions architect must recommend a solution to detect and protect against large-scale DDoS attacks. Which solution meets these requirements?

## Correct Answer

**Enable AWS Shield Advanced and assign it to protect the Elastic Load Balancer.**

## Why this is correct

AWS Shield Standard is automatically enabled for every AWS customer at no extra cost, but it only defends against the most common, lower-sophistication network and transport layer attacks. The question specifically calls out "large-scale" DDoS attacks, which is the signal to escalate to **Shield Advanced** — it provides expanded detection and mitigation capabilities, near-real-time visibility into attacks, and access to the AWS DDoS Response Team (DRT) for active large-scale events.

Shield Advanced can be explicitly attached to protected resources including Elastic Load Balancers, EC2 instances, CloudFront distributions, Global Accelerator accelerators, and Route 53 hosted zones. Since this architecture's public entry point is the ELB (and DNS is handled by a third party, not Route 53), assigning Shield Advanced to the ELB directly protects the resource actually receiving the traffic.

Shield Advanced also includes cost protection against scaling charges incurred during a DDoS event, which matters for a company about to launch a public-facing application at unpredictable scale.

## Why the alternatives fall short

- **Relying on Shield Standard only** — provides baseline protection automatically, but lacks the enhanced detection, reporting, and dedicated response support needed for "large-scale" attacks.
- **AWS WAF alone** — protects against application-layer exploits like SQL injection and XSS, but isn't designed to absorb or mitigate volumetric DDoS traffic.
- **Migrating DNS to Route 53** — an unnecessary re-architecture; Shield Advanced can protect the ELB directly regardless of which DNS provider is in use, so this doesn't address the core requirement.

## Exam Tip

"Large-scale DDoS" plus a need for detailed attack visibility/response support is the signal for **Shield Advanced explicitly attached to the exposed resource** (ELB, CloudFront, Global Accelerator, or Route 53) — not just the default Shield Standard.
