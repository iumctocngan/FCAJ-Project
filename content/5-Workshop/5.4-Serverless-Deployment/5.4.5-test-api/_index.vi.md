---
title: "Kiểm thử API & Xử lý Edge Cases"
date: 2026-08-04
weight: 5
chapter: false
pre: " <b> 5.4.5 </b> "
---

#### Quy trình Kiểm thử Toàn trình & Xử lý các Trường hợp Biên (Edge Cases)

Trong bước này, bạn sẽ mở địa chỉ trang web AWS Amplify, kết nối với Invoke URL từ API Gateway và kiểm thử toàn bộ hệ thống qua các kịch bản thực tế.

#### 1. Đấu nối Endpoint URL vào Giao diện Web UI:
- Mở giao diện trang web vừa host trên AWS Amplify (URL được tạo sau khi deploy thành công ở bước 5.4.4).
- Dán đường dẫn Invoke URL vừa lấy từ API Gateway (có dạng `https://<id>.execute-api.ap-southeast-1.amazonaws.com/prod/predict`) vào ô Endpoint Config ở góc trên bên phải giao diện Web.

#### 2. Kịch bản 1 — Nhận diện Món ăn Chuẩn (Happy Path):
- Kéo thả ảnh một món ăn thuộc 50 nhóm đã học (ví dụ: Bánh Táo Apple Pie, Phở Bò, Pizza) vào vùng tải ảnh ➔ Chọn khẩu phần Vừa (1.0x) ➔ Nhấp Nhận diện Calo.
- **Kết quả kỳ vọng**: Trả về tên món ăn chuẩn xác, lượng calo ước tính và biểu đồ dinh dưỡng Protein / Carbs / Fat / Fiber.

![Test Web UI Success Result](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/web_ui_test_success.png)

#### 3. Kịch bản 2 — Kiểm thử AI Dự phòng Rekognition (Fallback — Ảnh chụp khó nhận dạng):
- Tải lên một bức ảnh chụp món ăn thuộc 50 nhóm nhưng ở góc nghiêng lạ hoặc đang chan nước dùng (ví dụ: ảnh chan nước dùng Phở Bò) khiến mô hình chính ONNX có độ tin cậy thấp (55.3% < 60%).
- **Kết quả kỳ vọng**: Độ tin cậy ONNX < 60% ➔ Lambda tự động kích hoạt Amazon Rekognition (`detect_labels`) quét nhãn tổng quan (Soup, Noodle, Broth...) ➔ Ánh xạ thành công sang món Phở Bò Việt Nam (380 KCAL) và ghi log ảnh vào `s3://.../ood_logs/`.
- **Kiểm tra thông số phản hồi**: Giao diện Web hiển thị Engine: Amazon Rekognition fallback và độ tin cậy AI: 55.3%.

![Test Web UI Rekognition Fallback Result](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/web_ui_test_rekognition_fallback.png)

#### 4. Kịch bản 3 — Món ăn ngoài phạm vi (Out-of-Distribution — HTTP 422):
- Tải lên ảnh một món ăn hoàn toàn không nằm trong 50 món đã học và không có từ khóa Rekognition nào khớp (ví dụ: món bánh phô mai que / corn dog hoặc món ăn địa phương hiếm gặp).
- **Kết quả kỳ vọng**: Cả ONNX lẫn Rekognition đều không khớp được ➔ Hệ thống chủ động trả về HTTP 422 thay vì đoán sai. Ảnh OOD được lưu tự động vào `s3://.../ood_logs/` để kỹ sư AI xét duyệt và tái huấn luyện sau.

![Test Web UI OOD Result](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/web_ui_test_ood_422.png)

- **Xác minh ảnh lưu trên Amazon S3**: Mở S3 Console ➔ Bucket fcaj-food-ai-storage-... ➔ Thư mục `ood_logs/YYYY/MM/DD/` để kiểm tra tệp ảnh vừa được hệ thống tự động ghi nhận kèm chỉ số độ tin cậy của ONNX (ví dụ `..._0.1738.jpg`).

![S3 OOD Logs Verification](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/s3_ood_logs_verification.png)

#### 5. Kịch bản 4 — Kiểm tra Chất lượng Ảnh (Ảnh lỗi / Quá tối — HTTP 422):
- Kéo thả một bức ảnh bị tối đen hoàn toàn, quá mờ hoặc lóa sáng.
- **Kết quả kỳ vọng**: Hàm kiểm tra chất lượng ảnh của Lambda (`check_image_quality`) chặn request ngay lập tức trước khi chạy AI, trả về mã lỗi HTTP 422 Unprocessable Entity kèm thông báo: *"Cảnh báo chất lượng ảnh (HTTP 422): Ảnh quá tối. Vui lòng chụp lại ở nơi đủ ánh sáng."*

![Test Web UI Poor Quality 422 Result](/FCAJ-Project/images/5-Workshop/5.4-S3-onprem/web_ui_test_poor_quality_422.png)
