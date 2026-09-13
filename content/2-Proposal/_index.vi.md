---
title: "Bản đề xuất"
date: 2026-08-03
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# NutriVision
## Hệ thống nhận diện món ăn và phân tích dinh dưỡng tự động trên hạ tầng AWS Serverless

### 1. Tóm tắt
Dự án NutriVision giải quyết bài toán theo dõi chế độ ăn uống bằng cách tự động hóa quy trình nhận diện món ăn và tính toán hàm lượng calo cùng các chỉ số dinh dưỡng (Protein, Carbs, Fat, Fiber) từ hình ảnh bữa ăn dựa trên mô hình Thị giác máy tính.

Mô hình được huấn luyện dựa trên tập dữ liệu Food-101 (gồm 101,000 hình ảnh chia đều cho 101 nhóm món ăn). Để tối ưu hóa cho bài toán theo dõi dinh dưỡng thực tế, dự án đã chọn lọc ra 50 nhóm món ăn có tần suất tiêu thụ cao nhất (tương ứng 50,000 ảnh). Mô hình EfficientNet-B0 được tinh chỉnh trên tập dữ liệu này, đạt độ chính xác Test Top-1 Accuracy 85.62% (Weighted F1-Score 0.86) trên 5,000 ảnh test độc lập.

Mô hình sau đó được chuyển đổi sang định dạng ONNX (15.5 MB) và vận hành trên kiến trúc Serverless của AWS (Amplify, API Gateway, Lambda, ECR, S3, Rekognition, CloudWatch, SNS). Giải pháp giúp đơn giản hóa quy trình ghi chép dinh dưỡng chỉ còn vài giây mỗi bữa ăn với chi phí vận hành tối ưu nhờ cơ chế tính phí theo mức độ sử dụng thực tế.

### 2. Đặt vấn đề
#### Vấn đề thực tế
Việc tính toán hàm lượng calo và dinh dưỡng hiện nay vẫn phụ thuộc nhiều vào các ứng dụng nhập liệu thủ công (như MyFitnessPal, Yazio). Người dùng phải tự tìm tên từng món ăn, ước lượng khối lượng và tra cứu thủ công. Quy trình này trải qua nhiều bước rườm rà, dẫn đến việc bỏ dở giữa chừng sau một thời gian sử dụng.

#### Giải pháp đề xuất
NutriVision cung cấp giao diện web được lưu trữ trực tiếp trên AWS Amplify Hosting, hỗ trợ truy cập HTTPS từ mọi thiết bị mà không yêu cầu đăng ký tài khoản phức tạp. Người dùng chỉ cần chụp hoặc tải ảnh bữa ăn lên hệ thống. Ảnh được gửi qua API Gateway đến AWS Lambda để nhận diện món ăn nhanh chóng và trả về bảng calo chi tiết.
- Cơ chế dự phòng với Amazon Rekognition: Khi mô hình ONNX chính có độ tin cậy dưới 60% (do góc chụp khó, ánh sáng yếu hoặc bị che khuất), hệ thống kích hoạt Amazon Rekognition để quét nhãn tổng quan và cố gắng ánh xạ sang một trong 50 món đã định nghĩa. Nếu không khớp được, hệ thống chủ động trả về mã lỗi HTTP 422 thay vì đưa ra kết quả sai lệch. Toàn bộ ảnh độ tin cậy thấp được lưu vào S3 để xem xét và cải thiện mô hình sau này.
- Tính toán theo khẩu phần ăn linh hoạt: Hệ thống cho phép người dùng tùy chọn hệ số khẩu phần trực tiếp trên giao diện (Nhỏ 0.7x, Vừa 1.0x, Lớn 1.5x, Đặc biệt 2.0x), từ đó Lambda tự động nhân tỷ lệ Calo và dinh dưỡng tương ứng.

#### Tính thực tiễn và lợi ích của giải pháp
- Giảm thời gian ghi chép: Rút ngắn thao tác nhập liệu dinh dưỡng hàng ngày cho người tập thể hình, người ăn kiêng hoặc người cần theo dõi chỉ số sức khỏe bằng một thao tác chụp ảnh.
- Tiết kiệm chi phí: Kiến trúc Serverless chỉ phát sinh chi phí khi có request thực tế, không tốn chi phí duy trì máy chủ liên tục.

### 3. Kiến trúc giải pháp
![NutriVision Architecture Diagram](/FCAJ-Project/images/2-Proposal/solution_architecture.drawio.png)

#### Các dịch vụ AWS sử dụng:
- AWS Amplify: Lưu trữ giao diện web tĩnh (Frontend), tích hợp CI/CD với GitHub, phân phối qua CloudFront CDN và cấp chứng chỉ HTTPS.
- Amazon API Gateway: Quản lý REST API endpoint POST /predict, cấu hình CORS và áp dụng giới hạn tần suất gọi chống spam (Rate Limiting 20 req/s).
- AWS Lambda: Xử lý tính toán Serverless (Python 3.12, ONNX Runtime, Boto3), giải mã ảnh, kiểm tra định dạng và tính calo theo khẩu phần.
- Amazon ECR: Lưu trữ và quản lý Docker Container Image của Lambda, đóng gói mã nguồn và các thư viện cần thiết.
- Amazon S3: Lưu trữ tệp mô hình food_model.onnx (15.5 MB), bảng tra cứu calo calorie_map.json và nhật ký hình ảnh chất lượng thấp.
- Amazon Rekognition: Cơ chế AI dự phòng khi mô hình chính có độ tin cậy dưới 60%.
- Amazon CloudWatch: Thu thập log, theo dõi độ trễ và thiết lập cảnh báo khi hệ thống phát sinh lỗi.
- Amazon SNS: Gửi email cảnh báo sự cố đến kỹ sư quản trị khi có cảnh báo từ CloudWatch.

### 4. Triển khai kỹ thuật và quy trình MLOps

Quá trình xây dựng dự án được chia làm 2 giai đoạn:

#### Giai đoạn 1: Xử lý dữ liệu, huấn luyện và xuất mô hình (Google Colab GPU)
- Sàng lọc dữ liệu: Từ tập dữ liệu gốc Food-101 (101 món), dự án chọn lọc 50 nhóm món ăn phổ biến nhất (50,000 ảnh) dựa trên 3 tiêu chí:
  - Độ phổ biến và tần suất tiêu thụ: Ưu tiên các món ăn quen thuộc của ẩm thực Á Đông (Phở, Cơm chiên, Pad Thai, Sushi, Bibimbap, Gyoza) và Âu - Mỹ (Hamburger, Pizza, Steak, Spaghetti, Club Sandwich, Mac & Cheese).
  - Đa dạng nhóm dinh dưỡng: Phân bổ đều qua các nhóm giàu đạm, món cơm - mì, đồ ăn nhanh, salad ăn kiêng, đồ ăn sáng và tráng miệng.
  - Khả năng phân biệt thị giác: Hạn chế chọn các món quá tương đồng nhau để mô hình đạt độ chính xác cao khi phân loại.
  - Tập dữ liệu 50 món được chia theo tỷ lệ chuẩn: 37,500 ảnh Train (750 ảnh/lớp), 7,500 ảnh Validation (150 ảnh/lớp) và 5,000 ảnh Test độc lập (100 ảnh/lớp).
- Huấn luyện và xuất ONNX: Sử dụng mạng EfficientNet-B0 với quy trình Fine-Tuning 2 giai đoạn (Freeze và Unfreeze Backbone). Kết quả đạt Test Top-1 Accuracy 85.62% trên 5,000 ảnh test. Mô hình được xuất sang định dạng tĩnh food_model.onnx (15.5 MB) giúp giảm dung lượng và tối ưu tốc độ nạp mô hình.

#### Giai đoạn 2: Triển khai Cloud và giám sát vận hành
- Tự động hóa CI/CD cho Frontend: Kết nối mã nguồn giao diện từ GitHub lên AWS Amplify Hosting. Mỗi khi có thay đổi code được push lên nhánh main, AWS Amplify sẽ tự động build và deploy bản mới nhất.
- Triển khai backend Serverless: Đóng gói Lambda function thành Docker Image, đẩy lên Amazon ECR và cấu hình các tài nguyên AWS (Lambda, API Gateway, S3, Rekognition, CloudWatch, SNS) qua AWS Console kết hợp AWS CLI.
- Giám sát và ghi nhận ngoại lệ:
  - Hệ thống ghi nhận log thực thi và thời gian suy luận tại CloudWatch Log Groups.
  - Khi ảnh tải lên nằm ngoài 50 lớp đã học hoặc có độ tin cậy thấp (< 60%), hệ thống lưu ảnh về thư mục ood_logs trên S3 để phục vụ đánh giá và cải thiện mô hình sau này.

### 5. Lộ trình và mốc thực hiện
Dự án được triển khai trong 3 tuần cuối của kỳ thực tập (Tuần 8 – Tuần 10):
- Tuần 8: Khảo sát bài toán, thiết kế kiến trúc hệ thống, chọn lọc dữ liệu từ Food-101 (50 nhóm món), fine-tune mô hình EfficientNet-B0 và xuất sang định dạng ONNX.
- Tuần 9: Xây dựng backend suy luận Serverless trên AWS Lambda (Docker container), tích hợp S3, cơ chế fallback Amazon Rekognition, API Gateway và triển khai Web UI lên AWS Amplify Hosting.
- Tuần 10: Kiểm thử end-to-end toàn hệ thống, tối ưu độ trễ, cấu hình giám sát CloudWatch và cảnh báo SNS, hoàn thiện tài liệu workshop và nghiệm thu dự án.

### 6. Ước tính ngân sách và quản lý chi phí

| Dịch vụ AWS | Chi phí ước tính |
|---|---|
| AWS Amplify Hosting | ~$0.50/tháng |
| Amazon API Gateway | ~$0.35/tháng |
| AWS Lambda | ~$0.00/tháng |
| Amazon ECR | ~$0.03/tháng |
| Amazon S3 (Lưu trữ và requests) | ~$0.15/tháng |
| Amazon Rekognition (AI dự phòng) | ~$5.00/tháng |
| Amazon CloudWatch | ~$0.02/tháng |
| Amazon SNS | ~$0.00/tháng |
| Tổng chi phí ước tính | ~$6.05 USD/tháng |

> [!NOTE] 
> Chi phí tiêu chuẩn theo mức sử dụng là khoảng ~$6.05 USD/tháng cho quy mô khoảng 100,000 lượt nhận diện. Với tài khoản trong thời gian AWS Free Tier, hầu hết dịch vụ hạ tầng đều được miễn phí nên chi phí thực tế chỉ phát sinh rất nhỏ (chủ yếu nếu lượt gọi Rekognition vượt mức miễn phí 5,000 lượt).

### 7. Đánh giá rủi ro và phương án xử lý
- Món ăn nằm ngoài danh mục đã học: Khi độ tin cậy dưới 60%, hệ thống tự động kích hoạt Amazon Rekognition và thử đối chiếu sang 50 món đã định nghĩa. Nếu không ánh xạ được, hệ thống chủ động trả về mã lỗi HTTP 422 thay vì đưa ra kết quả sai lệch, đảm bảo tính trung thực về dữ liệu dinh dưỡng. Ảnh này cũng được lưu vào S3 để phục vụ đánh giá sau.
- Độ trễ khởi động của Lambda (Cold Start): Tối ưu hóa mã nguồn Python và import tối giản các thư viện cần thiết giúp giảm thời gian cold start. Đối với môi trường thực tế cần cam kết SLA ổn định, có thể cân nhắc kích hoạt Provisioned Concurrency.
- Chất lượng ảnh đầu vào: Kiểm tra định dạng và chất lượng ảnh ngay tại hàm Lambda để chặn sớm các file không hợp lệ và phản hồi hướng dẫn người dùng chụp lại.
- Vượt ngưỡng chi phí hoặc lỗi hệ thống: Thiết lập giới hạn tần suất gọi trên API Gateway (20 req/s) và cài đặt CloudWatch Budget Alarm tự động kích hoạt Amazon SNS gửi email cảnh báo cho quản trị viên.

### 8. Kết quả kỳ vọng
- Về mặt kỹ thuật: Xây dựng thành công hệ thống Serverless hoàn chỉnh kết hợp mô hình Thị giác máy tính nhận diện 50 nhóm món ăn phổ biến, giao diện web chạy trên AWS Amplify và backend tự động co giãn theo lượng truy cập thực tế.
- Về mặt vận hành: Không phát sinh chi phí duy trì máy chủ khi không có request so với EC2 truyền thống nhờ kiến trúc Serverless, thời gian phản hồi nhanh và chi phí vận hành chỉ vài USD mỗi tháng.
- Về mặt tài liệu: Cung cấp bộ tài liệu hướng dẫn thực hành (Workshop) chi tiết từng bước, kèm hình ảnh minh họa thực tế trên AWS Console, giúp người học dễ dàng tự triển khai lại giải pháp.