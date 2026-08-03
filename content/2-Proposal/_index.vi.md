---
title: "Bản đề xuất"
date: 2026-08-03
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# AI NutriVision
## Hệ thống nhận diện món ăn và phân tích dinh dưỡng tự động trên hạ tầng AWS Serverless

### 1. Tóm tắt điều hành
Dự án **AI NutriVision** giải quyết bài toán theo dõi chế độ ăn uống bằng cách tự động hóa quy trình nhận diện món ăn và tính toán hàm lượng calo cùng các chỉ số dinh dưỡng (Protein, Carbs, Fat, Fiber) từ hình ảnh bữa ăn. 

Mô hình được huấn luyện dựa trên tập dữ liệu **Food-101** (gồm 101,000 hình ảnh chia đều cho 101 nhóm món ăn). Để tối ưu hóa cho bài toán theo dõi dinh dưỡng thực tế, dự án đã chọn lọc dữ liệu (data cleaning) để chọn ra **50 nhóm món ăn có tần suất tiêu thụ cao nhất** (tương ứng 50,000 ảnh). Mô hình học sâu EfficientNet-B0 được tinh chỉnh (Fine-Tuning) trên tập dữ liệu này, đạt độ chính xác **Test Top-1 Accuracy 85.62%** (Weighted F1-Score 0.86) trên 5,000 ảnh test độc lập. 

Mô hình được nén sang định dạng ONNX (15.5 MB) và vận hành hoàn toàn trên kiến trúc Serverless của AWS (API Gateway, Lambda, S3, DynamoDB, Rekognition, CloudWatch). Giải pháp giúp giảm thời gian ghi chép dinh dưỡng từ 5 phút xuống còn dưới 3 giây mỗi bữa ăn, với thời gian suy luận khi Warm Start của Lambda đạt **35–50 ms** (tổng thời gian phản hồi giao diện Web < 150 ms) và khả năng tối ưu hóa chi phí vượt trội so với máy chủ truyền thống.

### 2. Tuyên bố vấn đề
#### Vấn đề thực tế
Việc tính toán hàm lượng calo và dinh dưỡng (Protein, Carbs, Fat) hiện nay vẫn phụ thuộc nhiều vào các ứng dụng nhập liệu thủ công (như MyFitnessPal, Yazio). Người dùng phải tự gõ tên từng món ăn, ước lượng khối lượng và tra cứu thủ công. Quy trình này trải qua nhiều bước rườm rà, dẫn đến việc bỏ dở giữa chừng sau vài ngày sử dụng.

#### Giải pháp đề xuất
**AI NutriVision** cung cấp giao diện web cho phép người dùng chỉ cần chụp hoặc tải ảnh bữa ăn lên hệ thống. Ảnh được gửi qua API Gateway đến AWS Lambda để mô hình ONNX nhận diện món ăn với thời gian tính toán nội bộ trong khoảng 35–50 ms (khi warm) và trả về bảng calo chi tiết. 
- **Cơ chế Fallback AI linh hoạt**: Để xử lý các trường hợp món ăn nằm ngoài danh mục 50 món (Out-of-Distribution), hệ thống tự động tích hợp **Amazon Rekognition** khi độ tin cậy của mô hình chính dưới 60%. Rekognition trả về các nhãn tổng quan (như "Dish", "Noodle", "Soup"), Lambda sẽ thực hiện khớp từ khóa (Fuzzy Matching) với bảng dữ liệu hoặc gán giá trị dinh dưỡng mặc định theo nhóm món (`general_food`), đồng thời lưu ảnh về `s3://.../ood_logs/` để tái huấn luyện.
- **Tính toán khẩu phần ăn linh hoạt**: Hệ thống cho phép người dùng tùy chọn hệ số khẩu phần trực tiếp trên Web UI (Nhỏ 0.7x, Vừa 1.0x, Lớn 1.5x, Đặc biệt 2.0x), từ đó Lambda tự động nhân tỷ lệ Calo và Macros chính xác theo lượng ăn thực tế.
- Lịch sử nhận diện và thông số dinh dưỡng được lưu trữ tự động vào **Amazon DynamoDB** để xây dựng nhật ký ăn uống cho người dùng.

#### Lợi ích và hiệu quả đầu tư (ROI)
- **Tối ưu thời gian**: Giảm 90% thời gian nhập liệu dinh dưỡng hàng ngày cho người tập thể hình, người ăn kiêng hoặc bệnh nhân cần theo dõi chỉ số chỉ bằng một thao tác chụp ảnh.
- **Hiệu quả chi phí**: Kiến trúc Serverless chỉ phát sinh chi phí khi có request thực tế, không tốn chi phí duy trì máy chủ 24/7, giúp tối ưu hóa ngân sách vận hành tối đa.

### 3. Kiến trúc giải pháp
![Sơ đồ kiến trúc AI NutriVision](/FCAJ-Project/images/2-Proposal/architecture.png)

#### Các dịch vụ AWS sử dụng:
1. **Amazon API Gateway**: Tiếp nhận request HTTPS `POST /predict` từ giao diện Web, xác thực API Key, xử lý cấu hình CORS, chống tấn công DDoS và giới hạn lưu lượng (Rate Limiting 20 req/s).
2. **AWS Lambda**: Hàm Serverless tính toán chạy môi trường Python 3.11, đảm nhận việc giải mã ảnh base64, kiểm tra chất lượng ảnh mờ/tối (Quality Check), chạy mô hình ONNX Runtime và tính calo theo khẩu phần do người dùng chọn (0.7x – 2.0x).
3. **Amazon S3**: Lưu trữ tập trung tệp mô hình `food_model.onnx` (15.5 MB), bảng tra cứu calo `calorie_map.json` và nhật ký lưu ảnh món ăn lạ (`ood_logs/`). Áp dụng S3 Lifecycle Policy tự động hủy logs sau 90 ngày để tiết kiệm chi phí.
4. **Amazon Rekognition**: Dịch vụ AI dùng làm phương án dự phòng (Fallback Engine). Khi mô hình ONNX chính đạt độ tin cậy < 60%, Lambda kích hoạt Rekognition để quét nhãn tổng quan, tránh trường hợp hệ thống không phản hồi.
5. **Amazon DynamoDB**: Cơ sở dữ liệu NoSQL lưu trữ lịch sử các lượt nhận diện (`prediction_id`, `timestamp`, `food_class`, `confidence`, `calories`, `macronutrients`).
6. **Amazon CloudWatch**: Giám sát hiệu năng Lambda, thời gian phản hồi API (P95 Latency), tỷ lệ lỗi và phát cảnh báo Alarms khi hệ thống gặp sự cố.

### 4. Triển khai kỹ thuật & Tối ưu MLOps
Quá trình xây dựng dự án được chia làm 2 giai đoạn chính:

#### Giai đoạn 1: Chuẩn hóa dữ liệu, huấn luyện & nén mô hình AI (Google Colab GPU)
- **Sàng lọc & Dọn dẹp dữ liệu (Data Cleaning)**: Tập dữ liệu gốc Food-101 chứa 101,000 ảnh cho 101 danh mục món ăn. Qua phân tích nhu cầu theo dõi dinh dưỡng thực tế, 50 lớp món ăn phổ biến nhất đã được lựa chọn dựa trên 3 tiêu chí:
  1. *Tần suất xuất hiện trong khẩu phần ăn*: Ưu tiên các món ăn phổ biến trong ẩm thực Á – Âu (Phở, Cơm chiên, Pizza, Sushi, Hamburger, Steak, Salad...).
  2. *Độ phức tạp dinh dưỡng*: Chọn lọc các món có cấu trúc Macro đa dạng (Protein, Carbs, Fat) cần kiểm soát calo nghiêm ngặt.
  3. *Loại bỏ yếu tố dư thừa*: Loại bỏ các món quá đặc thù vùng miền hoặc ít có nhu cầu tra cứu calo trong các ứng dụng fitness.
  Tập dữ liệu 50 món (50,000 ảnh) được chia theo tỷ lệ: 37,500 ảnh Train (750 ảnh/lớp), 7,500 ảnh Validation (150 ảnh/lớp) và 5,000 ảnh Test độc lập (100 ảnh/lớp).
- **Kỹ thuật huấn luyện & Đánh giá**: Sử dụng mạng `EfficientNet-B0` với quy trình Fine-Tuning 2 giai đoạn (Freeze Backbone 3 epochs; Unfreeze full backbone với Cosine Annealing 7 epochs). Kết quả đạt **Test Top-1 Accuracy 85.62%** và Weighted F1-Score 0.86 trên 5,000 ảnh test.
- **Nén mô hình & Hướng tối ưu INT8**: Chuyển đổi mô hình PyTorch (`.pth`) sang định dạng tĩnh `food_model.onnx` (15.5 MB). Trong tương lai có thể áp dụng Lượng tử hóa INT8 (ONNX Quantization) để nén model xuống **~1.8 MB**, giảm độ trễ thêm 40%.

#### Giai đoạn 2: Triển khai hạ tầng Cloud & CI/CD (AWS CloudFormation / SAM)
- Đóng gói mã nguồn Lambda cùng các thư viện phụ thuộc (`onnxruntime`, `Pillow`, `boto3`).
- Viết tệp cấu hình IaC `infrastructure/template.yaml` (AWS SAM / CloudFormation) tích hợp quy trình CI/CD tự động hóa việc kiểm thử và triển khai chỉ bằng 1 dòng lệnh CLI.
- **Kế hoạch Re-train định kỳ**: Ảnh món lạ thu thập tại `s3://.../ood_logs/` sẽ được đánh giá hàng tháng. Khi đạt 1,000 ảnh mới, hệ thống tự động kích hoạt pipeline re-train để cập nhật mô hình lên danh mục 100+ món ăn.

### 5. Bảo mật & Quy định riêng tư (Security & Privacy)
- **Xác thực & Phân quyền**: Giới hạn truy cập API bằng API Keys / Cognito User Pools. Áp dụng IAM Least-Privilege Role cho Lambda (chỉ có quyền đọc S3 bucket và ghi DynamoDB table chỉ định).
- **Mã hóa dữ liệu**: Kích hoạt mã hóa Server-Side Encryption (SSE-S3) trên Amazon S3 và mã hóa dữ liệu DynamoDB at-rest.
- **Quản lý vòng đời dữ liệu**: Cấu hình S3 Lifecycle Rule tự động xóa ảnh nhật ký `ood_logs/` sau 90 ngày để đảm bảo quyền riêng tư và tối ưu chi phí lưu trữ.

### 6. Lộ trình & Mốc thực hiện
- **Tuần 1–2**: Phân tích tập dữ liệu Food-101, dọn dẹp nhãn nhiễu, lọc 50 lớp món ăn phổ biến, thiết kế sơ đồ kiến trúc bằng Draw.io và phân tích chi phí.
- **Tuần 3–4**: Huấn luyện mô hình AI trên Colab GPU, đạt Test Top-1 Acc 85.62% (F1-Score 0.86), nén ONNX và kiểm thử các trường hợp biên (Edge Cases) tại máy cục bộ.
- **Tuần 5–6**: Viết kịch bản CloudFormation/SAM, triển khai toàn bộ tài nguyên lên vùng `ap-southeast-1` (Singapore) và đấu nối giao diện Web với API Gateway.
- **Tuần 7–8**: Đánh giá hiệu năng hệ thống trên CloudWatch (phân tích Cold Start vs Warm Start), viết tài liệu Workshop step-by-step và kiểm thử quy trình dọn dẹp tài nguyên (`cleanup.sh`).

### 7. Ước tính ngân sách & Quản lý chi phí (Budget Estimation)
Đánh giá chi phí được tính toán dựa trên AWS Pricing Calculator chính thức cho quy mô **100,000 lượt nhận diện/tháng** (tương đương ~3,300 request/ngày - quy mô ứng dụng thực tế):

| Dịch vụ AWS | Thông số sử dụng hàng tháng | Chi phí niêm yết (List Price) |
|---|---|---|
| **Amazon API Gateway** | 100,000 REST API calls ($3.50 / 1M) | $0.35 USD |
| **AWS Lambda** | 100,000 invocations (512MB RAM, 50ms/req) | $0.04 USD |
| **Amazon S3** | 5.0 GB lưu trữ + 10,000 PUT/GET requests | $0.15 USD |
| **Amazon DynamoDB** | 100,000 lượt ghi (WCU) + 2 GB lưu trữ dữ liệu | $1.50 USD |
| **Amazon Rekognition** | ~10,000 lượt gọi Fallback AI (10% tổng request) | $10.00 USD |
| **Amazon CloudWatch** | 5 GB Log Storage + 2 CloudWatch Alarms | $2.50 USD |
| **TỔNG CHI PHÍ HÀNG THÁNG** | **Quy mô 100,000 nhận diện / tháng** | **~ 14.54 USD / tháng** |

> [!TIP] Điểm tối ưu chi phí & Kiểm soát rủi ro Rekognition:
> - **Tối ưu Serverless**: So với việc duy trì máy chủ EC2 GPU 24/7 ($150–$300/tháng), kiến trúc Serverless tiết kiệm >90% chi phí.
> - **Tối ưu chi phí Rekognition (chiếm ~69% ngân sách)**: Để ngăn chi phí Rekognition tăng cao nếu tỷ lệ fallback vượt 10%, hệ thống áp dụng cơ chế Caching kết quả Rekognition theo Hash ảnh trên DynamoDB và điều chỉnh linh hoạt ngưỡng Confidence Threshold sau go-live.

### 8. Đánh giá rủi ro & Phương án xử lý
- **Rủi ro món ăn lạ (Out-of-Distribution)**: Món ăn không nằm trong 50 món đã học ➔ *Xử lý*: Nếu confidence < 60%, tự động kích hoạt Amazon Rekognition quét nhãn, khớp từ khóa fuzzy matching với `calorie_map.json` hoặc gán nhóm dinh dưỡng mặc định, đồng thời lưu ảnh về `s3://.../ood_logs/`.
- **Rủi ro Cold Start của Lambda**: Lần gọi đầu tiên sau khi hết container có độ trễ ~350–500ms ➔ *Xử lý*: Giữ mô hình ONNX siêu nhẹ (15.5 MB) giúp giảm thời gian cold start xuống tối thiểu. Với môi trường production cần cam kết SLA < 150ms nhất quán, có thể kích hoạt AWS Lambda Provisioned Concurrency.
- **Rủi ro chất lượng ảnh đầu vào**: Ảnh chụp quá mờ, quá tối hoặc file hỏng ➔ *Xử lý*: Hàm kiểm tra chất lượng ảnh (Quality Check) trên Lambda sẽ chặn ngay lập tức và trả về mã lỗi HTTP 400/422 kèm thông báo hướng dẫn người dùng chụp lại.
- **Rủi ro vượt ngưỡng chi phí**: Bị spam request liên tục ➔ *Xử lý*: Thiết lập Throttling trên API Gateway (giới hạn 20 req/s) và cài đặt CloudWatch Budget Alarm cảnh báo qua email khi chi phí vượt quá ngưỡng cho phép.

### 9. Kết quả kỳ vọng
1. **Về kỹ thuật**: Xây dựng thành công hệ thống Serverless hoàn chỉnh kết hợp AI Computer Vision với thời gian suy luận nội bộ của Lambda khi warm đạt 35–50ms (tổng thời gian phản hồi web < 150ms), khả năng tự động mở rộng (Auto-scaling) và xử lý 100% các trường hợp biên. Mô hình phân loại chuẩn xác 50 nhóm món ăn phổ biến nhất với **Test Top-1 Accuracy 85.62%** (Weighted F1-Score 0.86).
2. **Về giá trị thực tiễn**: Cung cấp giải pháp nhận diện dinh dưỡng tiện lợi cho người dùng, đóng vai trò là mẫu bài lab chuẩn hóa cho cộng đồng FCAJ/AWS về việc đưa mô hình AI/ML lên môi trường Serverless.