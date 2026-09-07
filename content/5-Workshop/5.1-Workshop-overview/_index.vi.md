---
title: "Giới thiệu tổng quan"
date: 2026-08-04
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

### 1. Mục tiêu
Thực hiện quy trình toàn diện (End-to-End) đưa mô hình Thị giác máy tính (Computer Vision / Machine Learning) từ môi trường nghiên cứu (Google Colab) lên hạ tầng AWS Serverless Production. Người học sẽ thực hành huấn luyện, nén mô hình ONNX, hosting giao diện Web tĩnh trên AWS Amplify, xây dựng REST API Serverless, thiết lập AI dự phòng và tự động hóa giám sát/dọn dẹp tài nguyên.

---

### 2. Tổng quan giải pháp
NutriVision tự động nhận diện món ăn và tính toán calo cùng chỉ số dinh dưỡng (Protein, Carbs, Fat, Fiber) từ hình ảnh bữa ăn:
* **Giao diện Web 24/7**: Lưu trữ giao diện Frontend (HTML/CSS/JS) trên AWS Amplify Hosting với tên miền HTTPS bảo mật và tự động tích hợp CI/CD với GitHub.
* **Hiệu năng & Chi phí**: Mô hình ONNX (15.5 MB) chạy trên AWS Lambda xử lý nhanh chóng, tối ưu hóa chi phí vận hành nhờ kiến trúc Serverless tính phí theo mức độ sử dụng thực tế.
* **AI Fallback (Amazon Rekognition)**: Khi mô hình ONNX chính mất tự tin (confidence < 60%) do chất lượng ảnh kém, hệ thống kích hoạt Amazon Rekognition và thử ánh xạ sang một trong 50 món đã định nghĩa. Nếu không khớp được, hệ thống trả HTTP 422 thay vì đoán sai. Toàn bộ ảnh độ tin cậy thấp được lưu tự động về S3 ood_logs/ để tái huấn luyện.
* **Giám sát thời gian thực**: Tự động gửi Email cảnh báo qua Amazon SNS khi phát sinh lỗi hệ thống hoặc vượt ngưỡng ngân sách.

---

### 3. Kiến trúc & Luồng hệ thống

![Kiến trúc hệ thống NutriVision](/FCAJ-Project/images/2-Proposal/solution_architecture.drawio.png)

Hệ thống hoạt động theo 4 nhóm luồng tương ứng với các mũi tên trên sơ đồ kiến trúc:

#### A. Quy trình Đóng gói & Triển khai (Deployment Flow):
1. **Dev ➔ AWS ECR**: Nhà phát triển đóng gói mã nguồn xử lý và mô hình AI ONNX vào Docker Image, sau đó đẩy lên kho lưu trữ Amazon ECR.
2. **AWS ECR ➔ AWS Lambda**: AWS Lambda khởi tạo và kéo Container Image từ ECR để thực thi suy luận trực tiếp với hiệu năng cao và thời gian cold start tối ưu.
3. **Dev ➔ Amazon S3**: Nhà phát triển tải tệp mô hình gốc food_model.onnx (15.5 MB) và bảng dữ liệu dinh dưỡng calorie_map.json lên Amazon S3 để làm kho lưu trữ trung tâm (Master Store).

#### B. Luồng Người dùng & Suy luận Thời gian thực (Runtime Inference Flow):
4. **User ➔ AWS Amplify**: Người dùng truy cập ứng dụng thông qua tên miền HTTPS bảo mật được phân phối bởi AWS Amplify Hosting.
5. **AWS Amplify ➔ Amazon API Gateway**: Giao diện Web gửi hình ảnh bữa ăn và hệ số khẩu phần ăn (0.7x – 2.0x) qua request POST /predict.
6. **Amazon API Gateway ➔ AWS Lambda**: API Gateway tiếp nhận request, xử lý CORS, áp dụng Rate Limiting (20 req/s, Burst 40) chống spam và chuyển tiếp payload đến hàm Lambda.
7. **AWS Lambda (Inference Engine)**: Giải mã ảnh, kiểm tra chất lượng ảnh (Quality Check: độ sáng, độ mờ) ➔ nạp tensor và chạy suy luận AI với mô hình EfficientNet-B0 ONNX.

#### C. Cơ chế Phân nhánh & AI Dự phòng (Decision, Fallback & Logging Flow):
8. **Nhánh chính (Độ tin cậy ≥ 60%)**: Lambda tra cứu hàm lượng Calo, Protein, Carbs, Fat theo khẩu phần và trả kết quả JSON về Web UI.
9. **Nhánh dự phòng (Độ tin cậy < 60%)**:
   - **Lambda ➔ Amazon Rekognition**: Tự động kích hoạt Rekognition quét nhãn tổng quan (nếu khớp từ khóa ➔ trả về món tương ứng; nếu không khớp ➔ chủ động trả về mã lỗi HTTP 422).
   - **Lambda ➔ Amazon S3**: Tự động lưu trữ tệp ảnh món lạ vào thư mục s3://.../ood_logs/ kèm nhãn độ tin cậy để phục vụ đánh giá và tái huấn luyện mô hình.

#### D. Giám sát Vận hành & Cảnh báo Sự cố (Observability & Alerting Flow):
10. **AWS Lambda ➔ Amazon CloudWatch**: Ghi nhận toàn bộ nhật ký suy luận (Log Streams) và số liệu hiệu năng (Metrics: Latency, Invocations, Errors).
11. **Amazon CloudWatch ➔ Amazon SNS ➔ Email**: Khi CloudWatch Alarm phát hiện số lượng lỗi vượt ngưỡng quy định (Errors >= 1), hệ thống tự động kích hoạt Amazon SNS gửi Email cảnh báo sự cố thời gian thực đến kỹ sư vận hành.

---

### 4. Dịch vụ AWS sử dụng (8 Dịch vụ Core)

| Dịch vụ AWS | Vai trò |
|---|---|
| AWS Amplify | Hosting giao diện Web tĩnh (Frontend), tích hợp CI/CD với GitHub, phân phối qua CloudFront CDN & SSL HTTPS. |
| Amazon API Gateway | Quản lý REST API, CORS, xác thực & Rate Limiting (20 req/s). |
| AWS Lambda | Tính toán Serverless (Python 3.12 + ONNX Runtime), suy luận & tính calo theo khẩu phần. |
| Amazon ECR | Lưu trữ và quản lý Docker Container Image cho AWS Lambda. |
| Amazon S3 | Lưu trữ mô hình ONNX (15.5 MB), calorie_map.json và log ảnh món lạ ood_logs/. |
| Amazon Rekognition | Engine AI dự phòng khi mô hình chính có confidence < 60%. |
| Amazon CloudWatch | Giám sát P95 Latency, Log Groups và cấu hình Alarms. |
| Amazon SNS | Gửi Email cảnh báo sự cố thời gian thực cho kỹ sư. |

---

### 5. Kết quả kỳ vọng
1. **Triển khai Serverless LIVE**: Hoàn thành hệ thống AI trên AWS với Web Frontend hosted trên AWS Amplify và API Gateway hoạt động ổn định, phản hồi nhanh chóng.
2. **Làm chủ MLOps**: Thành thạo nén PyTorch sang ONNX (15.5 MB) và đóng gói môi trường suy luận nhẹ.
3. **Hoàn thành các kịch bản kiểm thử**: Nhận diện 50 món ăn, Kích hoạt Fallback Rekognition, và Bắt lỗi ảnh mờ HTTP 422.
4. **Giám sát & Dọn dẹp**: Thiết lập cảnh báo Email từ SNS và dọn dẹp sạch tài nguyên sau khi hoàn thành.
