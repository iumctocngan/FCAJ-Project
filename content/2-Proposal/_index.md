---
title: "Proposal"
date: 2026-08-03
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# NutriVision
## Automated Food Recognition and Nutrition Analysis System on AWS Serverless Infrastructure

### 1. Executive Summary
The **NutriVision** project streamlines dietary management by automating food identification and macronutrient calculation (Protein, Carbs, Fat, Fiber) directly from meal photographs using Machine Learning & Computer Vision.

The underlying model is trained on the Food-101 dataset (comprising 101,000 images across 101 food classes). To optimize for practical nutrition tracking, the dataset underwent curated data cleaning to select 50 highest-frequency food categories (50,000 images). An EfficientNet-B0 deep neural network was fine-tuned on this dataset, achieving a Test Top-1 Accuracy of 85.62% (Weighted F1-Score 0.86) across 5,000 independent test images under experimental conditions.

The model is exported to an optimized static ONNX format (15.5 MB) and operates on AWS Serverless architecture (Amplify, API Gateway, Lambda, ECR, S3, Rekognition, CloudWatch, SNS). This solution reduces meal logging time to mere seconds per meal while maintaining minimal operational expenditure via pay-per-use billing.

### 2. Problem Statement
#### The Challenge
Calculating daily calories and macronutrient breakdown currently depends heavily on manual-entry apps (such as MyFitnessPal, Yazio). Users must manually search for food names, estimate portion weights, and log each item. This multi-step process introduces high friction, leading to user fatigue and abandonment over time.

#### Proposed Solution
NutriVision delivers a modern web application hosted on AWS Amplify Hosting, supporting secure HTTPS access from any device without requiring complex user registration. Users simply capture or upload a meal photo. The image is processed through API Gateway to AWS Lambda for rapid food classification and nutrition estimation.
- **AI Fallback Mechanism (Amazon Rekognition)**: When the primary ONNX model confidence drops below 60% (due to poor lighting, challenging angles, or partial occlusion), the system automatically triggers Amazon Rekognition to scan general visual labels and maps them against the 50 learned food items in the database. Note: This mechanism recovers confidence for dishes within the 50 defined categories and does not classify out-of-scope foods. All low-confidence images are automatically saved to s3://.../ood_logs/ for offline inspection and retraining.
- **Flexible Portion Scaling**: The Web UI enables users to select standard portion multipliers (Small 0.7x, Medium 1.0x, Large 1.5x, Extra 2.0x), prompting Lambda to proportionally scale calories and macronutrients accordingly.

#### Benefits & Return on Investment (ROI)
- **Time Savings**: Substantially cuts daily dietary logging effort for fitness enthusiasts, dieters, and patients needing nutritional monitoring via a single photo upload.
- **Cost Efficiency**: Serverless compute incurs zero idle server costs, executing on-demand and minimizing operational overhead.

### 3. Solution Architecture
![NutriVision Architecture Diagram](/FCAJ-Project/images/2-Proposal/solution_architecture.drawio.png)

#### AWS Services Utilized (8 Core Services):
1. **AWS Amplify**: Hosts static frontend assets with automated GitHub CI/CD, global CloudFront CDN distribution, and managed HTTPS certificates.
2. **Amazon API Gateway**: Manages the REST API endpoint POST /predict with CORS handling and rate limiting (20 req/s) for abuse prevention.
3. **AWS Lambda**: Executes serverless inference (Python 3.12 / ONNX Runtime / Boto3), handles image decoding, quality validation, and portion scaling (0.7x – 2.0x).
4. **Amazon ECR**: Hosts and manages the Lambda Docker container image bundling the AI model and native C++ runtime dependencies.
5. **Amazon S3**: Centralized object storage for model weights food_model.onnx (15.5 MB), calorie_map.json database, and Out-of-Distribution audit images (ood_logs/).
6. **Amazon Rekognition**: Fallback AI engine. Triggers label detection when primary model confidence drops below 60%.
7. **Amazon CloudWatch**: Monitors P95 Latency, manages Log Groups, and configures alarms for system anomaly detection.
8. **Amazon SNS**: Dispatches immediate email notifications to engineers whenever CloudWatch alarms trigger.

### 4. Technical Implementation & Real-World MLOps
The project is organized into two primary phases:

#### Phase 1: Data Preparation, Training & ONNX Export (Google Colab GPU)
- **Data Curation**: From the original Food-101 dataset (101 classes), the project selected 50 top consumed food categories (50,000 images) according to three technical criteria:
  1. *Consumption Frequency & Popularity*: Balances Asian staples (Pho, Fried Rice, Pad Thai, Sushi, Bibimbap, Gyoza) and Western staples (Hamburger, Pizza, Steak, Spaghetti, Club Sandwich, Mac & Cheese).
  2. *Dietary Diversity*: Spans 6 balanced nutritional categories: High-protein dishes (Steak, Ribs, Wings), Grains & Noodles (Pho, Fried Rice), Fast food (Burger, Pizza), Salads (Caesar, Greek), Breakfast (Omelette, Pancakes), and Desserts (Cheesecake, Apple Pie).
  3. *Visual Separability*: Excludes visually ambiguous classes to maintain high classification precision and compact model footprint.
  - The curated 50-dish dataset is partitioned into: 37,500 training images (750/class), 7,500 validation images (150/class), and 5,000 independent test images (100/class).
- **Training & ONNX Serialization**: EfficientNet-B0 backbone fine-tuned via a two-stage protocol (Freeze & Unfreeze). Achieved Test Top-1 Accuracy of 85.62% on 5,000 test images in experiment benchmarks. The PyTorch checkpoint is exported to a static food_model.onnx (15.5 MB) artifact for minimal storage and rapid initialization.

#### Phase 2: Cloud Deployment & Operational Observability
- **Frontend CI/CD Automation**: Connects the GitHub repository to AWS Amplify Hosting. Each commit pushed to the main branch automatically triggers build and zero-downtime deployment in seconds.
- **Infrastructure & Backend Deployment**: Packages Lambda Function as a Docker Image, pushes to Amazon ECR, and provisions AWS resources (Lambda, API Gateway, S3, Rekognition, CloudWatch, SNS) via AWS Console GUI and AWS CLI.
- **Monitoring & Edge Case Handling**:
  - Centralizes execution telemetry in Amazon CloudWatch Log Groups.
  - Automatically captures low-confidence samples (< 60%) to s3://.../ood_logs/.
  - *Current Workflow*: OOD data auditing and retraining is handled via a human-in-the-loop workflow on Google Colab GPU.

> [!NOTE] Future Roadmap: Fully automate retraining using Amazon SageMaker Pipelines (data labeling via SageMaker Ground Truth, automated retraining triggers, model registry sync, and zero-downtime Lambda updates).

### 5. Project Roadmap & Milestones
Executed over a 2-month timeframe across 4 key stages:
- **Month 1 (First Half - Weeks 1-2)**: Analyze Food-101 dataset, clean label noise, curate 50 food categories, draft Draw.io architecture diagrams, and evaluate AWS cost projections.
- **Month 1 (Second Half - Weeks 3-4)**: Fine-tune EfficientNet-B0 model on Colab GPU, export ONNX artifact, build Lambda inference code, and validate locally.
- **Month 2 (First Half - Weeks 5-6)**: Deploy all AWS Serverless infrastructure to ap-southeast-1 (Singapore) using Docker Container / AWS CLI, host Frontend on AWS Amplify Hosting, and connect Web UI with API Gateway & Lambda.
- **Month 2 (Second Half - Weeks 7-8)**: Evaluate CloudWatch performance metrics (Cold vs Warm Start), configure SNS Email Alarms, execute comprehensive testing, and finalize step-by-step Workshop documentation.

### 6. Budget Estimation & Cost Management

| Service | Estimated Cost |
|---|---|
| AWS Amplify Hosting | ~$0.50/month |
| Amazon API Gateway | ~$0.35/month |
| AWS Lambda | ~$0.00/month |
| Amazon ECR | ~$0.03/month |
| Amazon S3 (Storage & Requests) | ~$0.15/month |
| Amazon Rekognition (Fallback AI) | ~$5.00/month |
| Amazon CloudWatch | ~$0.02/month |
| Amazon SNS | ~$0.00/month |
| **Total Estimate** | **~$6.05 USD/month** |

> [!NOTE] 
> Standard pricing (Pay-as-you-go) is approximately ~$6.05 USD/month for 100,000 recognitions. Under AWS Free Tier (first year), core infrastructure services are free, reducing actual cost to $0.00 – $5.00 USD/month (only incurring cost if Rekognition exceeds 5,000 free calls).

### 7. Risk Management & Mitigation Strategies
- **Out-of-Distribution (OOD) Food Risk**: Unlearned food items outside the 50 classes ➔ *Mitigation*: If confidence < 60%, automatically trigger Amazon Rekognition and attempt to map the result to the nearest dish among the 50 defined items. If no valid mapping is found, the system deliberately returns HTTP 422 rather than producing inaccurate nutritional data, preserving data integrity. All OOD images are stored to s3://.../ood_logs/ for offline retraining.
- **Lambda Cold Start Risk**: First-time invocation after container idle may incur latency ➔ *Mitigation*: Maintain lightweight Python execution logic to minimize cold start duration. For production environments requiring consistent SLA, consider AWS Lambda Provisioned Concurrency.
- **Input Image Quality Risk**: Photos that are blurry, dark, or corrupted ➔ *Mitigation*: Image format and quality validation intercept invalid payloads immediately, returning error responses with user re-shooting guidance.
- **Cost Overrun & Incident Risk**: Request spamming or Lambda runtime errors ➔ *Mitigation*: Configure API Gateway Throttling (20 req/s limit) and CloudWatch Budget Alarms to trigger Amazon SNS real-time email notifications to administrators.

### 8. Expected Outcomes
1. **Technical Delivery**: Successfully deploy an end-to-end Serverless Computer Vision system hosted on AWS Amplify with automatic scaling and edge-case handling. The 50-dish classification model achieves a Test Top-1 Accuracy of 85.62% (Weighted F1-Score 0.86).
2. **Economic & Operational Value**: Eliminates idle server costs compared to legacy EC2 instances, providing rapid response times at only a few dollars per month for 100,000 requests.
3. **Educational Material**: Complete step-by-step Workshop documentation with real AWS Console screenshots for easy reproducibility.