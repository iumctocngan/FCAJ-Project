---
title: "Workshop Overview"
date: 2026-08-04
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

### 1. Objectives
Execute a complete deployment workflow migrating a Computer Vision model from research (Google Colab) to AWS Serverless Production infrastructure. Participants will practice model training, ONNX serialization, static Web UI hosting via AWS Amplify, Serverless REST API construction, AI Fallback integration, and automated monitoring and resource cleanup.

---

### 2. Solution Overview
NutriVision automatically recognizes food dishes and computes calories alongside macronutrient profiles (Protein, Carbs, Fat, Fiber) from meal photos:
- Web Application: Hosts the frontend UI on AWS Amplify Hosting with managed HTTPS domains and GitHub CI/CD integration.
- Performance and Cost: Lightweight ONNX model (15.5 MB) running on AWS Lambda executes rapidly with minimal operational cost via pay-per-use Serverless billing.
- AI Fallback (Amazon Rekognition): When primary model confidence drops below 60% due to poor image conditions, the system triggers Amazon Rekognition to map general labels to the 50 defined dishes. If unmapped, it returns HTTP 422 rather than guessing incorrectly. Low-confidence images are logged to S3 for retraining.
- Monitoring: Automatically dispatches incident alert emails via Amazon SNS upon detecting system errors.

---

### 3. Architecture & Operational Workflow

![NutriVision Architecture Diagram](/FCAJ-Project/images/2-Proposal/solution_architecture.drawio.png)

The system operates across 4 core flow groups:

#### A. Packaging and Deployment
1. Package and Push Docker Image: Developer packages inference code and the ONNX model into a Docker image, then pushes it to Amazon ECR.
2. Initialize Lambda Function: AWS Lambda pulls the Container Image from ECR to serve inference directly.
3. Master Artifact Storage: Upload model weights food_model.onnx (15.5 MB) and calorie_map.json to Amazon S3 as the centralized store.

#### B. User Lifecycle and Runtime Inference
4. Access Application: Users access the web application via HTTPS endpoints distributed by AWS Amplify Hosting.
5. Submit Request: The web UI submits meal images and portion scale factors (0.7x – 2.0x) via POST /predict requests.
6. API Routing: API Gateway handles CORS, enforces Rate Limiting (20 req/s) against abuse, and routes the payload to Lambda.
7. Model Inference: Lambda decodes images, validates image quality, and runs inference via the EfficientNet-B0 ONNX model.

#### C. Decision and Fallback Flow
8. Primary Flow: For confidence 60% or higher, Lambda calculates portion-scaled nutrition and returns JSON to the web UI.
9. Fallback Flow: For confidence below 60%, Lambda triggers Rekognition label detection to verify the dish; if unmapped, returns HTTP 422 and stores the image to S3 for offline review.

#### D. Observability and Alerting
10. Ingest Logs and Metrics: AWS Lambda logs execution details and operational metrics to CloudWatch.
11. Incident Alerts: When errors exceed thresholds, CloudWatch triggers Amazon SNS to send email alerts to engineers.

---

### 4. AWS Services Utilized

| AWS Service | Role |
|---|---|
| AWS Amplify | Static web hosting (Frontend) with GitHub CI/CD, CloudFront CDN, and SSL HTTPS. |
| Amazon API Gateway | Manages REST API, CORS configuration, and Rate Limiting (20 req/s). |
| AWS Lambda | Serverless compute (Python 3.12, ONNX Runtime), inference, and portion scaling. |
| Amazon ECR | Stores and manages Docker Container Images for AWS Lambda. |
| Amazon S3 | Stores ONNX model (15.5 MB), calorie_map.json, and audit images. |
| Amazon Rekognition | Fallback AI engine when primary model confidence is below 60%. |
| Amazon CloudWatch | Monitors latency, manages log groups, and configures alarms. |
| Amazon SNS | Sends real-time incident notification emails to administrators. |

---

### 5. Expected Outcomes
1. Deploy a Serverless application on AWS with Amplify frontend and API Gateway handling requests.
2. Practice PyTorch to ONNX serialization (15.5 MB) and lightweight containerized serverless runtimes.
3. Validate test scenarios: classify 50 food dishes, trigger fallback Rekognition, and handle low-quality image inputs (HTTP 422).
4. Configure SNS email alerting and cleanly tear down resources after workshop completion.