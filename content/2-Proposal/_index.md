---
title: "Proposal"
date: 2026-08-03
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# AI NutriVision
## Automated Food Recognition and Nutrition Analysis System on AWS Serverless Infrastructure

### 1. Executive Summary
The **AI NutriVision** project addresses daily dietary tracking challenges by automating food recognition and calorie estimation along with macronutrient metrics (Protein, Carbs, Fat, Fiber) directly from meal photos.

The model is trained on the benchmark **Food-101** dataset (comprising 101,000 images across 101 food categories). To optimize for real-world nutrition tracking, the project performed data cleaning to curate a high-density subset of the **50 most popular food categories** (totaling 50,000 images). The fine-tuned EfficientNet-B0 deep learning model achieves a **Test Top-1 Accuracy of 85.62%** (Weighted Average F1-score of 0.86) across 5,000 independent test images.

The model is compressed into static ONNX format (15.5 MB) and operates entirely on AWS Serverless architecture (API Gateway, Lambda, S3, DynamoDB, Rekognition, CloudWatch). This solution reduces nutrition logging time from 5 minutes down to under 3 seconds per meal, with a warm Lambda execution duration of **35–50 ms** (total end-to-end web response latency < 150 ms) and superior cost optimization compared to traditional servers.

### 2. Problem Statement
#### Real-world Challenge
Calculating daily calories and macronutrient intake (Protein, Carbs, Fat) currently relies heavily on manual logging applications (such as MyFitnessPal, Yazio). Users must manually type food names, estimate portion weights, and search database entries. This multi-step process is tedious and frequently leads to user abandonment within days.

#### Proposed Solution
**AI NutriVision** provides a web interface where users simply capture or upload a meal image. The image is routed via API Gateway to AWS Lambda, where the ONNX model identifies the food dish with a warm compute duration of 35–50ms and returns a detailed nutritional breakdown.
- **Flexible Fallback AI Mechanism**: To handle out-of-distribution (OOD) dishes outside the 50 trained classes, the system automatically integrates **Amazon Rekognition** whenever the primary model confidence falls below 60%. Rekognition returns general labels (e.g., "Dish", "Noodle", "Soup"), which Lambda fuzzy-matches against `calorie_map.json` or assigns default macro estimates (`general_food`), while logging the image to `s3://.../ood_logs/` for future retraining.
- **Dynamic Portion Scaling**: The system allows users to select portion size multipliers directly on the Web UI (Small 0.7x, Medium 1.0x, Large 1.5x, Special 2.0x), enabling Lambda to automatically scale calories and macros based on actual consumption.
- Recognition history and nutrition metrics are automatically persisted into **Amazon DynamoDB** to build a personal food diary.

#### Benefits and Return on Investment (ROI)
- **Time Optimization**: Reduces daily logging time by 90% for gym-goers, dieters, or patients tracking nutrition with a single photo capture.
- **Cost Efficiency**: Serverless architecture incurs costs strictly per actual request without 24/7 server maintenance fees, maximizing operational budget optimization.

### 3. Solution Architecture
![AI NutriVision Architecture Diagram](/FCAJ-Project/images/2-Proposal/architecture.png)

#### AWS Services Used:
1. **Amazon API Gateway**: Receives HTTPS `POST /predict` requests from the Web UI, authenticates API Keys, manages CORS configurations, protects against DDoS, and enforces Rate Limiting (20 req/s).
2. **AWS Lambda**: Serverless compute function running Python 3.11, responsible for Base64 image decoding, blur/dark quality checks, running ONNX Runtime inference, and scaling calories by user-selected portion size (0.7x – 2.0x).
3. **Amazon S3**: Central storage for `food_model.onnx` (15.5 MB), nutritional lookup database `calorie_map.json`, and out-of-distribution image logs (`ood_logs/`). Uses S3 Lifecycle Policy to automatically expire logs after 90 days.
4. **Amazon Rekognition**: Managed AI service serving as a Fallback Engine. When the primary ONNX model confidence is < 60%, Lambda triggers Rekognition for general label detection, avoiding system deadlocks.
5. **Amazon DynamoDB**: Serverless NoSQL database storing prediction records (`prediction_id`, `timestamp`, `food_class`, `confidence`, `calories`, `macronutrients`).
6. **Amazon CloudWatch**: Monitors Lambda performance, API latency (P95 Latency), error rates, and triggers alarms during unexpected failures.

### 4. Technical Implementation & MLOps Optimization
Project implementation was executed in 2 core phases:

#### Phase 1: Data Cleaning, AI Model Training & Compression (Google Colab GPU)
- **Data Selection & Cleaning**: The raw Food-101 dataset contains 101,000 images with some label noise. The project conducted manual data cleaning combined with noise detection algorithms to select the 50 most popular food categories based on 3 scientific criteria:
  1. *Consumption Frequency*: Prioritizing widely consumed dishes in Asian and Western diets (e.g., Pho, Fried Rice, Pizza, Sushi, Hamburger, Steak, Salads, etc.).
  2. *Nutritional Complexity*: Selecting dishes with diverse macronutrient structures (Protein, Carbs, Fat) requiring strict caloric monitoring.
  3. *Redundancy Elimination*: Filtering out niche regional dishes or items with low lookup demand in fitness tracking applications.
  The resulting 50-class dataset (50,000 images) was partitioned into standard splits: 37,500 training images (750/class), 7,500 validation images (150/class), and 5,000 independent test images (100/class).
- **Training & Evaluation**: Utilized an `EfficientNet-B0` backbone with a 2-stage fine-tuning workflow (Phase 1: Freeze Backbone for 3 epochs; Phase 2: Full Unfreeze with Cosine Annealing scheduler for 7 epochs). Results achieved **Test Top-1 Accuracy of 85.62%** and a Weighted Average F1-score of 0.86.
- **Model Compression & INT8 Roadmap**: Converted the PyTorch checkpoint (`.pth`) into static `food_model.onnx` (15.5 MB) format. Future work includes INT8 quantization to compress the model to **~1.8 MB**, reducing latency by an additional 40%.

#### Phase 2: Cloud Infrastructure Deployment & CI/CD (AWS CloudFormation / SAM)
- Packaged Lambda source code with required dependencies (`onnxruntime`, `Pillow`, `boto3`).
- Developed IaC template `infrastructure/template.yaml` (AWS SAM / CloudFormation) integrating automated CI/CD pipelines to test and deploy with a single CLI command.
- **Periodic Retraining Strategy**: Out-of-distribution images collected in `s3://.../ood_logs/` will be evaluated monthly. Upon reaching 1,000 new images, the system automatically triggers retraining pipelines to expand coverage to 100+ food classes.

### 5. Security & Privacy
- **Authentication & Authorization**: REST API access restricted via API Keys / Cognito User Pools. IAM Least-Privilege Roles applied to Lambda (granting read access only to the specified S3 bucket and write access to the DynamoDB table).
- **Data Encryption**: Server-Side Encryption (SSE-S3) enabled on Amazon S3 and DynamoDB data encryption at-rest.
- **Data Retention Management**: Configured S3 Lifecycle Rules to automatically delete `ood_logs/` images after 90 days to protect user privacy and optimize storage costs.

### 6. Timeline & Milestones
- **Weeks 1–2**: Food-101 dataset analysis, label cleaning, 50-class selection, Draw.io architecture design, and cost estimation.
- **Weeks 3–4**: AI model fine-tuning on Colab GPU (achieving Test Top-1 Acc 85.62% and F1-Score 0.86), ONNX export, and local edge-case testing.
- **Weeks 5–6**: CloudFormation/SAM template writing, resource deployment to region `ap-southeast-1` (Singapore), and Web UI integration with API Gateway.
- **Weeks 7–8**: CloudWatch performance evaluation (Cold Start vs Warm Start analysis), step-by-step Workshop documentation writing, and resource cleanup verification (`cleanup.sh`).

### 7. Budget Estimation & Cost Management
Cost calculations are derived from the official AWS Pricing Calculator for an operational scale of **100,000 requests/month** (~3,300 requests/day - realistic production scale):

| AWS Service | Monthly Workload | List Price Cost |
|---|---|---|
| **Amazon API Gateway** | 100,000 REST API calls ($3.50 / 1M) | $0.35 USD |
| **AWS Lambda** | 100,000 invocations (512MB RAM, 50ms/req) | $0.04 USD |
| **Amazon S3** | 5.0 GB storage + 10,000 PUT/GET requests | $0.15 USD |
| **Amazon DynamoDB** | 100,000 Write Units (WCU) + 2 GB data storage | $1.50 USD |
| **Amazon Rekognition** | ~10,000 Fallback AI calls (10% of requests) | $10.00 USD |
| **Amazon CloudWatch** | 5 GB Log Storage + 2 CloudWatch Alarms | $2.50 USD |
| **TOTAL MONTHLY COST** | **100,000 requests/month scale** | **~ $14.54 USD / month** |

> [!TIP] Superior Cost Optimization & Rekognition Risk Management:
> - **Serverless Advantage**: Compared to maintaining a 24/7 GPU EC2 server ($150–$300/month), Serverless saves >90% in operational costs.
> - **Rekognition Cost Mitigation (~69% of budget)**: To prevent Rekognition costs from rising if fallback traffic exceeds 10%, the system implements DynamoDB image hash caching for fallback results and dynamic Confidence Threshold tuning post-go-live.

### 8. Risk Assessment & Mitigation
- **Out-of-Distribution (OOD) Dish Risk**: Unseen food dishes outside the 50 classes ➔ *Mitigation*: If confidence < 60%, automatically trigger Amazon Rekognition label detection, fuzzy-match labels with `calorie_map.json` or assign a default nutritional group, while logging the image to `s3://.../ood_logs/`.
- **Lambda Cold Start Risk**: Initial call after container idle suffers ~350–500ms latency ➔ *Mitigation*: Maintaining an ultra-lightweight ONNX model (15.5 MB) keeps cold start minimal. For production environments requiring consistent SLA < 150ms, AWS Lambda Provisioned Concurrency can be enabled.
- **Image Quality Risk**: Uploaded photos are blurry, dark, or corrupted ➔ *Mitigation*: Image quality check function on Lambda intercepts low-quality images immediately, returning HTTP 400/422 with guidance for re-shooting.
- **Cost Overage Risk**: Traffic spam or denial of service ➔ *Mitigation*: Configure API Gateway throttling (limit 20 req/s) and set up CloudWatch Budget Alarms sending email alerts if monthly costs exceed threshold limits.

### 9. Expected Outcomes
1. **Technical Excellence**: Successfully built an end-to-end Serverless AI Computer Vision system with ultra-low warm Lambda compute latency (35–50ms), total web response latency < 150ms, auto-scaling capabilities, and 100% edge-case handling. Accurately classifies the 50 most popular food categories with a **Test Top-1 Accuracy of 85.62%** (Weighted Average F1-score of 0.86).
2. **Practical Value**: Delivers an automated nutrition tracking solution for users while providing a standardized AWS lab template for the FCAJ community on deploying Serverless AI/ML workloads.