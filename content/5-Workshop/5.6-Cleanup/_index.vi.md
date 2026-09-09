---
title: "Dọn dẹp tài nguyên"
date: 2026-08-04
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Quy trình Dọn dẹp Tài nguyên AWS (Resource Cleanup)

Để tránh phát sinh chi phí ngoài ý muốn sau khi hoàn thành bài thực hành, bạn hãy thực hiện xóa các tài nguyên đã khởi tạo trên AWS theo thứ tự các bước dưới đây:

---

#### 1. Xóa Amazon S3 Bucket:
1. Mở [Amazon S3 Console](https://s3.console.aws.amazon.com/s3/home?region=ap-southeast-1).
2. Chọn Bucket `fcaj-food-ai-storage-...`
3. Nhấp nút **Empty** ➔ Nhập `permanently delete` để xóa toàn bộ dữ liệu (bao gồm mô hình ONNX, bảng calo và thư mục ood_logs/).
4. Nhấp nút **Delete** ➔ Nhập tên Bucket để xác nhận xóa hoàn toàn.

![Xóa dữ liệu trong S3 Bucket](/FCAJ-Project/images/5-Workshop/5.6-Cleanup/cleanup_s3_empty.png)

---

#### 2. Xóa Amazon ECR Repository:
1. Mở [Amazon ECR Console](https://ap-southeast-1.console.aws.amazon.com/ecr/repositories?region=ap-southeast-1).
2. Chọn Repository `nutrivision-lambda`.
3. Nhấp nút **Delete** ➔ Nhập `delete` để xác nhận xóa toàn bộ Docker Image đã lưu trữ.

![Xóa ECR Repository](/FCAJ-Project/images/5-Workshop/5.6-Cleanup/cleanup_ecr.png)

---

#### 3. Xóa AWS Lambda Function:
1. Mở [AWS Lambda Console](https://ap-southeast-1.console.aws.amazon.com/lambda/home?region=ap-southeast-1#/functions).
2. Chọn hàm `NutriVisionPredictor`.
3. Nhấp **Actions ➔ Delete** ➔ Nhập `confirm` để xác nhận xóa hàm Lambda.

![Xóa hàm Lambda](/FCAJ-Project/images/5-Workshop/5.6-Cleanup/cleanup_lambda.png)

---

#### 4. Xóa Amazon API Gateway:
1. Mở [Amazon API Gateway Console](https://ap-southeast-1.console.aws.amazon.com/apigateway/main/apis?region=ap-southeast-1).
2. Chọn API `NutriVisionRestApi`.
3. Nhấp **Manage API ➔ Delete** ➔ Nhập `confirm` để xác nhận xóa REST API.

![Xóa API Gateway](/FCAJ-Project/images/5-Workshop/5.6-Cleanup/cleanup_apigateway.png)

---

#### 5. Xóa AWS Amplify App (Frontend Hosting):
1. Mở [AWS Amplify Console](https://ap-southeast-1.console.aws.amazon.com/amplify/home?region=ap-southeast-1).
2. Chọn ứng dụng `AI-NutriVision`.
3. Nhấp **App actions ➔ Delete app** ➔ Nhập `delete` để xác nhận gỡ bỏ ứng dụng Web.

---

#### 6. Xóa Amazon SNS Topic & Subscription:
1. Mở [Amazon SNS Console](https://ap-southeast-1.console.aws.amazon.com/sns/v3/home?region=ap-southeast-1#/topics).
2. Chọn Topic `NutriVision-AlarmNotifications` ➔ Nhấp **Delete** ➔ Nhập `delete me` để xác nhận xóa Topic.
3. Chuyển sang mục **Subscriptions** ➔ Chọn Subscription tương ứng và nhấp **Delete**.

![Xóa SNS Topic](/FCAJ-Project/images/5-Workshop/5.6-Cleanup/cleanup_sns.png)

---

#### 7. Xóa Amazon CloudWatch Alarm & Log Group:
1. Mở [Amazon CloudWatch Console](https://ap-southeast-1.console.aws.amazon.com/cloudwatch/home?region=ap-southeast-1).
2. Chọn mục **Alarms ➔ All alarms** ➔ Chọn Alarm `NutriVision-HighLambdaErrors` ➔ Nhấp **Actions ➔ Delete**.
3. Chuyển sang mục **Logs ➔ Log groups** ➔ Chọn Log Group `/aws/lambda/NutriVisionPredictor` ➔ Nhấp **Actions ➔ Delete log group(s)**.