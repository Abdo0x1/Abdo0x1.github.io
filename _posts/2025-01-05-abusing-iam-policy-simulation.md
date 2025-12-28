---
title: Abusing IAM Policy Simulation to Identify Hidden AWS Permissions
date: 2025-01-05 22:00:00 +0200
categories: [Cloud, AWS, IAM]
tags: [aws, iam, simulateprincipalpolicy, s3, cloud-security, ctf]
---

## Overview

This challenge demonstrates how the AWS IAM permission  
`iam:SimulatePrincipalPolicy` can be abused to enumerate effective permissions
even when direct IAM policy inspection is restricted.

The objective was to identify the **permission attached to the authenticated user**,
which represents the challenge flag.

---

## Initial Access

AWS credentials were provided for the following user:

- **User Name:** DevAppUser  
- **User ARN:** `arn:aws:iam::058264439561:user/DevAppUser`

Identity confirmation was performed using:

```bash
aws sts get-caller-identity
Phase 1: IAM Restrictions
Direct IAM policy inspection was not permitted for the user:

iam:GetUserPolicy → implicitDeny

iam:ListUserPolicies → implicitDeny

iam:GetPolicy → implicitDeny

iam:GetPolicyVersion → implicitDeny

However, the user had the following permission:

makefile
iam:SimulatePrincipalPolicy
This permission allows simulation of effective permissions without executing
actual API actions.

Phase 2: Confirming Simulation Capability
The simulation permission was validated:


aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::058264439561:user/DevAppUser \
  --action-names iam:SimulatePrincipalPolicy
Result: allowed

Phase 3: Enumerating Effective Permissions
Wildcard expansion (e.g., s3:*) is not supported by IAM simulation, so
individual actions were tested.

The following S3 actions were identified as allowed:

s3:ListAllMyBuckets

s3:ListBucket

s3:GetObject

s3:GetObjectVersion

s3:GetBucketLocation

s3:GetBucketAcl

s3:GetBucketPolicyStatus

s3:GetEncryptionConfiguration

s3:GetBucketPublicAccessBlock

s3:ListBucketVersions

s3:ListBucketMultipartUploads

Each action was simulated to determine the source policy.

Example:

bash
Copy code
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::058264439561:user/DevAppUser \
  --action-names s3:GetObject \
  --output json
Simulation output revealed:

makefile
SourcePolicyId: AmazonS3ReadOnlyAccess
Phase 4: Flag Identification
The challenge description stated:

The flag is hidden in the permission attached with the user

Based on the simulation results:

No inline or custom policy content was readable

All allowed S3 actions mapped to a single AWS managed policy

No other effective permissions were discovered

Therefore, the permission attached to the user — and the challenge flag — is:

nginx
AmazonS3ReadOnlyAccess
Result
✅ Flag Successfully Identified

nginx
AmazonS3ReadOnlyAccess
Security Impact
Granting iam:SimulatePrincipalPolicy can expose sensitive authorization logic,
including:

Managed policy attachments

Effective permissions

Hidden access paths

This allows attackers to plan privilege escalation or data access without
triggering resource-level activity logs.

Mitigation
Restrict iam:SimulatePrincipalPolicy to security or audit-only roles

Avoid attaching broad managed policies unless strictly required

Enforce least-privilege principles across IAM roles and users

Monitor IAM simulation activity via AWS CloudTrail
