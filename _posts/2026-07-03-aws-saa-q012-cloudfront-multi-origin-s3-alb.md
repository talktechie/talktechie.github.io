---
title: "Q12: Reducing Latency for Both Static and Dynamic Content on a Global Web App"
date: 2026-07-03 08:05:00 +0530
categories: [AWS SAA, Design High-Performing Architectures]
tags: [aws, cloudfront, s3, alb, route53, saa-c03, caching]
description: "A global web app serves static data from S3 and dynamic data from EC2 behind an ALB. The company wants improved performance and lower latency for both."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design High-Performing Architectures |
| **Services** | Amazon CloudFront, Amazon S3, Application Load Balancer, Amazon Route 53 |
| **Difficulty** | Easy–Medium |

## Question

A global company hosts a web application on EC2 instances behind an **Application Load Balancer**. Static data is stored in **S3**; dynamic data comes from the application tier. The company wants to **improve performance and reduce latency for both static and dynamic data**, using its own domain registered in **Route 53**.

What should a solutions architect do?

## Correct Answer

**Create a CloudFront distribution with two origins — the S3 bucket and the ALB. Configure Route 53 to route the domain's traffic to the CloudFront distribution.**

## Why this is correct

The requirement covers **two different content types with one latency goal**, and CloudFront is the single service that addresses both simultaneously.

CloudFront distributions support **multiple origins with path-based behaviors**: you can route requests like `/static/*` to the **S3 bucket** origin (cached at edge locations close to every global user) and everything else (e.g., `/api/*`) to the **ALB** origin for dynamic content. Even dynamic requests benefit from CloudFront: traffic travels over **Amazon's optimized backbone network** from the nearest edge location to the origin, instead of the public internet the whole way — reducing latency even for content that can't be cached.

Since the company already owns a domain in **Route 53**, pointing an alias record at the CloudFront distribution's domain name means users worldwide resolve straight to their nearest edge location — no extra infrastructure, minimal changes to the existing setup.

## Why the alternatives fall short

- **Create two separate CloudFront distributions, one per origin** — technically works but adds unnecessary complexity (two distributions, two sets of settings, potentially two different domains/paths to manage) when one distribution with multiple origins does the job.
- **Use S3 Transfer Acceleration for the static content and leave dynamic content as-is** — improves upload speed to S3, not download/serving latency to end users, and does nothing for the ALB-served dynamic content at all.
- **Deploy the application in every AWS Region and use Route 53 latency-based routing** — a valid pattern for truly global HA and lowest ping to compute, but it's a much larger operational lift (multi-region deployment, data replication) than the question's ask, and doesn't specifically address caching static content at the edge.

## Exam Tip

**"Global users" + "static AND dynamic content" + "reduce latency" + "single domain" → CloudFront with multiple origins.** Remember CloudFront isn't just a static-content cache — even purely dynamic, uncacheable traffic gets a latency boost by riding CloudFront's edge network back to the origin over AWS's private backbone.
