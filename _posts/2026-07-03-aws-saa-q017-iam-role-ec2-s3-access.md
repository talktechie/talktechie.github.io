---
title: "Q17: The Right Way for EC2 Instances to Access an S3 Bucket"
date: 2026-07-03 08:30:00 +0530
categories: [AWS SAA, Design Secure Architectures]
tags: [aws, iam, ec2, s3, saa-c03, security, fundamentals]
description: "A new business application runs on two EC2 instances and needs access to an S3 bucket for document storage."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Secure Architectures |
| **Services** | AWS IAM, Amazon EC2, Amazon S3 |
| **Difficulty** | Easy |

## Question

A new business application runs on two EC2 instances and uses an S3 bucket for document storage. A solutions architect needs to ensure the EC2 instances can access the S3 bucket.

What should the solutions architect do?

## Correct Answer

**Create an IAM role that grants access to the S3 bucket, and attach the role to the EC2 instances.**

## Why this is correct

This is the SAA exam's most fundamental security pattern, and it comes up constantly disguised in bigger scenarios: **AWS resources should authenticate to other AWS resources using IAM roles, never hardcoded long-term credentials.**

An **IAM role** attached to an EC2 instance (via an **instance profile**) provides the instance with **temporary, automatically rotated credentials** through the instance metadata service. The application code (or AWS SDK) picks these up automatically — no access keys stored on disk, no secrets to manage or leak. You scope the role's policy to grant exactly the S3 permissions needed (e.g., `s3:GetObject`, `s3:PutObject` on the specific bucket/prefix), following least privilege.

Because the role can be attached to any number of instances, and both EC2 instances in this scenario need the same access, one role serves both — simple and consistent.

## Why the alternatives fall short

- **Create an IAM user with an access key/secret key, and hardcode the credentials into the application** — works, but hardcoded long-term credentials are a security anti-pattern: they don't rotate automatically, are easy to accidentally leak (e.g., committed to source control), and require manual rotation and redistribution to every instance.
- **Make the S3 bucket public** — grants access, but to *everyone*, not just your application — a serious security exposure with no meaningful access control.
- **Use a bucket policy that allows access from the EC2 instances' public IP addresses** — IP-based access control is fragile (IPs change on instance stop/start unless using Elastic IPs) and doesn't authenticate *who* is making the request, only *where* it's coming from — far weaker and more brittle than IAM-role-based identity.

## Exam Tip

**Any AWS-service-to-AWS-service access question where the "correct" mechanism isn't obviously something else (like a VPC endpoint) almost always reduces to: attach an IAM role with a least-privilege policy.** If you see "access key," "secret key," "hardcoded credentials," or "public bucket" as answer options for service-to-service access, they're virtually always the wrong choice on this exam.
