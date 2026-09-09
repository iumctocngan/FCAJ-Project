---
title: "Hosting Web UI trên AWS Amplify"
date: 2026-08-04
weight: 4
chapter: false
pre: " <b> 5.4.4 </b> "
---

#### Hosting Giao diện Frontend trên AWS Amplify

Sau khi tạo xong API Gateway REST API, bạn sẽ đưa giao diện trang web NutriVision lên môi trường Internet 24/7 với tên miền bảo mật HTTPS bằng dịch vụ AWS Amplify Hosting.

#### 1. Chuẩn bị Mã nguồn Frontend:
Đảm bảo thư mục giao diện Web chứa đủ các tệp tin:
- index.html (Khung giao diện Web)
- styles.css (Định dạng giao diện & hiệu ứng)
- app.js (Mã xử lý gửi request POST tới API Gateway)

Push toàn bộ mã nguồn lên repository GitHub của bạn (`https://github.com/iumctocngan/NutriVision`).

#### 2. Khởi tạo AWS Amplify App trên Console GUI:
1. Đăng nhập vào [AWS Amplify Console](https://ap-southeast-1.console.aws.amazon.com/amplify/home?region=ap-southeast-1).
2. Nhấp chọn **Create new app** ➔ Chọn nguồn mã nguồn **GitHub** (hoặc Deploy without Git provider).
3. Kết nối với Repository **iumctocngan/NutriVision** và chọn nhánh **main**.

![Chọn Repository và nhánh trên AWS Amplify](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/amplify_select_repo_branch.png)

4. Tại bước **App settings**:
   - **App name**: Nhập `NutriVision`.
   - **Frontend build command**: Để trống (ứng dụng Web tĩnh HTML/CSS/JS thuần).
   - **Build output directory**: Nhập `/`.

![AWS Amplify App Settings](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/amplify_step1_app_settings.png)

5. Chuyển sang bước **Review** ➔ Kiểm tra cấu hình và nhấp nút màu tím **Save and deploy**.

![AWS Amplify Review and Save and Deploy](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/amplify_step2_save_deploy.png)

#### 3. Truy cập đường dẫn Web Public HTTPS:
1. Sau khi quá trình Build & Deploy hoàn tất (khoảng 30 giây), màn hình quản lý **NutriVision: Overview** hiển thị trạng thái phát hành thành công kèm nút **Visit deployed URL**.

![AWS Amplify Overview - Visit deployed URL](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/amplify_step3_visit_deployed_url.png)

2. Nhấp vào nút **Visit deployed URL** để mở ứng dụng Web:  
👉 `https://main.dnrnzxbbonuba.amplifyapp.com`

- Mở đường dẫn trên bất kỳ trình duyệt thiết bị di động hoặc máy tính.
- Giao diện Web đã sẵn sàng kết nối trực tiếp với API Gateway Endpoint cho bước kiểm thử toàn trình!
