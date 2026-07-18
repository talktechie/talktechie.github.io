---
title: "Q24: Diagnosing an EC2 Cost Spike Caused by Unwanted Vertical Scaling"
date: 2026-07-04 08:15:00 +0530
categories: [AWS SAA, Design Cost-Optimized Architectures]
tags: [aws, cost-explorer, ec2, saa-c03, cost-optimization, billing]
description: "EC2 costs increased due to unwanted vertical scaling of some instance types. The architect needs a graph comparing the last two months of costs, with an in-depth root-cause analysis."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Cost-Optimized Architectures |
| **Services** | AWS Cost Explorer |
| **Difficulty** | Easy |

## Question

EC2 costs increased on the latest bill, traced to **unwanted vertical scaling** of instance types for a couple of EC2 instances. A solutions architect needs a **graph comparing the last 2 months** of EC2 costs and an **in-depth analysis** to find the root cause.

How should the architect do this with the **least operational overhead**?

## Correct Answer

**Use AWS Cost Explorer's granular filtering feature to analyze EC2 costs broken down by instance type.**

## Why this is correct

**AWS Cost Explorer** is purpose-built for exactly this kind of investigation: visualizing historical cost and usage data, with filtering and grouping options that let you drill into *why* a bill changed.

For this scenario, Cost Explorer lets the architect:
- **Filter by service (EC2)** and **group by instance type**, immediately surfacing which instance types' costs grew month-over-month.
- **Compare custom date ranges** (the "last 2 months") directly in the built-in graph view — no extra tooling needed.
- Drill further by **usage type**, revealing whether the cost increase came from larger instance sizes running the same hours (confirming the "vertical scaling" suspicion) versus simply running more hours.

All of this is available directly in the Cost Explorer console with a few clicks — no data export, no custom dashboards to build, no extra service to configure — which is exactly "least operational overhead."

## Why the alternatives fall short

- **Export Cost and Usage Reports (CUR) to S3 and analyze with Athena/QuickSight** — powerful for deep, custom, large-scale cost analytics, but building the pipeline (S3 bucket, Glue/Athena tables, QuickSight dashboards) is significant setup effort for what Cost Explorer already provides out of the box for this level of analysis.
- **Set up AWS Budgets with an alert threshold** — Budgets is for proactively *alerting* when spend crosses a threshold, not for retrospectively analyzing *why* costs changed with a breakdown by instance type — different tool, different job.
- **Enable detailed CloudWatch billing alarms** — CloudWatch billing alarms can notify on total estimated charges crossing a limit, but they don't provide the graphical, filterable, root-cause cost breakdown by dimension (like instance type) that Cost Explorer offers.

## Exam Tip

**"Analyze/visualize historical AWS costs," "compare costs over time," "identify what's driving a cost increase," with "least operational overhead" → AWS Cost Explorer.** Reserve **AWS Budgets** for proactive threshold alerting, and **Cost and Usage Reports + Athena/QuickSight** for scenarios explicitly requiring large-scale custom analytics beyond what Cost Explorer's built-in views support.
