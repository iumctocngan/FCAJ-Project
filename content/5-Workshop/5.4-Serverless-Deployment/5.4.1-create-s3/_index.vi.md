---
title: "Tạo S3 Bucket & Tải mô hình ONNX"
date: 2026-08-04
weight: 1
chapter: false
pre: " <b> 5.4.1 </b> "
---

#### Các bước Khởi tạo S3 Bucket & Upload Tài nguyên Mô hình AI qua AWS Console

Thực hiện các bước khởi tạo bộ lưu trữ đám mây Amazon S3 trên giao diện AWS Management Console để chứa tệp mô hình nén ONNX (15.5 MB) và dữ liệu tra cứu calo:

1. **Truy cập Amazon S3 Console:**
   - Đăng nhập vào [AWS Management Console](https://console.aws.amazon.com/s3/).
   - Tại màn hình Amazon S3, nhấp vào nút **Create bucket**.

![Màn hình Amazon S3 - Create bucket](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step1_create.png)

2. **Cấu hình Tên Bucket & Region:**
   - **AWS Region**: Chọn **Asia Pacific (Singapore) ap-southeast-1**.
   - **Bucket type**: Chọn **General purpose**.
   - **Bucket namespace**: Chọn **Account Regional namespace (recommended)**.
   - **Bucket name prefix**: Nhập `fcaj-food-ai-storage-2026-sg` (AWS sẽ tự động ghép thêm Account ID và Region thành tên duy nhất).

![Cấu hình Tên Bucket & Region](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step2_config.png)

3. **Cấu hình Quyền Bảo mật (Block Public Access):**
   - **Object Ownership**: Giữ mặc định **ACLs disabled (recommended)**.
   - **Block Public Access settings for this bucket**: Tích chọn **Block all public access** để bảo vệ dữ liệu mô hình.

![Cấu hình Quyền Bảo mật](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step3_security.png)

   - Cuộn xuống dưới và nhấp nút **Create bucket**.

4. **Xác nhận Khởi tạo Bucket Thành công:**
   - Màn hình hiển thị thông báo màu xanh tạo bucket thành công.
   - Nhấp trực tiếp vào tên Bucket vừa tạo để mở trang chi tiết quản lý đối tượng.

![Xác nhận Khởi tạo Bucket Thành công](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step4_success.png)

5. **Upload Tệp Mô hình ONNX (food_model.onnx):**
   - Tại thư mục gốc của Bucket, nhấp vào nút **Upload**. Chọn tệp mô hình food_model.onnx (15.5 MB) từ máy cục bộ.
   - Nhấp nút **Upload** ở góc dưới. Màn hình báo trạng thái **Succeeded (100.00%)**.

![Upload Tệp Mô hình ONNX Thành công](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step5_upload.png)

6. **Tạo Thư mục data trong Bucket:**
   - Quay lại trang chi tiết Bucket, nhấp nút **Create folder**.
   - **Folder name**: Nhập `data`.
   - Nhấp nút **Create folder** để hoàn tất khởi tạo thư mục chứa dữ liệu.

![Tạo Thư mục data trong Bucket](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step6_create_folder_data.png)

7. **Upload Tệp calorie_map.json vào Thư mục data:**
   - Truy cập vào thư mục **data** vừa tạo.
   - Nhấp nút **Upload** và chọn tệp calorie_map.json (8.5 KB) từ máy cục bộ.
   - Nhấp **Upload**. Màn hình báo trạng thái **Succeeded (100.00%)** tại đường dẫn `s3://.../data/calorie_map.json`.

![Upload Tệp calorie_map.json vào Thư mục data Thành công](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_step7_upload_calorie_map.png)
