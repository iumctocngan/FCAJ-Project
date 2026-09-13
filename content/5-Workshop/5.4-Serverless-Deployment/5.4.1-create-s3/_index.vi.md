---
title: "Tạo S3 Bucket & Tải mô hình ONNX"
date: 2026-08-04
weight: 1
chapter: false
pre: " <b> 5.4.1 </b> "
---

#### Các bước khởi tạo S3 Bucket và tải tài nguyên mô hình AI

Khởi tạo bộ lưu trữ Amazon S3 trên AWS Management Console để chứa tệp mô hình nén ONNX (15.5 MB) và dữ liệu tra cứu calo:

1. Truy cập Amazon S3 Console:
   - Đăng nhập vào [AWS Management Console](https://console.aws.amazon.com/s3/).
   - Tại giao diện Amazon S3, chọn nút Create bucket.

![Màn hình Amazon S3 - Create bucket](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step1_create.png)

2. Cấu hình tên bucket và region:
   - AWS Region: Chọn Asia Pacific (Singapore) ap-southeast-1.
   - Bucket type: Chọn General purpose.
   - Bucket namespace: Chọn Account Regional namespace (recommended).
   - Bucket name prefix: Nhập `fcaj-food-ai-storage-2026-sg` (AWS sẽ tự động ghép thêm Account ID và Region thành tên duy nhất).

![Cấu hình Tên Bucket & Region](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step2_config.png)

3. Cấu hình quyền bảo mật (Block Public Access):
   - Object Ownership: Giữ mặc định ACLs disabled (recommended).
   - Block Public Access settings for this bucket: Tích chọn Block all public access để bảo vệ dữ liệu mô hình.

![Cấu hình Quyền Bảo mật](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step3_security.png)

   - Cuộn xuống dưới và chọn Create bucket.

4. Xác nhận khởi tạo bucket thành công:
   - Màn hình hiển thị thông báo tạo bucket thành công.
   - Nhấp vào tên bucket vừa tạo để mở trang chi tiết quản lý đối tượng.

![Xác nhận Khởi tạo Bucket Thành công](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step4_success.png)

5. Tải tệp mô hình food_model.onnx lên S3:
   - Tại thư mục gốc của bucket, chọn Upload. Chọn tệp mô hình food_model.onnx (15.5 MB) từ máy cá nhân.
   - Nhấp nút Upload ở góc dưới. Màn hình báo trạng thái Succeeded (100.00%).

![Upload Tệp Mô hình ONNX Thành công](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step5_upload.png)

6. Tạo thư mục data trong bucket:
   - Quay lại trang chi tiết bucket, chọn Create folder.
   - Folder name: Nhập `data`.
   - Nhấp Create folder để hoàn tất tạo thư mục chứa dữ liệu.

![Tạo Thư mục data trong Bucket](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step6_create_folder_data.png)

7. Tải tệp calorie_map.json vào thư mục data:
   - Mở thư mục data vừa tạo.
   - Chọn Upload và chọn tệp calorie_map.json (8.5 KB) từ máy cá nhân.
   - Nhấp Upload. Màn hình báo trạng thái Succeeded (100.00%) tại đường dẫn s3://.../data/calorie_map.json.

![Upload Tệp calorie_map.json vào Thư mục data Thành công](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step7_upload_calorie_map.png)
