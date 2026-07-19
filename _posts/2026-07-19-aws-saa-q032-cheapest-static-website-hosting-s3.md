---
title: "Q32: Host a Static Website for Pennies Using S3"
date: 2026-07-19 08:40 +0530
categories: [AWS SAA, Design Cost-Optimized Architectures]
tags: [aws, s3, static-website, cost-optimization, saa-c03]
description: "A team needs to host an internal HTML/CSS/JS website for other teams in the most cost-effective way possible."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Cost-Optimized Architectures |
| **Services** | Amazon S3 |
| **Difficulty** | Easy |

## Question

A development team needs to host a website, accessed by other teams, consisting of HTML, CSS, client-side JavaScript, and images. Which method is the most cost-effective for hosting the website?

## Correct Answer

**Create an Amazon S3 bucket, enable static website hosting on it, and serve the site directly from S3.**

## Why this is correct

The content described — HTML, CSS, client-side JS, and images — is entirely static; none of it requires server-side processing. S3's static website hosting feature serves these files straight from the bucket over HTTP(S), with no compute layer to provision, patch, or scale.

Cost-wise, this is as lean as it gets: you pay only for storage (cents per GB) and requests, with no idle compute charges. There's no EC2 instance, load balancer, or container service sitting around waiting for traffic, and S3 already provides high durability and availability out of the box.

Because the audience is internal teams rather than a global public user base, there's no immediate need for a CDN layer either — S3 alone meets the requirement at the lowest possible cost while remaining simple to operate.

## Why the alternatives fall short

- **EC2 instance behind an Elastic Load Balancer** — introduces ongoing compute and load balancer costs for content that needs no server-side logic; pure over-provisioning.
- **AWS Elastic Beanstalk** — still provisions underlying EC2 and other resources to manage, adding cost and operational surface area for a workload that doesn't need an application server.
- **S3 + CloudFront** — a valid performance upgrade, but adds distribution costs that aren't justified for an internal team audience where "most cost-effective" is the deciding factor.

## Exam Tip

Whenever a question describes **static content only** (HTML/CSS/JS/images, no server-side code) and asks for the cheapest hosting option, the answer is almost always **S3 static website hosting** by itself.
