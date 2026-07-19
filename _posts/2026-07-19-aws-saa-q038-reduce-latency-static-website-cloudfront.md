---
title: "Q38: Cut Global Website Latency with CloudFront in Front of S3"
date: 2026-07-19 08:10 +0530
categories: [AWS SAA, Design High-Performing Architectures]
tags: [aws, cloudfront, s3, route53, saa-c03]
description: "A static website on S3 with Route 53 DNS is seeing growing global demand and needs lower latency, most cost-effectively."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design High-Performing Architectures |
| **Services** | Amazon CloudFront, Amazon S3, Amazon Route 53 |
| **Difficulty** | Easy |

## Question

A company is hosting a static website on Amazon S3 and using Amazon Route 53 for DNS. The website is experiencing increased demand from around the world, and the company must decrease latency for users accessing it. Which solution meets these requirements most cost-effectively?

## Correct Answer

**Add an Amazon CloudFront distribution in front of the S3 bucket, and update the Route 53 records to point to the CloudFront distribution.**

## Why this is correct

CloudFront is a global content delivery network that caches content at edge locations around the world, so users are served from the location nearest to them instead of always reaching back to the S3 bucket's region. For a static website suddenly seeing worldwide demand, this directly reduces round-trip latency without changing the underlying storage layer at all.

Because CloudFront's pricing is pay-as-you-go based on actual data transfer and requests, adding it in front of an existing S3-hosted site doesn't introduce new fixed infrastructure costs — it scales with real traffic and typically reduces load (and therefore cost pressure) on the origin bucket, since repeat requests are served from cache rather than S3 directly.

Updating the Route 53 record to an alias pointing at the CloudFront distribution is a simple DNS change, so the whole upgrade requires no re-architecture of the website itself — just a caching layer placed in front of what's already there.

## Why the alternatives fall short

- **Replicating the S3 bucket across multiple regions** — adds cross-region sync complexity and storage duplication costs, and still doesn't solve last-mile latency the way edge caching does.
- **Migrating the site to EC2 web servers in multiple regions** — a major and unnecessary increase in cost and operational overhead for content that's purely static.
- **S3 Transfer Acceleration** — designed to speed up uploads into S3 over long distances, not to reduce latency for end users reading the site's content.

## Exam Tip

Static content + growing **global** audience + need to reduce latency **cost-effectively** is almost always solved by **CloudFront in front of S3** — look for this pattern instantly.
