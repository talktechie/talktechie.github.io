---
title: "Q3: Restrict an S3 Bucket to Only Accounts Inside Your AWS Organization"
date: 2026-07-02 08:10:00 +0530
categories: [AWS SAA, Design Secure Architectures]
tags: [aws, s3, iam, organizations, saa-c03, security]
description: "Limit access to an S3 bucket in the management account to only users from accounts within the AWS Organization, with the least operational overhead."
hidden: true
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Secure Architectures |
| **Services** | Amazon S3, AWS Organizations, IAM condition keys |
| **Difficulty** | Medium |

## Question

A company uses **AWS Organizations** to manage multiple accounts for different departments. The **management account** has an S3 bucket containing project reports. The company wants to restrict access to this bucket to **only users from accounts within the organization**.

Which solution meets this with the **least operational overhead**?

## Correct Answer

**Add the `aws:PrincipalOrgID` global condition key, referencing the organization ID, to the S3 bucket policy.**

## Why this is correct

The naive approach — listing every member account ID as an allowed principal in the bucket policy — technically works but is an operational nightmare: every time an account joins or leaves the organization, you have to edit the policy.

`aws:PrincipalOrgID` solves this in one line. It's a **global IAM condition key** that lets a resource policy check whether the requester's account belongs to a specific AWS Organization, **without enumerating account IDs at all**:

```json
{
  "Effect": "Allow",
  "Principal": "*",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::project-reports-bucket/*",
  "Condition": {
    "StringEquals": {
      "aws:PrincipalOrgID": "o-xxxxxxxxxx"
    }
  }
}
```

New accounts added to the org are automatically covered; removed accounts automatically lose access — zero policy maintenance. That's exactly "least operational overhead."

## Why the alternatives fall short

- **List every account ID explicitly in the bucket policy** — works today, but doesn't scale operationally: it requires manual updates on every membership change, and grows unwieldy as departments/accounts multiply.
- **Set up an S3 VPC endpoint with a bucket policy scoped to the endpoint** — solves *network path* restriction (only reachable from inside a VPC), not *organizational membership* restriction. Wrong tool for this requirement.
- **Use IAM users with individually attached policies per account** — massive overhead: you'd manage cross-account IAM roles/users per department instead of one condition key on one resource policy.

## Exam Tip

Whenever the exam says **"AWS Organizations"** + **"restrict access to only accounts within the org"** + **"least operational overhead,"** the answer is almost always `aws:PrincipalOrgID` (or its sibling `aws:PrincipalOrgPaths` for restricting to specific OUs). Contrast this with `aws:SourceIp` (network-based) or `aws:SourceVpce` (VPC-endpoint-based) condition keys, which restrict by *where the request comes from*, not *who the requester's account belongs to*.
