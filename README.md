---
lang: en
title: S3 + Route 53 Infrastructure Blueprint
---

# Project: Agent-Managed Static Infrastructure

This document details the configuration for hosting a high-performance
static site using **S3** and **Route 53**, with local execution rights
granted to **OpenClaw**.

## 1. Bucket Configuration

  Bucket Name         Type         Function
  ------------------- ------------ --------------------------------------------
  `example.com`       Main Host    Storage for `index.html`, CSS, and Assets.
  `www.example.com`   Redirector   301 Redirect to `example.com`.

### Essential Bucket Settings

- **Static Website Hosting:** Enabled on both. For the main bucket, set
  index document to `index.html`.
- **Public Access:** \"Block all public access\" must be **OFF** for the
  main bucket.
- **Bucket Policy:** Applied to main bucket to allow `s3:GetObject` for
  everyone (`"Principal": "*"`).

## 2. Route 53 DNS Configuration

To point your domain to the S3 buckets, we use **Alias Records**. These
are free and ensure that your domain stays mapped even if AWS changes
the underlying IP addresses.

    Record Type: A
    Name: [blank] or @
    Value: Alias to S3 Website Endpoint
    Target: s3-website-us-east-1.amazonaws.com (Matches your bucket region)

::: note
**Important:** The bucket name [must]{.underline} match the domain name
exactly for Route 53 to recognize the S3 bucket as a valid Alias target.
:::

## 3. OpenClaw Agent Integration

Instead of manual uploads, the agent manages the site via the **AWS
CLI**. Since the CLI is configured locally on Ubuntu, the agent uses the
`exec` tool for deployments.

### Deployment Workflow

1.  Agent generates/updates web content in a local directory.
2.  Agent executes: `aws s3 sync . s3://example.com/ --delete`
3.  Website is live instantly.

## 4. Technical Validation

To verify the entire chain is working, run these checks:

- **CLI Check:** `aws s3 ls s3://example.com` (Should show your files).
- **DNS Check:** `dig +short example.com` (Should return AWS IP
  addresses).
- **HTTP Check:** `curl -I http://example.com` (Should return 200 OK).

Last updated: 2026-03-15 \| System: Ubuntu + OpenClaw + AWS CLI
