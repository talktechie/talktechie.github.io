---
title: "Q9: Extending On-Premises SMB Storage With Automatic File Lifecycle Management"
date: 2026-07-02 08:40:00 +0530
categories: [AWS SAA, Design Cost-Optimized Architectures]
tags: [aws, storage-gateway, s3, lifecycle-policy, saa-c03, hybrid]
description: "An SMB file server is running out of capacity, but the most recently created files need low-latency access while older files can move to cheaper storage automatically."
hidden: true
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Cost-Optimized Architectures |
| **Services** | AWS Storage Gateway (S3 File Gateway), S3 Lifecycle policies |
| **Difficulty** | Medium |

## Question

A company runs an **SMB file server on-premises**. Large files are accessed frequently for the first **7 days**, then rarely touched. Total data is close to filling the on-prem storage capacity. A solutions architect must **increase available storage** without losing **low-latency access to recently accessed files**, and must also provide **file lifecycle management** to prevent future storage issues.

Which solution meets these requirements?

## Correct Answer

**Deploy an Amazon S3 File Gateway to extend the on-premises storage into S3. Set an S3 Lifecycle policy to transition objects to S3 Glacier Deep Archive after 7 days.**

## Why this is correct

This is a **hybrid storage** problem with two distinct needs: low-latency access to *fresh* files, and automatic cost control for *stale* files.

**S3 File Gateway** (part of AWS Storage Gateway) presents an **SMB/NFS file share** on-premises, backed transparently by an S3 bucket in the cloud. Recently written/accessed files are cached locally on the gateway's storage, giving **low-latency access** exactly like the existing file server — while the actual objects live durably in S3, immediately increasing the effective capacity beyond what local disks alone could hold. This satisfies "increase available storage space without losing low-latency access."

Because every file written through the gateway becomes a real S3 object, you can attach a standard **S3 Lifecycle policy** to the bucket: after 7 days (matching the observed access pattern), objects automatically transition to **S3 Glacier Deep Archive** — the cheapest S3 storage class, appropriate for data that's rarely accessed. This is fully automated "file lifecycle management," requiring no ongoing manual intervention — directly solving the "avoid future storage issues" requirement as data keeps growing.

## Why the alternatives fall short

- **Just add more local disks / a bigger on-prem NAS** — solves capacity temporarily but doesn't address lifecycle management, and the company will hit the same ceiling again later; it's not a scalable fix.
- **Amazon FSx for Windows File Server** — a fully cloud-hosted SMB file system, great if the company wants to move the file server *entirely* to AWS, but the requirement here is to **extend** existing **on-premises** storage, not replace it outright, making File Gateway the closer fit for a hybrid extension.
- **Manually move old files to S3 via a scheduled script, then delete locally** — technically achievable, but it's custom-built lifecycle management the company has to write and maintain — the opposite of the fully managed automation an S3 Lifecycle policy provides "for free" once files are in S3.

## Exam Tip

**"On-premises file server, low-latency access to recent files, running low on capacity, needs automated tiering" → S3 File Gateway + S3 Lifecycle policy.** Remember Storage Gateway's three flavors for the exam: **File Gateway** (SMB/NFS share backed by S3), **Volume Gateway** (iSCSI block storage, cached or stored mode), and **Tape Gateway** (virtual tape library backed by S3/Glacier, for backup software that expects physical tapes).
