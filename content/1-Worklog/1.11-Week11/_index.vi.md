---
title: "Worklog Tuần 11"
date: 2024-01-01
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---


### Mục tiêu tuần 11:

- Xây dựng backend suy luận và triển khai mô hình lên kiến trúc AWS Serverless.
- Phát triển giao diện web, kết nối các thành phần thành một hệ thống hoạt động hoàn chỉnh.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành |
| --- | --- | --- | --- |
| 2 | - Xây dựng Lambda inference bằng Python, xử lý ảnh, kiểm tra chất lượng và tính dinh dưỡng theo khẩu phần | 21/09/2026 | 21/09/2026 |
| 3 | - Đóng gói backend và mô hình ONNX thành Docker Image, đẩy lên Amazon ECR và triển khai Lambda | 22/09/2026 | 22/09/2026 |
| 4 | - Tạo S3 lưu artefact và ảnh độ tin cậy thấp; tích hợp Amazon Rekognition làm cơ chế AI fallback | 23/09/2026 | 23/09/2026 |
| 5 | - Cấu hình API Gateway POST /predict, CORS, Rate Limiting và kiểm thử kết nối với Lambda | 24/09/2026 | 24/09/2026 |
| 6 | - Hoàn thiện Web UI, kết nối API và triển khai frontend lên AWS Amplify Hosting | 25/09/2026 | 25/09/2026 |

### Kết quả đạt được tuần 11:

- Triển khai thành công backend suy luận ONNX trên AWS Lambda bằng Docker Container Image từ Amazon ECR.
- Hoàn thiện API POST /predict với kiểm tra đầu vào, CORS, Rate Limiting và phản hồi dinh dưỡng theo khẩu phần.
- Tích hợp Amazon S3 để lưu artefact, ảnh OOD và Amazon Rekognition làm cơ chế dự phòng khi độ tin cậy dưới 60%.
- Đưa Web UI lên AWS Amplify và kết nối thành công luồng end-to-end từ tải ảnh đến hiển thị kết quả.
