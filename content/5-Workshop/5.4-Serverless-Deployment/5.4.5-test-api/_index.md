---
title: "Test API & Edge Cases"
date: 2026-08-04
weight: 5
chapter: false
pre: " <b> 5.4.5 </b> "
---

#### End-to-End API Testing & Edge Cases Handling Workflow

In this step, access the live web application on AWS Amplify, connect your API Gateway Invoke URL, and test the entire system across real-world scenarios.

#### 1. Configure Endpoint URL in Web UI:
- Open your web application hosted on AWS Amplify (URL generated from step 5.4.4).
- Paste your API Gateway Invoke URL (format: https://<id>.execute-api.ap-southeast-1.amazonaws.com/prod/predict) into the Endpoint Config input box at the top right of the Web UI.

#### 2. Scenario 1 — Standard Food Recognition (Happy Path):
- Drag and drop a food image belonging to the 50 learned categories (e.g., Apple Pie, Pho, Pizza) ➔ Select portion size Medium (1.0x) ➔ Click Phân Tích Dinh Dưỡng.
- Expected Result: Returns accurate food name, estimated calories, and macronutrient distribution breakdown (Protein, Carbs, Fat, Fiber).

![Test Web UI Success Result](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/web_ui_test_success.png)

#### 3. Scenario 2 — AI Fallback Testing with Amazon Rekognition (Challenging Visual Angle):
- Upload a photo of a dish belonging to the 50 categories but captured under a challenging angle or while pouring broth causing primary ONNX confidence to drop (55.3% < 60%).
- Expected Result: ONNX confidence < 60% ➔ Lambda automatically triggers Amazon Rekognition (detect_labels) to scan visual labels (Soup, Noodle, Broth...) ➔ Successfully maps to Phở Bò (380 KCAL) and logs the image to s3://.../ood_logs/.
- Verify Telemetry: Web UI displays Engine: Amazon Rekognition fallback and Confidence: 55.3%.

![Test Web UI Rekognition Fallback Result](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/web_ui_test_rekognition_fallback.png)

#### 4. Scenario 3 — Out-of-Distribution Food (HTTP 422):
- Upload a photo of a dish completely outside the 50 learned categories with no matching Rekognition labels (e.g., cheese corn dog or a rare local dish).
- Expected Result: Both ONNX and Rekognition fail to find a valid match ➔ System intentionally returns HTTP 422 rather than guessing incorrectly. The OOD image is automatically logged to s3://.../ood_logs/ for offline retraining review.

![Test Web UI OOD Result](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/web_ui_test_ood_422.png)

- Verify Image Storage in Amazon S3: Open S3 Console ➔ Bucket fcaj-food-ai-storage-... ➔ Folder ood_logs/YYYY/MM/DD/ to inspect the newly recorded image file tagged with ONNX confidence score (e.g., ..._0.1738.jpg).

![S3 OOD Logs Verification](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_ood_logs_verification.png)

#### 5. Scenario 4 — Image Quality Check (Corrupted / Dark Image — HTTP 422):
- Drag and drop a completely black, severely blurred, or overexposed image.
- Expected Result: Lambda image quality check function (check_image_quality) halts execution before AI inference, returning HTTP 422 Unprocessable Entity with notification: "Cảnh báo chất lượng ảnh (HTTP 422): Ảnh quá tối. Vui lòng chụp lại ở nơi đủ ánh sáng."

![Test Web UI Poor Quality 422 Result](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/web_ui_test_poor_quality_422.png)
