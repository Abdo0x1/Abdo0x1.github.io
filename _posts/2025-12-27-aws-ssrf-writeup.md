---
title: "AWS EC2 Exploitation: Stealing IAM Credentials via SSRF"
date: 2025-12-27 12:00:00 +0200
categories: [Cloud Security, AWS]
tags: [ctf, ssrf, aws, ec2, imds, red-teaming]
---

## Introduction

Server-Side Request Forgery (SSRF) remains one of the most critical vulnerabilities in cloud environments. In this write-up, I will demonstrate how I exploited an SSRF vulnerability in a web application hosted on AWS EC2 to access the **Instance Metadata Service (IMDS)** and exfiltrate sensitive **IAM Security Credentials**.

## Challenge Description

* **Target:** A web application hosted on an EC2 instance (`54.166.141.194`).
* **Objective:** Compromise the server and retrieve the hidden flag (Instance ID) and IAM keys.
* **Vulnerability:** The application processes user input to fetch data from external URLs without proper validation.

## Phase 1: Reconnaissance

I started by exploring the web application functionality. I identified a feature that accepts URLs as input. To test for SSRF, I attempted to force the server to make a request to the AWS-specific "Magic IP": `169.254.169.254`.

### What is 169.254.169.254?
This is a link-local address used by AWS to expose the **Instance Metadata Service (IMDS)**. It allows EC2 instances to retrieve information about themselves (IP, Region, Instance ID, and IAM Roles).

I injected the following payload into the vulnerable parameter:

```http
[http://169.254.169.254/latest/meta-data/](http://169.254.169.254/latest/meta-data/)
