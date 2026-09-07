---
title: "Create S3 Bucket & Upload ONNX Model"
date: 2026-08-04
weight: 1
chapter: false
pre: " <b> 5.4.1 </b> "
---

#### Step-by-Step S3 Bucket Creation & Artifact Upload via AWS Console

Follow these steps to create an Amazon S3 cloud storage bucket on the AWS Management Console to store the static ONNX model artifact (15.5 MB) and nutritional database:

1. **Navigate to Amazon S3 Console:**
   - Log in to the [AWS Management Console](https://console.aws.amazon.com/s3/).
   - On the Amazon S3 overview dashboard, click **Create bucket**.

![Amazon S3 - Create bucket](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step1_create.png)

2. **Configure Bucket Name & Region:**
   - **AWS Region**: Select **Asia Pacific (Singapore) ap-southeast-1**.
   - **Bucket type**: Select **General purpose**.
   - **Bucket namespace**: Select **Account Regional namespace (recommended)**.
   - **Bucket name prefix**: Enter `fcaj-food-ai-storage-2026-sg` (AWS automatically appends your Account ID and Region for global uniqueness).

![Configure Bucket Name & Region](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step2_config.png)

3. **Configure Security & Public Access Block:**
   - **Object Ownership**: Keep default **ACLs disabled (recommended)**.
   - **Block Public Access settings for this bucket**: Check **Block all public access** to protect model weights.

![Configure Security Permissions](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step3_security.png)

   - Scroll down and click **Create bucket**.

4. **Verify Bucket Creation:**
   - A green confirmation banner confirms bucket creation.
   - Click on the newly created bucket name to open its object overview page.

![Bucket Creation Confirmation](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step4_success.png)

5. **Upload ONNX Model Artifact (food_model.onnx):**
   - In the bucket root directory, click **Upload**. Select the food_model.onnx file (15.5 MB) from your local computer.
   - Click **Upload** at the bottom. The screen confirms **Succeeded (100.00%)**.

![Upload ONNX Model Success](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step5_upload.png)

6. **Create Data Subfolder:**
   - Return to the bucket overview, click **Create folder**.
   - **Folder name**: Enter `data`.
   - Click **Create folder** to initialize the data directory.

![Create Data Folder](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step6_create_folder_data.png)

7. **Upload calorie_map.json to Data Folder:**
   - Open the newly created **data** folder.
   - Click **Upload** and select calorie_map.json (8.5 KB) from your local machine.
   - Click **Upload**. The screen confirms **Succeeded (100.00%)** at `s3://.../data/calorie_map.json`.

![Upload calorie_map.json to Data Folder](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step7_upload_calorie_map.png)
