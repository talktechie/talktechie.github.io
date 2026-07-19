---
title: "Q36: Encrypt S3 Data in Two Regions with One Multi-Region KMS Key"
date: 2026-07-19 08:00 +0530
categories: [AWS SAA, Design Secure Architectures]
tags: [aws, kms, s3, encryption, saa-c03]
description: "An application storing data in S3 buckets across two Regions must encrypt and decrypt everything with the exact same customer managed KMS key, with the least operational overhead."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Secure Architectures |
| **Services** | AWS KMS, Amazon S3, S3 Replication |
| **Difficulty** | Medium |

## Question

A company is building an application in AWS that will store data in Amazon S3 buckets in two AWS Regions. The company must use an AWS KMS customer managed key to encrypt all data stored in the S3 buckets. The data in both S3 buckets must be encrypted and decrypted with the **same** KMS key, and both the data and the key must exist in each of the two Regions. Which solution meets these requirements with the least operational overhead?

## Correct Answer

**Create a customer managed multi-Region KMS key, create an S3 bucket in each Region, configure replication between the buckets, and have the application use the multi-Region KMS key with client-side encryption.**

## Why this is correct

Standard AWS KMS keys are strictly regional — a customer managed key created in one Region cannot be used to encrypt or decrypt data in another Region, and each Region would normally need its own independent key. That directly conflicts with the requirement that both S3 buckets use the *exact same* key. AWS KMS multi-Region keys solve this: they are a set of KMS keys in different Regions that share the same key material and key ID, so an object encrypted with the primary key in one Region can be decrypted with its replica key in another Region — genuinely the "same" key, not just a similarly configured one.

Because S3 server-side encryption with SSE-KMS replicates by having the destination bucket re-encrypt objects with a KMS key local to that destination Region (which would be a *different* key unless multi-Region keys are used), the requirement to use the *same* key on both sides pushes this scenario toward **client-side encryption**: the application itself encrypts data using the multi-Region KMS key before uploading it to S3, so the same key material is consistently applied regardless of which Region's bucket receives the object, and S3 Cross-Region Replication simply copies the already-encrypted object.

This approach satisfies "least operational overhead" because AWS manages the underlying key replication and synchronization for multi-Region keys automatically — there's no need to build custom cross-Region key management, re-encryption logic, or key rotation coordination between two independent keys.

## Why the alternatives fall short

- **SSE-S3 with S3-managed keys (with or without a customer managed KMS key in the mix)** — SSE-S3 doesn't use a customer managed KMS key at all, so it fails the core requirement outright regardless of replication setup.
- **A single-Region customer managed key with SSE-KMS and S3 replication** — SSE-KMS replication re-encrypts objects with a destination-Region KMS key by default, meaning the data in each Region ends up encrypted by two different regional keys, not the same key.
- **Two independently created customer managed KMS keys, one per Region** — even if manually kept in sync, they are different key resources with different key IDs, violating the "same KMS key" requirement and adding manual key-parity overhead.

## Exam Tip

Whenever a question demands the **same customer managed KMS key** be usable for encrypt/decrypt across **multiple Regions**, look for **multi-Region KMS keys** — a single-Region key can never satisfy that requirement, no matter how replication is configured.
