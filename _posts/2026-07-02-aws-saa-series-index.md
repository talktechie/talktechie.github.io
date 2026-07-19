---
title: "AWS SAA-C03 Series — Complete Question Index"
date: 2026-07-02 07:00:00 +0530
categories: [AWS SAA, Index]
tags: [aws, saa-c03, solutions-architect, certification, index]
description: "650+ AWS Certified Solutions Architect – Associate (SAA-C03) practice questions with detailed explanations, organized by exam domain. New questions added daily."
pin: true
---

A complete AWS Certified Solutions Architect – Associate (SAA-C03) question bank, explained the way I wish someone had explained it to me — not just "the answer is B," but *why* B beats A, C, and D, and which AWS service behavior the question is really testing.

This series is both a public study resource and my own SAA prep log. I add **10 new questions every day** until all 650+ are covered. Each question below links to its own post with the full scenario, correct answer, and a breakdown of the underlying AWS concept.

> 🟢 New here? Start with Domain 1 and work down — the domains roughly follow the weight they carry on the real exam.
{: .prompt-tip }

**Progress: 40 / 650+ questions published**

---

## 📋 Exam Domain Weighting (SAA-C03)

| Domain                                         | Weight |
| ---------------------------------------------- | ------ |
| Domain 1: Design Secure Architectures          | 30%    |
| Domain 2: Design Resilient Architectures       | 26%    |
| Domain 3: Design High-Performing Architectures | 24%    |
| Domain 4: Design Cost-Optimized Architectures  | 20%    |

---

## 🔐 Domain 1 — Design Secure Architectures

| #   | Question                                                               | Core Service                    | Link                                                                       |
| --- | ------------------------------------------------------------------------| -------------------------------- | --------------------------------------------------------------------------- |
| Q3  | Restrict an S3 bucket to only accounts inside an AWS Organization      | S3 + Organizations               | [Read more →](/posts/aws-saa-q003-s3-bucket-policy-aws-organizations/)      |
| Q4  | Give an EC2 instance private access to S3 with no internet             | VPC Gateway Endpoint             | [Read more →](/posts/aws-saa-q004-vpc-gateway-endpoint-s3-private-access/)  |
| Q11 | Minimize operational overhead of EC2-to-Aurora credential management   | Secrets Manager                  | [Read more →](/posts/aws-saa-q011-secrets-manager-ec2-aurora-credentials/)  |
| Q13 | Rotate RDS credentials across multiple Regions, least overhead         | Secrets Manager (multi-Region)   | [Read more →](/posts/aws-saa-q013-secrets-manager-multi-region-rotation/)   |
| Q15 | Replicate an on-prem inspection server's traffic filtering in AWS      | AWS Network Firewall             | [Read more →](/posts/aws-saa-q015-aws-network-firewall-vpc-inspection/)     |
| Q16 | Visualize a data lake with tiered access (management vs everyone else) | QuickSight                       | [Read more →](/posts/aws-saa-q016-quicksight-data-lake-visualization-access/) |
| Q17 | Let EC2 instances access an S3 bucket the right way                    | IAM Role                         | [Read more →](/posts/aws-saa-q017-iam-role-ec2-s3-access/)                  |
| Q19 | Route all inbound traffic through a third-party firewall appliance     | Gateway Load Balancer            | [Read more →](/posts/aws-saa-q019-gateway-load-balancer-appliance-inspection/) |
| Q26 | Catch unauthorized configuration changes on S3 buckets                 | AWS Config                       | [Read more →](/posts/aws-saa-q026-aws-config-s3-unauthorized-changes/)      |
| Q27 | Share a CloudWatch dashboard with a user who has no AWS account        | CloudWatch Dashboard Sharing     | [Read more →](/posts/aws-saa-q027-cloudwatch-dashboard-sharing-least-privilege/) |
| Q28 | Org-wide SSO while keeping on-prem Active Directory as the source      | AWS SSO + AD Trust               | [Read more →](/posts/aws-saa-q028-aws-sso-active-directory-trust/)          |
| Q31 | Detect untagged EC2/RDS/Redshift resources with minimal effort         | AWS Config                       | [Read more →](/posts/aws-saa-q031-detect-untagged-resources-aws-config/)    |
| Q34 | Track config changes + API call history for compliance                 | AWS Config + CloudTrail          | [Read more →](/posts/aws-saa-q034-track-config-changes-api-calls-config-cloudtrail/) |
| Q35 | Protect a public app behind an ELB from large-scale DDoS               | AWS Shield Advanced              | [Read more →](/posts/aws-saa-q035-protect-against-ddos-shield-advanced/)    |
| Q36 | Same customer managed KMS key across S3 buckets in two Regions         | AWS KMS multi-Region key         | [Read more →](/posts/aws-saa-q036-multi-region-kms-key-s3-encryption/)      |
| Q37 | Secure, repeatable remote EC2 access with least overhead               | Systems Manager Session Manager  | [Read more →](/posts/aws-saa-q037-secure-remote-ec2-access-session-manager/) |

## 🛡️ Domain 2 — Design Resilient Architectures

| #   | Question                                                                         | Core Service              | Link                                                                            |
| --- | -----------------------------------------------------------------------------------| -------------------------- | ---------------------------------------------------------------------------------|
| Q5  | Two EC2 instances behind an ALB show different documents                         | EFS                        | [Read more →](/posts/aws-saa-q005-efs-shared-storage-multi-az/)                 |
| Q7  | Fan out bursty messages (100k/sec) to many consumers                             | SNS + SQS                  | [Read more →](/posts/aws-saa-q007-sns-sqs-fanout-decoupling/)                   |
| Q8  | Modernize a primary/worker-node batch platform                                   | SQS + Auto Scaling         | [Read more →](/posts/aws-saa-q008-sqs-auto-scaling-compute-nodes/)              |
| Q10 | Process ecommerce orders strictly in received order                              | SQS FIFO + API Gateway     | [Read more →](/posts/aws-saa-q010-api-gateway-sqs-fifo-order-processing/)       |
| Q18 | Build a durable, stateless image-compression microservice (Choose Two)           | SQS + Lambda                | [Read more →](/posts/aws-saa-q018-sqs-lambda-image-processing-microservice/)    |
| Q29 | Route a multi-Region UDP VoIP service to the lowest-latency Region with failover | Global Accelerator + NLB    | [Read more →](/posts/aws-saa-q029-global-accelerator-nlb-voip-multi-region/)    |

## ⚡ Domain 3 — Design High-Performing Architectures

| #   | Question                                                          | Core Service                          | Link                                                                             |
| --- | ---------------------------------------------------------------------| -------------------------------------- | ----------------------------------------------------------------------------------|
| Q1  | Aggregate 500 GB/day from global sites into one S3 bucket fast    | S3 Transfer Acceleration               | [Read more →](/posts/aws-saa-q001-s3-transfer-acceleration-multi-region-upload/) |
| Q21 | Serverless flash-sale site handling millions of requests/hour     | S3 + CloudFront + Lambda + DynamoDB    | [Read more →](/posts/aws-saa-q021-serverless-flash-sale-site/)                   |
| Q25 | Fix a Lambda function hitting concurrency quotas under load       | Lambda + SQS (split functions)         | [Read more →](/posts/aws-saa-q025-split-lambda-sqs-aurora-scaling/)              |
| Q2  | Ad-hoc SQL queries on JSON logs in S3, least overhead              | Athena                                 | [Read more →](/posts/aws-saa-q002-athena-query-json-logs-s3/)                    |
| Q12 | Reduce latency for static and dynamic content globally             | CloudFront (multi-origin)              | [Read more →](/posts/aws-saa-q012-cloudfront-multi-origin-s3-alb/)               |
| Q14 | Auto-scale a read-heavy database while keeping high availability   | Aurora Auto Scaling                    | [Read more →](/posts/aws-saa-q014-aurora-multi-az-auto-scaling-replicas/)        |
| Q20 | Clone large EBS volumes fast with consistently high I/O            | EBS Fast Snapshot Restore              | [Read more →](/posts/aws-saa-q020-ebs-fast-snapshot-restore-cloning/)            |
| Q33 | Share millions of transactions in near real time, strip sensitive data | Kinesis Data Streams + Lambda + DynamoDB | [Read more →](/posts/aws-saa-q033-streaming-financial-transactions-kinesis-lambda-dynamodb/) |
| Q38 | Reduce global website latency cost-effectively                     | CloudFront + S3                        | [Read more →](/posts/aws-saa-q038-reduce-latency-static-website-cloudfront/)     |
| Q39 | Fix slow RDS inserts caused by a storage bottleneck                 | RDS Provisioned IOPS SSD               | [Read more →](/posts/aws-saa-q039-fix-rds-insert-performance-provisioned-iops/)  |

## 💰 Domain 4 — Design Cost-Optimized Architectures

| #   | Question                                                              | Core Service               | Link                                                                          |
| --- | -------------------------------------------------------------------------| --------------------------- | ---------------------------------------------------------------------------------|
| Q6  | Migrate 70 TB of NFS video files to S3, minimal bandwidth              | Snowball Edge                | [Read more →](/posts/aws-saa-q006-snowball-edge-on-premises-migration/)       |
| Q9  | Extend on-prem SMB storage with lifecycle management                   | S3 File Gateway              | [Read more →](/posts/aws-saa-q009-s3-file-gateway-lifecycle-policy/)          |
| Q22 | Store media files with unpredictable access patterns at minimum cost   | S3 Intelligent-Tiering       | [Read more →](/posts/aws-saa-q022-s3-intelligent-tiering-unpredictable-access/) |
| Q23 | Keep backup files forever at the lowest possible cost                  | S3 Glacier Deep Archive      | [Read more →](/posts/aws-saa-q023-s3-lifecycle-glacier-deep-archive/)         |
| Q24 | Diagnose an EC2 cost spike from unwanted vertical scaling              | Cost Explorer                | [Read more →](/posts/aws-saa-q024-cost-explorer-ec2-cost-analysis/)           |
| Q30 | Cut costs on a database used only 48 hours a month                     | RDS Snapshot + Terminate     | [Read more →](/posts/aws-saa-q030-rds-snapshot-terminate-cost-savings/)       |
| Q32 | Most cost-effective way to host a static internal website              | Amazon S3                    | [Read more →](/posts/aws-saa-q032-cheapest-static-website-hosting-s3/)        |
| Q40 | Ingest + archive 1 TB/day of IoT alerts, 14-day retention then Glacier | Kinesis Data Firehose + S3 Lifecycle | [Read more →](/posts/aws-saa-q040-cost-effective-iot-alert-ingestion-kinesis-firehose/) |

---

## How this series works

- Every post follows the same structure: **Question → Correct Answer → Why it's right → Why the distractors are tempting but wrong → Exam tip**.
- Questions are tagged by the AWS service(s) they exercise, so you can jump straight to `s3`, `sqs`, `vpc`, etc. via [Tags](/tags).
- I batch questions in groups of 10 as I work through my own SAA-C03 prep, so this index gets a fresh update daily — bookmark it rather than any single post.

Spotted an error or have a trickier variant of one of these questions? Open an issue on [GitHub](https://github.com/talktechie/talktechie.github.io) — corrections make this more useful for everyone studying alongside me.
