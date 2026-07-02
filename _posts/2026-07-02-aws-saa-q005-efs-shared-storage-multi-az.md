---
title: "Q5: Why Two EC2 Instances Behind a Load Balancer Show Different Files"
date: 2026-07-02 08:20:00 +0530
categories: [AWS SAA, Design Resilient Architectures]
tags: [aws, efs, ebs, ec2, alb, saa-c03, storage]
description: "After scaling a web app to two EC2 instances across Availability Zones behind an ALB, users only ever see half of their uploaded documents at a time."
hidden: true
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Resilient Architectures |
| **Services** | Amazon EFS, Amazon EBS, Application Load Balancer |
| **Difficulty** | Easy–Medium |

## Question

A web app originally ran on a single EC2 instance, storing user-uploaded documents on an **EBS volume**. For scalability and availability, the company added a second EC2 instance with its own EBS volume in another Availability Zone, placing both behind an **Application Load Balancer**. Afterward, users see only *some* of their documents each time they refresh — never all of them together.

What should fix this?

## Correct Answer

**Copy the data from both EBS volumes into Amazon EFS, and modify the application to read/write documents through EFS.**

## Why this is correct

The root cause is architectural, not a bug: **EBS volumes are attached to a single EC2 instance in a single Availability Zone** and are not shared storage. Each instance now has its own, independent copy of "the documents," and the ALB round-robins requests between them — so a user sees instance A's files on one request and instance B's files on the next.

**Amazon EFS** is a fully managed **NFS file system** that can be mounted concurrently by multiple EC2 instances across multiple Availability Zones, all reading and writing to the **same underlying file set** in real time. Once both instances mount the same EFS file system and the app writes new uploads there, every instance behind the ALB sees an identical, consistent view of the documents — the "some documents disappear on refresh" problem goes away entirely.

## Why the alternatives fall short

- **EBS Multi-Attach** — only works within a single Availability Zone and for a very limited volume/instance combination (and typically for cluster-aware filesystems); it doesn't span AZs the way this architecture requires, and isn't a general-purpose fix for arbitrary application file storage.
- **Take EBS snapshots and periodically sync between instances** — introduces replication lag; users would still see stale or inconsistent state between syncs, and it adds ongoing operational overhead instead of removing it.
- **Use sticky sessions on the ALB** — this would route the *same user* to the *same instance* every time, hiding the symptom, but a new user (or a session that expires) still only ever sees one instance's subset of files. It doesn't unify the data.

## Exam Tip

Whenever you see **"multiple EC2 instances need the same files / shared file storage across AZs,"** think **Amazon EFS**. Reserve **EBS** for single-instance block storage (like a boot volume or a database's local storage) and reach for **S3** instead of EFS when the access pattern is really "store and retrieve whole objects" rather than "a POSIX file system multiple servers mount at once."
