---
title: "Q1: Fastest Way to Aggregate Multi-Continent Data into One S3 Bucket"
date: 2026-07-02 08:00:00 +0530
categories: [AWS SAA, Design High-Performing Architectures]
tags: [aws, s3, transfer-acceleration, saa-c03, storage]
description: "A company collects 500 GB/day per site from cities across continents and needs it aggregated into a single S3 bucket as fast as possible with minimal operational complexity."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design High-Performing Architectures |
| **Services** | Amazon S3, S3 Transfer Acceleration |
| **Difficulty** | Easy–Medium |

## Question

A company collects data for temperature, humidity, and atmospheric pressure in cities across multiple continents. Each site generates about 500 GB of data daily and has a high-speed internet connection. The company wants to aggregate the data from every global site into a **single Amazon S3 bucket as quickly as possible**, while keeping operational complexity to a minimum.

Which solution meets these requirements?

## Correct Answer

**Turn on S3 Transfer Acceleration on the destination S3 bucket, and use multipart uploads to send site data directly to it.**

## Why this is correct

Break the requirements down:

- **Global sites, high-speed links** → the bottleneck isn't local bandwidth, it's the long round-trip distance to a single regional S3 endpoint.
- **"As fast as possible"** → rules out slow/manual options like physical shipping.
- **"Minimize operational complexity"** → rules out building custom relay infrastructure.

**S3 Transfer Acceleration** solves exactly this. It routes uploads through the nearest **CloudFront edge location** and carries them over Amazon's optimized backbone network to the bucket's region, instead of over the public internet the whole way. AWS advertises accelerated transfers as **50–500% faster** than a standard upload for long-distance transfers. You enable it with one setting on the bucket — no extra servers, no custom software — which is exactly the "least operational complexity" signal in the question.

Pairing it with **multipart upload** lets each 500 GB daily payload be split into parallel parts, uploaded concurrently, and reassembled in the bucket — improving throughput and resilience to individual part failures (only a failed part needs retrying, not the whole object).

## Why the alternatives fall short

- **Snowball / physical transfer devices** — great for one-time bulk migrations (see Q6), but hopeless for a recurring daily 500 GB feed from many sites; the round-trip time defeats "as fast as possible."
- **Setting up Direct Connect per site** — technically fast, but wildly high operational and cost overhead for dozens of remote sites; violates "minimize operational complexity."
- **Uploading directly to S3 without acceleration** — works, but over long intercontinental distances the standard TCP path is slower and more variable than the accelerated path, especially for large daily payloads.

## Exam Tip

Whenever you see **"long-distance," "multiple continents/regions,"** or **"partners uploading to our bucket"** combined with **"speed up"**, think **S3 Transfer Acceleration**. If the scenario instead says **"one-time," "petabytes," "no network," or "limited bandwidth,"** think **Snowball/Snowmobile**. Multipart upload is a complementary technique, not a competing answer — it's almost always the right pairing for any large object upload to S3.
