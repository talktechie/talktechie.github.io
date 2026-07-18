---
title: "Q29: Routing a Multi-Region UDP VoIP Service to the Lowest-Latency Region With Failover"
date: 2026-07-04 08:40:00 +0530
categories: [AWS SAA, Design Resilient Architectures]
tags: [aws, global-accelerator, nlb, saa-c03, multi-region, networking]
description: "A VoIP service using UDP, deployed across multiple Regions behind Auto Scaling groups, needs to route users to the lowest-latency Region with automated failover between Regions."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Resilient Architectures |
| **Services** | AWS Global Accelerator, Network Load Balancer, EC2 Auto Scaling |
| **Difficulty** | Medium |

## Question

A VoIP service uses **UDP** connections, running on EC2 instances in Auto Scaling groups across **multiple AWS Regions**. The company needs to route users to the Region with the **lowest latency**, plus **automated failover** between Regions.

Which solution meets these requirements?

## Correct Answer

**Deploy a Network Load Balancer (NLB) with an associated target group tied to the Auto Scaling group in each Region. Register each Region's NLB as an endpoint behind an AWS Global Accelerator.**

## Why this is correct

Two requirements point straight at **AWS Global Accelerator**: routing to the **lowest-latency** Region, and **automated cross-Region failover** — both are Global Accelerator's core features. It uses **two static anycast IP addresses** that route incoming traffic over AWS's private global network backbone to the **closest healthy endpoint**, measured by real network latency, not simple geography. If an endpoint (or an entire Region) becomes unhealthy, Global Accelerator automatically redirects traffic to the next-best healthy endpoint — instant, automated failover with no DNS propagation delay (unlike Route 53-based failover, which is subject to DNS TTL caching on clients).

The **UDP** requirement is the second key clue: Global Accelerator supports **TCP and UDP** traffic, unlike CloudFront, which is HTTP/HTTPS (Layer 7) only — ruling out a CloudFront-based design for a VoIP workload. The endpoints behind Global Accelerator need to be load balancers or EC2 instances/Elastic IPs; a **Network Load Balancer** is the right choice here because NLB operates at **Layer 4** and natively supports UDP traffic to the Auto Scaling group's EC2 targets, whereas an ALB is HTTP/HTTPS-only and wouldn't support the VoIP service's UDP protocol at all.

## Why the alternatives fall short

- **Use Route 53 latency-based routing directly to each Region's NLB, with health checks for failover** — a valid multi-Region latency-routing pattern in general, but DNS-based failover is slower (bound by DNS TTL and client-side caching) compared to Global Accelerator's near-instant failover at the network layer, which better satisfies "automated failover."
- **Deploy an Application Load Balancer in each Region behind Global Accelerator** — ALB is Layer 7 (HTTP/HTTPS) and does not support UDP, making it incompatible with a VoIP service's UDP-based traffic.
- **Use Amazon CloudFront with multiple origins across Regions** — CloudFront is built for HTTP/HTTPS content delivery and caching, not for routing arbitrary UDP traffic; it's the wrong service class entirely for a real-time VoIP workload.

## Exam Tip

**"UDP" + "multi-Region" + "lowest latency routing" + "automated failover" → AWS Global Accelerator with Network Load Balancer endpoints.** Remember the Layer/protocol distinctions: **CloudFront** = HTTP/HTTPS content delivery (Layer 7, caching). **Global Accelerator** = TCP/UDP traffic acceleration and routing (Layer 3/4, no caching) — the go-to whenever a non-HTTP protocol like UDP appears alongside multi-Region latency/failover requirements.
