---
title: "Q28: Single Sign-On Across All Organization Accounts, Using On-Premises Active Directory"
date: 2026-07-04 08:35:00 +0530
categories: [AWS SAA, Design Secure Architectures]
tags: [aws, sso, active-directory, organizations, saa-c03, identity]
description: "A company managing multiple accounts via AWS Organizations needs single sign-on across all accounts, while continuing to manage users and groups in its self-managed on-premises Active Directory."
---

## Problem Info

| | |
|---|---|
| **Domain** | Design Secure Architectures |
| **Services** | AWS IAM Identity Center (AWS SSO), AWS Directory Service, AWS Organizations |
| **Difficulty** | Medium |

## Question

A company has multiple AWS accounts managed centrally through **AWS Organizations**. The security team wants **single sign-on (SSO)** across all accounts, while continuing to manage users/groups in its **on-premises self-managed Microsoft Active Directory**.

Which solution meets these requirements?

## Correct Answer

**Enable AWS Single Sign-On (now AWS IAM Identity Center). Create a one-way forest or domain trust connecting the on-premises Active Directory to AWS SSO, via AWS Directory Service for Microsoft Active Directory (AWS Managed Microsoft AD).**

## Why this is correct

Two requirements have to be satisfied together: **centralized SSO across every account in the Organization**, and **keep identity management in the existing on-premises AD** rather than recreating users in AWS.

**AWS SSO / IAM Identity Center** is the AWS service built to provide a single sign-on experience across all accounts within an AWS Organization, letting users log in once and access any account/role they're permitted to, without separate credentials per account. Critically, AWS SSO doesn't have to be its own identity source — it can be connected to an **external identity provider**, including **self-managed Active Directory**, via **AWS Directory Service for Microsoft Active Directory (AWS Managed Microsoft AD)** acting as the connector.

Setting up a **one-way trust** (forest or domain trust) between the on-premises AD and the AWS Managed Microsoft AD lets AWS SSO authenticate users against the existing on-premises directory in real time — the company keeps managing users and groups exactly where it always has, while gaining Organization-wide SSO on top.

## Why the alternatives fall short

- **Create IAM users manually in each AWS account matching the on-premises AD users** — completely defeats the goal of *not* re-managing identities in AWS; it duplicates user administration in two places and doesn't provide true single sign-on across accounts.
- **Use AWS Cognito with a SAML identity provider** — Cognito is designed primarily for *application-level* user authentication (e.g., a customer-facing app's sign-in), not for providing SSO access to the **AWS Management Console and accounts** within an Organization — wrong tool for this workforce-identity use case.
- **Migrate all users and groups into AWS Managed Microsoft AD as the primary directory** — solves SSO, but requires **migrating away from** the self-managed on-premises AD, directly violating "must continue managing users and groups in its on-premises self-managed Active Directory."

## Exam Tip

**"SSO across all Organization accounts" + "keep using on-premises self-managed Active Directory" → AWS SSO (IAM Identity Center) + AWS Managed Microsoft AD as a connector, joined via a one-way trust.** This differs from scenarios asking to **fully migrate** to AWS-managed identity (where AWS Managed Microsoft AD alone, without an on-prem trust, would be the answer) — the phrase "continue managing... on-premises" is the signal that a **trust relationship**, not a **migration**, is required.
