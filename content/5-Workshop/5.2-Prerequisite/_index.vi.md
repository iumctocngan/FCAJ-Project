---
title: "Yêu cầu chuẩn bị"
date: 2026-08-04
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### Các yêu cầu tiên quyết cho bài thực hành

Để hoàn thành bài thực hành NutriVision, bạn cần chuẩn bị môi trường phát triển cục bộ và cấu hình xác thực AWS CLI theo các yêu cầu bên dưới:

#### 1. Yêu cầu Môi trường & Công cụ Cục bộ:
- **Tài khoản AWS**: Tài khoản có đủ quyền hạn triển khai các dịch vụ AWS cần thiết (S3, ECR, Lambda, API Gateway, CloudWatch, SNS, Amplify, Rekognition).
- **AWS CLI v2**: Đã cài đặt phiên bản AWS CLI v2 trên máy tính cá nhân.
- **Docker Desktop**: Đã cài đặt và khởi chạy Docker Daemon để phục vụ đóng gói Container Image cho Lambda.

---

#### 2. Cấu hình & Xác thực AWS CLI

Mở Terminal/PowerShell tại máy tính cá nhân và thực thi lệnh nhập thông tin xác thực AWS:
```bash
aws configure
```
- **AWS Access Key ID**: Nhập Access Key của bạn.
- **AWS Secret Access Key**: Nhập Secret Key tương ứng.
- **Default region name**: `ap-southeast-1`
- **Default output format**: `json`

Xác thực thông tin tài khoản kết nối thành công:
```bash
aws sts get-caller-identity
```

![Xác thực AWS CLI](/FCAJ-Project/images/5-Workshop/5.2-Prerequisite/aws_cli_conf.png)