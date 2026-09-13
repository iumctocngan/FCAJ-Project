---
title: "Worklog Tuần 9"
date: 2024-01-01
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---


### Mục tiêu tuần 9:
- Xây dựng backend suy luận Serverless trên AWS Lambda bằng Docker Container Image.
- Tích hợp lưu trữ S3, cơ chế AI Fallback qua Rekognition, cấu hình API Gateway và triển khai Web UI lên AWS Amplify.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành |
| --- | --- | --- | --- |
| 2 | - Xây dựng Lambda inference bằng Python (ONNX Runtime), xử lý ảnh, kiểm tra chất lượng và tính dinh dưỡng theo khẩu phần | 07/09/2026 | 07/09/2026 |
| 3 | - Đóng gói backend và mô hình ONNX thành Docker Container Image, đẩy lên Amazon ECR và cấu hình Lambda function | 08/09/2026 | 08/09/2026 |
| 4 | - Thiết lập S3 Bucket lưu trữ hình ảnh; tích hợp Amazon Rekognition làm cơ chế AI fallback khi độ tin cậy < 60% | 09/09/2026 | 09/09/2026 |
| 5 | - Cấu hình REST API trên Amazon API Gateway (POST /predict, CORS, Throttle/Rate Limiting) kết nối Lambda | 10/09/2026 | 10/09/2026 |
| 6 | - Phát triển giao diện Web UI responsive, kết nối API và triển khai frontend lên AWS Amplify Hosting | 11/09/2026 | 11/09/2026 |

### Kết quả đạt được tuần 9:
- Triển khai thành công backend suy luận ONNX trên AWS Lambda bằng Docker Container Image từ Amazon ECR.
- Hoàn thiện API POST /predict với xác thực dữ liệu đầu vào, CORS và tính toán dinh dưỡng theo khẩu phần linh hoạt.
- Tích hợp Amazon S3 và Amazon Rekognition làm cơ chế dự phòng thông minh cho các ảnh độ tin cậy thấp.
- Hoàn thành giao diện Web UI hiện đại, đưa lên AWS Amplify Hosting và kết nối thành công luồng end-to-end từ người dùng đến backend.
