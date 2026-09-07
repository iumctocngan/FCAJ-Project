---
title: "Workshop"
date: 2026-08-04
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# NutriVision: Hệ thống nhận diện món ăn và phân tích dinh dưỡng tự động trên hạ tầng AWS Serverless

#### Tổng quan

Bài thực hành này hướng dẫn quy trình toàn diện (End-to-End) đưa một mô hình Thị giác máy tính (Computer Vision / Machine Learning) từ môi trường nghiên cứu (Google Colab) lên hạ tầng điện toán đám mây AWS Serverless Production.

Hệ thống NutriVision giúp tự động hóa việc tính toán hàm lượng calo và các chỉ số dinh dưỡng (Protein, Carbs, Fat, Fiber) từ hình ảnh bữa ăn với thời gian phản hồi nhanh chóng, tối ưu chi phí nhờ kiến trúc Serverless, kết hợp cơ chế dự phòng Amazon Rekognition và hệ thống giám sát cảnh báo thời gian thực Amazon CloudWatch & SNS.

* **Mã nguồn toàn bộ dự án**: [iumctocngan/NutriVision](https://github.com/iumctocngan/NutriVision)

---

#### Nội dung bài thực hành

1. [Tổng quan về workshop](5.1-Workshop-overview/)
2. [Chuẩn bị](5.2-Prerequisite/)
3. [Huấn luyện & Nén mô hình AI](5.3-Model-Training/)
4. [Triển khai Serverless trên AWS](5.4-Serverless-Deployment/)
5. [Giám sát CloudWatch & Cảnh báo SNS](5.5-Monitoring-Alerts/)
6. [Dọn dẹp tài nguyên](5.6-Cleanup/)
