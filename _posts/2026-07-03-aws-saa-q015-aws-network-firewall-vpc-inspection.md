---
title: "Q15: Replicating an On-Premises Inspection Server's Traffic Filtering in AWS"
date: 2026-07-03 08:20:00 +0530
categories: [AWS SAA, Design Secure Architectures]
tags: [aws, network-firewall, vpc, saa-c03, security, networking]
description: "A company migrated to AWS and wants to protect traffic in and out of the production VPC the way its on-premises inspection server used to."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Secure Architectures |
| **Services** | AWS Network Firewall, Amazon VPC |
| **Difficulty** | Easy–Medium |

## Question

A company migrated to AWS and wants to protect traffic flowing in and out of its **production VPC**. On-premises, an inspection server performed **traffic flow inspection and traffic filtering**. The company wants the same functionality in AWS.

Which solution meets these requirements?

## Correct Answer

**Use AWS Network Firewall to create the required rules for traffic inspection and filtering for the production VPC.**

## Why this is correct

**AWS Network Firewall** is a managed, stateful network firewall service purpose-built for exactly this: inspecting and filtering traffic entering and leaving a VPC. It supports:

- **Stateful and stateless rule groups** for filtering based on protocol, port, source/destination, and even domain names.
- **Intrusion detection/prevention (IDS/IPS)** style rules using Suricata-compatible rule syntax, for deep traffic flow inspection — a direct match for what the on-premises appliance did.
- Deployment as a managed service that you attach to VPC subnets via firewall endpoints, without you having to build, patch, or scale the underlying inspection infrastructure yourself.

Because it's a fully managed AWS service (not a marketplace appliance you have to run), it satisfies "same functionality" while also reducing the operational burden of what used to be a self-managed on-premises box.

## Why the alternatives fall short

- **Security Groups and Network ACLs only** — these provide basic allow/deny filtering by IP/port, but they're not stateful deep-packet inspection tools and can't do the kind of protocol-level traffic flow inspection an IDS/IPS-style appliance performs.
- **Deploy a third-party firewall appliance from AWS Marketplace** — can replicate the same functionality, but requires you to size, patch, and scale the appliance instances yourself — more operationally similar to the original on-premises setup than to a managed AWS-native solution, and not the leaner, purpose-built answer when AWS Network Firewall exists.
- **AWS WAF** — operates at Layer 7 for HTTP/HTTPS applications (e.g., behind CloudFront or an ALB), protecting against web exploits like SQL injection and XSS. It doesn't provide general VPC-wide traffic flow inspection and filtering across all protocols the way Network Firewall does.

## Exam Tip

Distinguish AWS's traffic-protection services by scope: **AWS WAF** = Layer 7 web application protection (HTTP/HTTPS). **AWS Shield** = DDoS protection. **Security Groups/NACLs** = basic instance/subnet-level allow-deny rules. **AWS Network Firewall** = managed, stateful, VPC-wide traffic inspection and filtering across all traffic — the go-to answer whenever a question describes replacing a general-purpose on-premises firewall/inspection appliance.
