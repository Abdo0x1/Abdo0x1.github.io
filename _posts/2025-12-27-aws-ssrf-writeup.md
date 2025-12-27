---
title: AWS SSRF Exploitation
date: 2025-12-27 12:00:00 +0200
categories: [Cloud, Security]
tags: [aws, ssrf, ctf]
---

## Introduction

In this challenge, I exploited an SSRF vulnerability on an AWS EC2 instance to steal IAM credentials.

## Step 1: Finding the Vulnerability

I found a web app taking a URL input. I tested it with the AWS metadata IP.
The payload used was: `http://169.254.169.254/latest/meta-data/`

It worked! The server returned the directory listing.

## Step 2: Stealing the Keys

To get the sensitive data, I targeted the IAM security credentials endpoint.

1. First, I got the role name: ec2-prod-role
2. Then, I requested the keys using this link:

`http://169.254.169.254/latest/meta-data/iam/security-credentials/ec2-prod-role`

## Result

The server responded with the Access Key and Secret Key.

Code: Success
AccessKeyId: ASIAQ3EGUZ...
SecretAccessKey: EBvAKCmU...

## Fix

Enable IMDSv2 on your EC2 instances to prevent this attack.
