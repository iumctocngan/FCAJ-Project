---
title: "Resource Cleanup"
date: 2026-08-04
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Resource Cleanup Workflow

To prevent unexpected ongoing cloud charges after completing the workshop, follow these teardown steps to delete all created AWS resources:

---

#### 1. Delete Amazon S3 Bucket:
1. Open [Amazon S3 Console](https://s3.console.aws.amazon.com/s3/home?region=ap-southeast-1).
2. Select Bucket `fcaj-food-ai-storage-...`
3. Click **Empty** ➔ Enter `permanently delete` to remove all stored data (including ONNX weights, calorie mapping, and ood_logs/).
4. Click **Delete** ➔ Enter bucket name to confirm complete bucket deletion.

![Empty S3 Bucket](/FCAJ-Project/images/5-Workshop/5.6-Cleanup/cleanup_s3_empty.png)

---

#### 2. Delete Amazon ECR Repository:
1. Open [Amazon ECR Console](https://ap-southeast-1.console.aws.amazon.com/ecr/repositories?region=ap-southeast-1).
2. Select Repository `nutrivision-lambda`.
3. Click **Delete** ➔ Enter `delete` to remove all container images.

![Delete ECR Repository](/FCAJ-Project/images/5-Workshop/5.6-Cleanup/cleanup_ecr.png)

---

#### 3. Delete AWS Lambda Function:
1. Open [AWS Lambda Console](https://ap-southeast-1.console.aws.amazon.com/lambda/home?region=ap-southeast-1#/functions).
2. Select function `NutriVisionPredictor`.
3. Click **Actions ➔ Delete** ➔ Enter `confirm` to confirm Lambda function deletion.

![Delete Lambda Function](/FCAJ-Project/images/5-Workshop/5.6-Cleanup/cleanup_lambda.png)

---

#### 4. Delete Amazon API Gateway:
1. Open [Amazon API Gateway Console](https://ap-southeast-1.console.aws.amazon.com/apigateway/main/apis?region=ap-southeast-1).
2. Select API `NutriVisionRestApi`.
3. Click **Manage API ➔ Delete** ➔ Enter `confirm` to confirm REST API deletion.

![Delete API Gateway](/FCAJ-Project/images/5-Workshop/5.6-Cleanup/cleanup_apigateway.png)

---

#### 5. Delete AWS Amplify App (Frontend Hosting):
1. Open [AWS Amplify Console](https://ap-southeast-1.console.aws.amazon.com/amplify/home?region=ap-southeast-1).
2. Select application `AI-NutriVision`.
3. Click **App actions ➔ Delete app** ➔ Enter `delete` to confirm web app removal.

---

#### 6. Delete Amazon SNS Topic & Subscription:
1. Open [Amazon SNS Console](https://ap-southeast-1.console.aws.amazon.com/sns/v3/home?region=ap-southeast-1#/topics).
2. Select Topic `NutriVision-AlarmNotifications` ➔ Click **Delete** ➔ Enter `delete me` to confirm topic deletion.
3. Navigate to **Subscriptions** ➔ Select the corresponding subscription and click **Delete**.

![Delete SNS Topic](/FCAJ-Project/images/5-Workshop/5.6-Cleanup/cleanup_sns.png)

---

#### 7. Delete Amazon CloudWatch Alarm & Log Group:
1. Open [Amazon CloudWatch Console](https://ap-southeast-1.console.aws.amazon.com/cloudwatch/home?region=ap-southeast-1).
2. Go to **Alarms ➔ All alarms** ➔ Select Alarm `NutriVision-HighLambdaErrors` ➔ Click **Actions ➔ Delete**.
3. Go to **Logs ➔ Log groups** ➔ Select Log Group `/aws/lambda/NutriVisionPredictor` ➔ Click **Actions ➔ Delete log group(s)**.