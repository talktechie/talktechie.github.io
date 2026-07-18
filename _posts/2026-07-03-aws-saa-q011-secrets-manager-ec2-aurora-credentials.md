---
title: "Q11: Minimizing Operational Overhead of EC2-to-Aurora Credential Management"
date: 2026-07-03 08:00:00 +0530
categories: [AWS SAA, Design Secure Architectures]
tags: [aws, secrets-manager, iam, aurora, ec2, saa-c03, security]
description: "EC2 instances connect to an Aurora database using credentials stored locally in a file. The company wants to minimize operational overhead of credential management."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Secure Architectures |
| **Services** | AWS Secrets Manager, IAM, Amazon Aurora |
| **Difficulty** | Easy–Medium |

## Question

An application on EC2 instances connects to an **Aurora** database using **user names and passwords stored locally in a file**. The company wants to **minimize the operational overhead of credential management**.

What should a solutions architect do?

## Correct Answer

**Use AWS Secrets Manager. Attach an IAM role to the EC2 instances that grants access to the secret. Turn on automatic rotation.**

## Why this is correct

Credentials sitting in a plaintext local file are both a security risk and an operational burden — every time a password changes, someone has to manually update the file on every instance, and there's no audit trail.

**AWS Secrets Manager** centralizes storage of database credentials and — critically for "minimize operational overhead" — supports **automatic rotation** on a schedule, using a built-in Lambda rotation function for supported databases like Aurora. Once rotation is turned on, Secrets Manager handles generating a new password, updating it in Aurora, and updating the stored secret — with zero manual steps.

The EC2 instances retrieve the credential at runtime by calling the Secrets Manager API, authorized via an **IAM role attached to the instance** (using instance profile credentials, no hardcoded keys). The application never stores the password locally again, and always fetches the current, valid credential.

## Why the alternatives fall short

- **Store credentials in the EC2 user data / launch template** — still static, still requires manual updates on rotation, and user data is not a secure long-term secrets store (visible via instance metadata).
- **Use AWS Systems Manager Parameter Store (standard, without rotation)** — can securely store secrets, but does **not** provide native automatic rotation the way Secrets Manager does; you'd have to build your own rotation Lambda and scheduling, adding operational overhead.
- **Encrypt the local credentials file with KMS** — improves security at rest, but does nothing to reduce the *operational* burden of manually rotating and redistributing credentials — the core problem in the question.

## Exam Tip

**"Rotate credentials automatically" + "minimize operational overhead" → AWS Secrets Manager**, virtually every time it appears on the SAA exam. Parameter Store is the right call when you just need centralized, encrypted configuration/secret storage *without* a rotation requirement — Secrets Manager is the answer once "automatic rotation" enters the question.
