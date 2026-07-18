---
title: "Q19: Routing All Inbound Web Traffic Through a Third-Party Firewall Appliance"
date: 2026-07-03 08:40:00 +0530
categories: [AWS SAA, Design Secure Architectures]
tags: [aws, gateway-load-balancer, vpc, saa-c03, security, networking]
description: "A three-tier app needs every request to pass through a third-party virtual firewall appliance in a separate inspection VPC before reaching the web tier, with the least operational overhead."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Secure Architectures |
| **Services** | Gateway Load Balancer (GWLB), Gateway Load Balancer Endpoint, VPC |
| **Difficulty** | Medium–Hard |

## Question

A three-tier web app has web servers in a public subnet, and app/database servers in private subnets, all in one VPC. A **third-party virtual firewall appliance** is already deployed in a separate **inspection VPC**, with an IP interface that accepts IP packets. All traffic to the web tier must pass through this appliance for inspection first.

Which solution meets this with the **least operational overhead**?

## Correct Answer

**Deploy a Gateway Load Balancer in the inspection VPC. Create a Gateway Load Balancer endpoint (in the application's VPC) to receive incoming packets and forward them to the appliance.**

## Why this is correct

**Gateway Load Balancer (GWLB)** exists specifically for this pattern: transparently inserting third-party virtual appliances (firewalls, IDS/IPS, deep packet inspection tools) into the traffic path between VPCs, at Layer 3, without redesigning the application's network architecture.

The design works as a **hub-and-spoke** model: the appliance(s) sit behind a GWLB in a centralized **inspection VPC** (the hub). Each "spoke" VPC (like the app's VPC here) gets a lightweight **Gateway Load Balancer endpoint (GWLBE)**, powered by AWS PrivateLink. Traffic destined for the web tier is routed (via a route table entry) to the GWLB endpoint, which transparently forwards it to the appliance for inspection, then back to its original destination if allowed.

This satisfies "least operational overhead" because:
- One centralized fleet of appliances behind GWLB can inspect traffic for **many spoke VPCs**, instead of deploying and managing separate appliance instances per VPC.
- GWLB automatically **load-balances and health-checks** the appliance fleet, scaling it as traffic grows — no manual scaling of the firewall layer.

## Why the alternatives fall short

- **Deploy the third-party appliance directly in the application's VPC, in-line via route table changes** — works for a single VPC, but doesn't centralize inspection for multiple VPCs, and requires managing appliance scaling/HA independently in every VPC that needs inspection — much higher long-term operational overhead.
- **Use an Application Load Balancer in front of the appliance** — ALB operates at Layer 7 (HTTP/HTTPS) and is designed for request routing to web application targets, not for transparently forwarding arbitrary IP packets to a Layer 3 network appliance. It's the wrong tool for this traffic-inspection use case.
- **Use AWS Transit Gateway alone to route traffic to the inspection VPC** — Transit Gateway is excellent for VPC-to-VPC and on-premises connectivity/routing at scale, but by itself it doesn't provide load balancing, health checking, or scaling across a fleet of inspection appliances — that's specifically what Gateway Load Balancer adds on top.

## Exam Tip

**"Third-party virtual appliance" + "inspect traffic between VPCs" + "least operational overhead" + "centralized inspection VPC" → Gateway Load Balancer + Gateway Load Balancer Endpoint.** This is a distinctly different load balancer type from ALB (Layer 7) and NLB (Layer 4) — GWLB operates at Layer 3 and is purpose-built for exactly one job: inserting and scaling third-party network appliances transparently into traffic flows.
