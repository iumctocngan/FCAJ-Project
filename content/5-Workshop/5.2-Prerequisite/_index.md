---
title: "Prerequisites"
date: 2026-08-04
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### Workshop Prerequisites

To complete the NutriVision workshop, prepare your local development environment and configure AWS CLI authentication as outlined below:

#### 1. Environment & Local Tool Requirements:
- AWS Account with necessary permissions to provision required AWS services (S3, ECR, Lambda, API Gateway, CloudWatch, SNS, Amplify, Rekognition).
- AWS CLI v2 installed on your local workstation.
- Docker Desktop installed and running locally to build and package Lambda container images.

---

#### 2. Configure & Verify AWS CLI

Open your Terminal or PowerShell and run the configuration command:
```bash
aws configure
```
- AWS Access Key ID: Enter your AWS Access Key.
- AWS Secret Access Key: Enter your Secret Access Key.
- Default region name: ap-southeast-1
- Default output format: json

Verify that your AWS CLI authentication is connected successfully:
```bash
aws sts get-caller-identity
```

![AWS CLI Verification](/FCAJ-Project/images/5-Workshop/5.2-Prerequisite/aws_cli_conf.png)