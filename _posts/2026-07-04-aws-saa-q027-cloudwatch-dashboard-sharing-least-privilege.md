---
title: "Q27: Sharing a CloudWatch Dashboard With Someone Who Has No AWS Account"
date: 2026-07-04 08:30:00 +0530
categories: [AWS SAA, Design Secure Architectures]
tags: [aws, cloudwatch, iam, saa-c03, security, least-privilege]
description: "A product manager without an AWS account needs periodic access to a CloudWatch dashboard, following the principle of least privilege."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Secure Architectures |
| **Services** | Amazon CloudWatch (dashboard sharing) |
| **Difficulty** | Easy |

## Question

A product manager needs periodic access to an application's **CloudWatch dashboard** but has **no AWS account**. A solutions architect must grant access following the **principle of least privilege**.

Which solution meets these requirements?

## Correct Answer

**Share the dashboard directly from the CloudWatch console using the product manager's email address, and send them the shareable link.**

## Why this is correct

CloudWatch has a built-in **dashboard sharing** feature designed exactly for this use case: giving **non-AWS-account users** limited, read-only access to a *specific* dashboard, without provisioning them an IAM user, federated role, or any broader AWS access.

The workflow is simple: from the CloudWatch console, you share the dashboard by entering the person's **email address**; they receive an invitation, set their **own password**, and from then on can log in through a dedicated shareable link to **view that one dashboard only**. They get no access to the AWS Management Console, no ability to see other resources, and no permissions beyond viewing that specific dashboard — a textbook example of **least privilege**: exactly the access needed, nothing more.

## Why the alternatives fall short

- **Create an IAM user for the product manager with `CloudWatchReadOnlyAccess`** — grants far more than needed: read access to *all* CloudWatch metrics, alarms, logs, and dashboards across the account, not just the one dashboard in question — violates least privilege by over-granting.
- **Create an IAM role and have the product manager use AWS SSO to assume it** — requires the product manager to have some form of federated identity/account access set up, which is unnecessary overhead (and access beyond a single dashboard) for someone who just needs to periodically view one screen.
- **Take periodic screenshots of the dashboard and email them** — technically avoids granting any AWS access, but it's manual, not "live," doesn't scale, and isn't really a scalable "solution" an architect would design — more of a workaround than an access-control mechanism.

## Exam Tip

**"Someone outside the AWS account/organization needs to view a specific CloudWatch dashboard" + "least privilege" → CloudWatch's native dashboard sharing (via email invite), not an IAM user/role.** This is a narrow but recurring exam pattern: recognize when a purpose-built, scoped sharing feature exists rather than reaching for broader IAM-based access.
