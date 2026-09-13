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
The NutriVision project streamlines dietary management by automating food identification and macronutrient calculation (Protein, Carbs, Fat, Fiber) directly from meal photographs using Computer Vision.

The underlying model is trained on the Food-101 dataset (comprising 101,000 images across 101 food classes). To optimize for practical nutrition tracking, the dataset underwent curated data cleaning to select 50 highest-frequency food categories (50,000 images). An EfficientNet-B0 neural network was fine-tuned on this dataset, achieving a Test Top-1 Accuracy of 85.62% (Weighted F1-Score 0.86) across 5,000 independent test images.

The model is exported to an optimized static ONNX format (15.5 MB) and operates on AWS Serverless architecture (Amplify, API Gateway, Lambda, ECR, S3, Rekognition, CloudWatch, SNS). This solution reduces meal logging time to mere seconds per meal while maintaining minimal operational expenditure via pay-per-use billing.

### 2. Problem Statement
#### The Challenge
Calculating daily calories and macronutrient breakdown currently depends heavily on manual-entry apps (such as MyFitnessPal, Yazio). Users must manually search for food names, estimate portion weights, and log each item. This multi-step process introduces high friction, leading to user fatigue and abandonment over time.

#### Proposed Solution
NutriVision delivers a web application hosted on AWS Amplify Hosting, supporting secure HTTPS access from any device without requiring complex user registration. Users simply capture or upload a meal photo. The image is processed through API Gateway to AWS Lambda for rapid food classification and nutrition estimation.
- AI Fallback Mechanism via Amazon Rekognition: When the primary ONNX model confidence drops below 60% (due to poor lighting, challenging angles, or partial occlusion), the system automatically triggers Amazon Rekognition to scan general visual labels and maps them against the 50 learned food items in the database. If unmapped, the system intentionally returns HTTP 422 rather than guessing incorrectly. All low-confidence images are automatically saved to S3 for offline inspection and retraining.
- Flexible Portion Scaling: The Web UI enables users to select standard portion multipliers (Small 0.7x, Medium 1.0x, Large 1.5x, Extra 2.0x), prompting Lambda to proportionally scale calories and macronutrients accordingly.

#### Practical Value and Benefits
- Time Savings: Substantially cuts daily dietary logging effort for fitness enthusiasts, dieters, and patients needing nutritional monitoring via a single photo upload.
- Cost Efficiency: Serverless compute incurs zero idle server costs, executing on-demand and minimizing operational overhead.

### 3. Solution Architecture
![NutriVision Architecture Diagram](/FCAJ-Project/images/2-Proposal/solution_architecture.drawio.png)

#### AWS Services Utilized:
- AWS Amplify: Hosts static frontend assets with automated GitHub CI/CD, global CloudFront CDN distribution, and managed HTTPS certificates.
- Amazon API Gateway: Manages the REST API endpoint POST /predict with CORS handling and rate limiting (20 req/s) for abuse prevention.
- AWS Lambda: Executes serverless inference (Python 3.12, ONNX Runtime, Boto3), handles image decoding, quality validation, and portion scaling.
- Amazon ECR: Hosts and manages the Lambda Docker container image bundling the AI model and native runtime dependencies.
- Amazon S3: Centralized object storage for model weights food_model.onnx (15.5 MB), calorie_map.json database, and low-confidence audit images.
- Amazon Rekognition: Fallback AI engine triggered when primary model confidence drops below 60%.
- Amazon CloudWatch: Monitors latency, manages Log Groups, and configures alarms for system anomaly detection.
- Amazon SNS: Dispatches immediate email notifications to engineers whenever CloudWatch alarms trigger.

### 4. Technical Implementation and MLOps Workflow
The project is organized into two primary phases:

#### Phase 1: Data Preparation, Training and ONNX Export (Google Colab GPU)
- Data Curation: From the original Food-101 dataset (101 classes), the project selected 50 top consumed food categories (50,000 images) according to three technical criteria:
  - Consumption frequency and popularity: Balances Asian staples (Pho, Fried Rice, Pad Thai, Sushi, Bibimbap, Gyoza) and Western staples (Hamburger, Pizza, Steak, Spaghetti, Club Sandwich, Mac & Cheese).
  - Dietary diversity: Spans high-protein dishes, grains and noodles, fast food, salads, breakfast items, and desserts.
  - Visual separability: Excludes visually ambiguous classes to maintain high classification precision and compact model footprint.
  - The curated 50-dish dataset is partitioned into: 37,500 training images (750/class), 7,500 validation images (150/class), and 5,000 independent test images (100/class).
- Training and ONNX serialization: EfficientNet-B0 backbone fine-tuned via a two-stage protocol (Freeze and Unfreeze). Achieved Test Top-1 Accuracy of 85.62% on 5,000 test images. The checkpoint is exported to a static food_model.onnx (15.5 MB) artifact for minimal storage and rapid initialization.

#### Phase 2: Cloud Deployment and Operational Observability
- Frontend CI/CD Automation: Connects the GitHub repository to AWS Amplify Hosting. Each commit pushed to the main branch automatically triggers build and zero-downtime deployment in seconds.
- Infrastructure and Backend Deployment: Packages Lambda Function as a Docker Image, pushes to Amazon ECR, and provisions AWS resources (Lambda, API Gateway, S3, Rekognition, CloudWatch, SNS) via AWS Console and AWS CLI.
- Monitoring and Edge Case Handling:
  - Centralizes execution telemetry in Amazon CloudWatch Log Groups.
  - Automatically captures low-confidence samples (< 60%) to S3 for offline review.

### 5. Project Roadmap & Milestones
Executed over the final 3 weeks of the internship (Weeks 8 – 10):
- Week 8: Analyze problem requirements, design system architecture, curate 50 food categories from Food-101, fine-tune EfficientNet-B0, and export model to ONNX.
- Week 9: Develop serverless inference backend on AWS Lambda (Docker container), integrate S3, configure Amazon Rekognition fallback, set up API Gateway, and deploy Web UI to AWS Amplify Hosting.
- Week 10: Conduct end-to-end system testing, optimize latency, configure CloudWatch monitoring and SNS email alerts, finalize step-by-step workshop documentation, and complete final handover.

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
| Total Estimate | ~$6.05 USD/month |

> [!NOTE] 
> Standard pricing is approximately ~$6.05 USD/month for 100,000 recognitions. Under AWS Free Tier (first year), core infrastructure services are free, reducing actual cost to $0.00 – $5.00 USD/month (only incurring cost if Rekognition exceeds 5,000 free calls).

### 7. Risk Management & Mitigation Strategies
- Out-of-Distribution Food Risk: If confidence < 60%, automatically trigger Amazon Rekognition and attempt to map the result to the nearest dish among the 50 defined items. If no valid mapping is found, the system deliberately returns HTTP 422 rather than producing inaccurate nutritional data, preserving data integrity. All low-confidence images are stored to S3 for offline retraining.
- Lambda Cold Start Risk: Optimize Python code and minimize dependencies to reduce cold start duration. For production environments requiring consistent SLA, consider AWS Lambda Provisioned Concurrency.
- Input Image Quality Risk: Image format and quality validation intercept invalid payloads immediately, returning error responses with user guidance.
- Cost Overrun and Incident Risk: Configure API Gateway Throttling (20 req/s limit) and CloudWatch Budget Alarms to trigger Amazon SNS real-time email notifications to administrators.

### 8. Expected Outcomes
- Technical Delivery: Successfully deploy an end-to-end Serverless Computer Vision system hosted on AWS Amplify with automatic scaling and edge-case handling. The 50-dish classification model achieves a Test Top-1 Accuracy of 85.62% (Weighted F1-Score 0.86).
- Economic & Operational Value: Incurs no idle server maintenance costs compared to traditional EC2 instances thanks to Serverless architecture, providing rapid response times at only a few dollars per month for 100,000 requests.
- Educational Material: Complete step-by-step Workshop documentation with real AWS Console screenshots for easy reproducibility.