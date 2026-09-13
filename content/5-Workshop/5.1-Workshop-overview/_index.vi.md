---
title: "Giới thiệu tổng quan"
date: 2026-08-04
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

### 1. Mục tiêu
Thực hiện quy trình đưa mô hình Thị giác máy tính từ môi trường nghiên cứu (Google Colab) lên hạ tầng AWS Serverless Production. Bài thực hành bao gồm các bước huấn luyện, nén mô hình ONNX, lưu trữ giao diện web trên AWS Amplify, xây dựng REST API Serverless, thiết lập AI dự phòng và cấu hình giám sát cũng như dọn dẹp tài nguyên.

---

### 2. Tổng quan giải pháp
NutriVision tự động nhận diện món ăn và tính toán calo cùng chỉ số dinh dưỡng (Protein, Carbs, Fat, Fiber) từ hình ảnh bữa ăn:
- Giao diện Web: Lưu trữ giao diện trên AWS Amplify Hosting với tên miền HTTPS bảo mật và tự động tích hợp CI/CD với GitHub.
- Hiệu năng và chi phí: Mô hình ONNX (15.5 MB) chạy trên AWS Lambda xử lý nhanh chóng, tối ưu hóa chi phí vận hành nhờ kiến trúc Serverless tính phí theo mức độ sử dụng thực tế.
- Cơ chế dự phòng Rekognition: Khi mô hình chính có độ tin cậy dưới 60% do chất lượng ảnh kém, hệ thống kích hoạt Amazon Rekognition để đối chiếu lại. Nếu không khớp được, hệ thống trả về mã lỗi HTTP 422 thay vì đoán sai. Ảnh độ tin cậy thấp được lưu tự động về S3 để phục vụ đánh giá sau.
- Giám sát hệ thống: Tự động gửi email cảnh báo qua Amazon SNS khi phát sinh lỗi hệ thống hoặc vượt ngưỡng ngân sách.

---

### 3. Kiến trúc & Luồng hệ thống

![Kiến trúc hệ thống NutriVision](/FCAJ-Project/images/2-Proposal/solution_architecture.drawio.png)

Hệ thống hoạt động theo 4 nhóm luồng chính trên sơ đồ kiến trúc:

#### A. Quy trình đóng gói và triển khai
1. Đóng gói và đẩy Docker Image: Nhà phát triển đóng gói mã nguồn xử lý và mô hình ONNX vào Docker Image, sau đó đẩy lên kho lưu trữ Amazon ECR.
2. Khởi tạo hàm Lambda: AWS Lambda kéo Container Image từ ECR để thực thi suy luận trực tiếp.
3. Lưu trữ tệp mô hình: Tải tệp mô hình food_model.onnx (15.5 MB) và bảng dữ liệu dinh dưỡng calorie_map.json lên Amazon S3 làm kho lưu trữ trung tâm.

#### B. Luồng người dùng và suy luận thời gian thực
4. Truy cập ứng dụng: Người dùng truy cập ứng dụng thông qua tên miền HTTPS được phân phối bởi AWS Amplify Hosting.
5. Gửi request phân tích: Giao diện Web gửi hình ảnh bữa ăn và hệ số khẩu phần ăn (0.7x – 2.0x) qua request POST /predict.
6. Định tuyến API: API Gateway tiếp nhận request, xử lý CORS, áp dụng giới hạn tần suất gọi chống spam (20 req/s) và chuyển tiếp payload đến hàm Lambda.
7. Suy luận mô hình: Lambda giải mã ảnh, kiểm tra chất lượng ảnh và chạy suy luận AI với mô hình EfficientNet-B0 ONNX.

#### C. Cơ chế phân nhánh và dự phòng
8. Phản hồi kết quả: Với ảnh có độ tin cậy từ 60% trở lên, Lambda tra cứu hàm lượng dinh dưỡng theo khẩu phần và trả kết quả JSON về giao diện.
9. Xử lý ngoại lệ: Với ảnh có độ tin cậy dưới 60%, Lambda kích hoạt Rekognition quét nhãn tổng quan để đối chiếu; nếu không khớp sẽ trả về mã lỗi HTTP 422, đồng thời lưu tệp ảnh vào thư mục ood_logs trên S3.

#### D. Giám sát vận hành và cảnh báo
10. Ghi nhận log và chỉ số: AWS Lambda ghi lại nhật ký thực thi và số liệu hiệu năng tại CloudWatch.
11. Cảnh báo sự cố: Khi CloudWatch Alarm phát hiện số lượng lỗi vượt ngưỡng, hệ thống tự động kích hoạt Amazon SNS gửi email cảnh báo đến kỹ sư vận hành.

---

### 4. Dịch vụ AWS sử dụng

| Dịch vụ AWS | Vai trò |
|---|---|
| AWS Amplify | Hosting giao diện Web tĩnh (Frontend), tích hợp CI/CD với GitHub, phân phối qua CloudFront CDN và chứng chỉ HTTPS. |
| Amazon API Gateway | Quản lý REST API, cấu hình CORS và giới hạn tần suất gọi (20 req/s). |
| AWS Lambda | Tính toán Serverless (Python 3.12, ONNX Runtime), suy luận và tính calo theo khẩu phần. |
| Amazon ECR | Lưu trữ và quản lý Docker Container Image cho AWS Lambda. |
| Amazon S3 | Lưu trữ mô hình ONNX (15.5 MB), calorie_map.json và ảnh ngoại lệ ood_logs. |
| Amazon Rekognition | Cơ chế AI dự phòng khi mô hình chính có độ tin cậy dưới 60%. |
| Amazon CloudWatch | Giám sát độ trễ, lưu trữ log groups và cấu hình alarms. |
| Amazon SNS | Gửi email cảnh báo sự cố thời gian thực cho kỹ sư quản trị. |

---

### 5. Kết quả kỳ vọng
1. Triển khai hoàn chỉnh hệ thống Serverless trên AWS với giao diện hosted trên AWS Amplify và API Gateway hoạt động ổn định.
2. Nắm được cách chuyển đổi mô hình PyTorch sang ONNX (15.5 MB) và đóng gói môi trường suy luận nhẹ với Docker Container.
3. Hoàn thành các kịch bản kiểm thử: nhận diện món ăn, kích hoạt fallback Rekognition và bắt lỗi ảnh không hợp lệ (HTTP 422).
4. Thiết lập cảnh báo email qua SNS và dọn dẹp sạch tài nguyên sau khi hoàn thành bài thực hành.
