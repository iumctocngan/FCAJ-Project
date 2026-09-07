---
title: "Workshop Overview"
date: 2026-08-04
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

### 1. Objectives
Execute an End-to-End deployment workflow migrating a Computer Vision / Machine Learning model from research (Google Colab) to AWS Serverless Production infrastructure. Participants will practice model training, ONNX serialization, static Web UI hosting via AWS Amplify, Serverless REST API construction, AI Fallback integration, and automated monitoring/resource cleanup.

---

### 2. Solution Overview
NutriVision automatically recognizes food dishes and computes calories alongside macronutrient profiles (Protein, Carbs, Fat, Fiber) from meal photos:
* **24/7 Web Application**: Hosts the frontend UI (HTML/CSS/JS) on AWS Amplify Hosting with managed HTTPS domains and GitHub CI/CD integration.
* **Performance & Cost**: Lightweight ONNX model (15.5 MB) running on AWS Lambda executes rapidly with optimized operational costs via pay-per-use Serverless billing.
* **AI Fallback (Amazon Rekognition)**: When primary ONNX model confidence drops below 60% due to poor image conditions, the system triggers Amazon Rekognition to map general labels to the 50 defined dishes. If unmapped, it intentionally returns HTTP 422 rather than guessing incorrectly. All low-confidence images are automatically logged to S3 ood_logs/ for retraining.
* **Real-time Monitoring**: Automatically dispatches incident alert emails via Amazon SNS upon detecting errors or budget overruns.

---

### 3. Architecture & Operational Workflow

![NutriVision Architecture Diagram](/FCAJ-Project/images/2-Proposal/solution_architecture.drawio.png)

The system operates across 4 core flow groups matching the architecture diagram:

#### A. Packaging & Deployment Flow:
1. **Dev ➔ AWS ECR**: Developer packages application inference code and the ONNX model into a Docker image, then pushes it to Amazon ECR.
2. **AWS ECR ➔ AWS Lambda**: AWS Lambda pulls and initializes the Container Image from ECR for high-performance direct inference and optimized cold start.
3. **Dev ➔ Amazon S3**: Developer uploads master artifacts food_model.onnx (15.5 MB) and calorie_map.json to Amazon S3 as the centralized Master Store.

#### B. User Lifecycle & Runtime Inference Flow:
4. **User ➔ AWS Amplify**: User accesses the web application via secure HTTPS endpoints distributed by AWS Amplify Hosting.
5. **AWS Amplify ➔ Amazon API Gateway**: Web UI submits meal images and portion scale factors (0.7x – 2.0x) via POST /predict requests.
6. **Amazon API Gateway ➔ AWS Lambda**: API Gateway terminates CORS, enforces Rate Limiting (20 req/s, Burst 40) against abuse, and routes the payload to Lambda.
7. **AWS Lambda (Inference Engine)**: Decodes images, performs Image Quality Checks (brightness, blur) ➔ constructs input tensors and runs inference via EfficientNet-B0 ONNX.

#### C. Decision, Fallback & Logging Flow:
8. **Primary Flow (Confidence ≥ 60%)**: Lambda calculates portion-scaled Calories, Protein, Carbs, and Fat, returning structured JSON to the Web UI.
9. **Fallback Flow (Confidence < 60%)**:
   - **Lambda ➔ Amazon Rekognition**: Triggers Rekognition label detection (returns mapped dish if keyword matches; otherwise returns HTTP 422).
   - **Lambda ➔ Amazon S3**: Automatically stores low-confidence images to s3://.../ood_logs/ tagged with confidence metrics for offline evaluation.

#### D. Observability & Real-Time Alerting Flow:
10. **AWS Lambda ➔ Amazon CloudWatch**: Ingests execution logs (Log Streams) and operational metrics (Latency, Invocations, Errors).
11. **Amazon CloudWatch ➔ Amazon SNS ➔ Email**: When errors exceed configured thresholds (Errors >= 1), CloudWatch triggers Amazon SNS to send real-time incident alert emails to engineers.

---

### 4. AWS Services Utilized (8 Core Services)

| AWS Service | Role |
|---|---|
| AWS Amplify | Static web hosting (Frontend) with GitHub CI/CD, CloudFront CDN & SSL HTTPS. |
| Amazon API Gateway | Manages REST API, CORS, authentication & Rate Limiting (20 req/s). |
| AWS Lambda | Serverless compute (Python 3.12 + ONNX Runtime), inference & portion scaling. |
| Amazon ECR | Stores and manages Docker Container Images for AWS Lambda. |
| Amazon S3 | Stores ONNX model (15.5 MB), calorie_map.json, and ood_logs/ images. |
| Amazon Rekognition | Fallback AI engine when primary model confidence < 60%. |
| Amazon CloudWatch | Monitors P95 Latency, Log Groups, and alarms. |
| Amazon SNS | Sends real-time incident notification emails to engineers. |

---

### 5. Expected Outcomes
1. **LIVE Serverless Deployment**: Deploy a production-ready AI application on AWS with Amplify frontend and API Gateway handling live traffic.
2. **MLOps Mastery**: Master PyTorch to ONNX serialization (15.5 MB) and lightweight containerized serverless runtimes.
3. **Validate Test Scenarios**: Classify 50 food dishes, trigger Fallback Rekognition, and handle low-quality image edge cases (HTTP 422).
4. **Monitoring & Cleanup**: Configure SNS email alerting and cleanly tear down resources after workshop completion.