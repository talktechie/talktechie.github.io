---
title: "Q37: Secure, Keyless EC2 Access with Systems Manager Session Manager"
date: 2026-07-19 08:05 +0530
categories: [AWS SAA, Design Secure Architectures]
tags: [aws, systems-manager, ec2, iam, saa-c03]
description: "A company needs a repeatable, low-overhead, native-AWS way to securely access and administer many EC2 instances remotely."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Secure Architectures |
| **Services** | AWS Systems Manager Session Manager, IAM |
| **Difficulty** | Easy–Medium |

## Question

A company recently launched a variety of new workloads on EC2 instances. It needs a strategy to access and administer these instances remotely and securely, using a repeatable process built on native AWS services that follows the AWS Well-Architected Framework. Which solution meets these requirements with the least operational overhead?

## Correct Answer

**Attach an appropriate IAM role to each existing and new EC2 instance, and use AWS Systems Manager Session Manager to establish remote sessions.**

## Why this is correct

Session Manager lets administrators open an interactive shell (browser-based or via the AWS CLI) to an EC2 instance without opening any inbound ports, maintaining bastion hosts, or distributing and rotating SSH keys. Access is governed entirely through IAM — the instance needs an IAM role with the SSM permissions, and the user needs IAM permissions to start a session — making it a natively repeatable, policy-driven process across both existing and future instances.

Because there's no bastion fleet to patch, no security group rules to open for SSH/RDP, and no key material to manage, this is the lowest-operational-overhead option available while still being fully auditable: every session can be logged to CloudWatch Logs or S3, satisfying the security and governance intent behind the Well-Architected Framework.

Attaching the IAM role uniformly to instances also makes the approach genuinely repeatable — new workloads inherit the same access model automatically as long as the role is part of the standard instance launch configuration.

## Why the alternatives fall short

- **Bastion host with SSH key pairs** — requires managing and rotating SSH keys, patching and hardening the bastion itself, and keeping an inbound SSH port open — all extra operational burden Session Manager removes.
- **EC2 Instance Connect** — still relies on opening an SSH port temporarily and managing security group rules per instance, adding more moving parts than a role-based Session Manager approach.
- **A self-managed VPN into the VPC** — requires building and operating additional networking infrastructure, which directly conflicts with the "least operational overhead" and "native AWS services" requirements.

## Exam Tip

"Remote access" + "least operational overhead" + "no open ports/keys to manage" is the classic signature for **Systems Manager Session Manager** paired with an **IAM instance role** — not a bastion host.
