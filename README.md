# Enterprise Multi-Account AWS Governance using AWS Organizations & Service Control Policies

## Scenario

You have joined an enterprise company as a **Cloud Governance Engineer**. The organization has grown rapidly and now manages multiple AWS accounts for different teams:

* **Development Team**
* **Testing Team**
* **Production Team**

Currently, there are no restrictions across accounts. Developers are creating expensive resources in production accounts, and security policies are inconsistent.

**The CTO assigns you a task:**

*"Implement centralized governance across all AWS accounts to enforce security, cost, and compliance policies."*

---

## Project Overview

This project demonstrates how to implement **centralized governance for multiple AWS accounts** using **AWS Organizations**, **Organizational Units (OUs)**, and **Service Control Policies (SCPs)**.

Large organizations typically separate environments such as **Development, Testing, and Production** into different AWS accounts. Without governance controls, developers might accidentally create expensive resources or disable important security services.

This project implements policies to enforce **security, cost control, and compliance** across all AWS accounts.

---

# Architecture

![Architecture Diagram](architecture-diagram.svg)

```
AWS Organization
        │
       Root
        │
 ┌──────┼────────┐
 DEV    TEST     PROD
  │       │        │
Dev     Test      Prod
Account Account   Account
```

### Governance Policies

| OU / Level | Policy Applied          | Purpose                                        |
| ---------- | ----------------------- | ---------------------------------------------- |
| DEV OU     | Deny-Large-EC2-Dev      | Prevent launching expensive EC2 instances      |
| Root       | Deny-Disable-CloudTrail | Prevent disabling CloudTrail logging           |
| Root       | Restrict-Regions        | Restrict resource creation to approved regions |

---

# Technologies Used

* AWS Organizations
* Organizational Units (OUs)
* Service Control Policies (SCP)
* AWS IAM
* AWS CloudTrail

---

# Organizational Structure

![Organization Structure](01-organization-structure.png)

```
Root
 ├── DEV
 │     └── Dev-Account
 ├── TEST
 │     └── Test-Account
 └── PROD
       └── Prod-Account
```

Each environment is isolated using separate AWS accounts.

![Organizational Units](02-organizational-units.png)

---

# Implemented Service Control Policies

![Service Control Policies](03-scp-policies.png)

![Policy Attachments](04-policy-attachment.png)

## 1. Deny Large EC2 Instances (Cost Control)

This policy prevents developers from launching expensive EC2 instance types in the **Dev environment**.

Restricted instance types:

* m5.*
* c5.*
* r5.*

Purpose:

* Prevent accidental high-cost infrastructure creation.

Policy JSON:

```json
{
 "Version": "2012-10-17",
 "Statement": [
  {
   "Sid": "DenyLargeInstances",
   "Effect": "Deny",
   "Action": "ec2:RunInstances",
   "Resource": "*",
   "Condition": {
    "StringLike": {
     "ec2:InstanceType": [
      "m5.*",
      "c5.*",
      "r5.*"
     ]
    }
   }
  }
 ]
}
```

---

## 2. Prevent Disabling CloudTrail (Security Governance)

CloudTrail records all AWS API activities across accounts.

This policy prevents:

* Deleting CloudTrail
* Stopping CloudTrail logging

Purpose:

* Maintain security audit logs across the entire organization.

Policy JSON:

```json
{
 "Version": "2012-10-17",
 "Statement": [
  {
   "Sid": "PreventCloudTrailDisable",
   "Effect": "Deny",
   "Action": [
    "cloudtrail:StopLogging",
    "cloudtrail:DeleteTrail"
   ],
   "Resource": "*"
  }
 ]
}
```

---

## 3. Restrict AWS Regions (Compliance)

This policy ensures that AWS resources can only be created in approved regions.

Allowed regions:

* ap-south-1
* us-east-1

Purpose:

* Enforce compliance
* Reduce operational complexity
* Prevent resource creation in unauthorized regions

Policy JSON:

```json
{
 "Version": "2012-10-17",
 "Statement": [
  {
   "Sid": "RestrictRegions",
   "Effect": "Deny",
   "NotAction": [
    "iam:*",
    "organizations:*",
    "route53:*"
   ],
   "Resource": "*",
   "Condition": {
    "StringNotEquals": {
     "aws:RequestedRegion": [
      "ap-south-1",
      "us-east-1"
     ]
    }
   }
  }
 ]
}
```

---

# Validation Process

To validate the governance policies:

1. Login to a member account (Dev account).
2. Attempt to launch a restricted EC2 instance such as **m5.large**.
3. Attempt to disable CloudTrail logging.
4. Attempt to create resources in a restricted region.

Expected Result:

```
AccessDenied
```

This confirms that the **Service Control Policies are successfully enforced**.

### Validation Screenshots

![EC2 Access Denied](05-access-denied-ec2.png)

![CloudTrail Protection](06-cloudtrail-protection.png)

---

# Governance Benefits

* Centralized governance for multiple AWS accounts
* Prevents misuse of cloud infrastructure
* Improves security and audit visibility
* Enforces compliance policies
* Reduces operational risk

---

# Key Learning Outcomes

* Implement AWS multi-account architecture
* Use Organizational Units for account grouping
* Apply Service Control Policies for governance
* Enforce security and cost management policies
* Understand enterprise cloud governance models

---

# Possible Interview Questions

### What is AWS Organizations?

AWS Organizations is a service that allows centralized management of multiple AWS accounts.

---

### What are Organizational Units (OUs)?

OUs are logical containers used to group AWS accounts and apply policies collectively.

---

### What is a Service Control Policy (SCP)?

SCPs define the **maximum permissions available** for accounts in an AWS Organization.

---

### Do SCPs apply to the management account?

No. SCPs apply only to **member accounts**, not the management account.

---

### Difference between IAM Policy and SCP

| IAM Policy             | SCP                           |
| ---------------------- | ----------------------------- |
| Grants permissions     | Restricts maximum permissions |
| Applied to users/roles | Applied to AWS accounts       |

---

# Conclusion

This project demonstrates how organizations implement **centralized governance using AWS Organizations and Service Control Policies**. The solution ensures strong security, cost control, and compliance across multiple AWS environments.

---

# Project Structure

```
Multi-Account-AWS-Governance/
│
├── README.md
├── architecture-diagram.svg
│
├── policies/
│   ├── deny-large-ec2-dev.json
│   ├── prevent-cloudtrail-disable.json
│   └── restrict-regions.json
│
├── 01-organization-structure.png
├── 02-organizational-units.png
├── 03-scp-policies.png
├── 04-policy-attachment.png
├── 05-access-denied-ec2.png
└── 06-cloudtrail-protection.png
```

---

# Deliverables

✔ SCP JSON Policies
✔ Organizational Unit Structure
✔ Service Control Policy Attachments
✔ Governance Validation Proof
✔ Complete Project Documentation

---
