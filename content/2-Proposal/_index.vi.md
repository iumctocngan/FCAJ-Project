---
title: "Bản đề xuất"
date: 2026-08-03
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# NutriVision
## Hệ thống nhận diện món ăn và phân tích dinh dưỡng tự động trên hạ tầng AWS Serverless

### 1. Tóm tắt điều hành
Dự án **NutriVision** giải quyết bài toán theo dõi chế độ ăn uống bằng cách tự động hóa quy trình nhận diện món ăn và tính toán hàm lượng calo cùng các chỉ số dinh dưỡng (Protein, Carbs, Fat, Fiber) từ hình ảnh bữa ăn dựa trên mô hình Machine Learning & Thị giác máy tính (Computer Vision). 

Mô hình được huấn luyện dựa trên tập dữ liệu Food-101 (gồm 101,000 hình ảnh chia đều cho 101 nhóm món ăn). Để tối ưu hóa cho bài toán theo dõi dinh dưỡng thực tế, dự án đã chọn lọc dữ liệu để chọn ra 50 nhóm món ăn có tần suất tiêu thụ cao nhất (tương ứng 50,000 ảnh). Mô hình học sâu EfficientNet-B0 được tinh chỉnh trên tập dữ liệu này, đạt độ chính xác Test Top-1 Accuracy 85.62% (Weighted F1-Score 0.86) trên 5,000 ảnh test độc lập trong điều kiện thử nghiệm. 

Mô hình được nén sang định dạng ONNX (15.5 MB) và vận hành trên kiến trúc Serverless của AWS (Amplify, API Gateway, Lambda, ECR, S3, Rekognition, CloudWatch, SNS). Giải pháp giúp đơn giản hóa quy trình ghi chép dinh dưỡng chỉ còn vài giây mỗi bữa ăn với chi phí vận hành tối ưu nhờ cơ chế tính phí theo mức độ sử dụng thực tế.

### 2. Tuyên bố vấn đề
#### Vấn đề thực tế
Việc tính toán hàm lượng calo và dinh dưỡng (Protein, Carbs, Fat) hiện nay vẫn phụ thuộc nhiều vào các ứng dụng nhập liệu thủ công (như MyFitnessPal, Yazio). Người dùng phải tự gõ tên từng món ăn, ước lượng khối lượng và tra cứu thủ công. Quy trình này trải qua nhiều bước rườm rà, dẫn đến việc bỏ dở giữa chừng sau một thời gian sử dụng.

#### Giải pháp đề xuất
NutriVision cung cấp giao diện web hiện đại được lưu trữ trực tiếp trên AWS Amplify Hosting, hỗ trợ truy cập HTTPS bảo mật từ mọi thiết bị mà không yêu cầu đăng ký tài khoản phức tạp. Người dùng chỉ cần chụp hoặc tải ảnh bữa ăn lên hệ thống. Ảnh được gửi qua API Gateway đến AWS Lambda để nhận diện món ăn nhanh chóng và trả về bảng calo chi tiết. 
- **Cơ chế AI Fallback (Amazon Rekognition)**: Khi mô hình ONNX chính có độ tin cậy dưới 60% (ví dụ do ảnh chụp góc nghiêng, ánh sáng kém, hoặc góc lạ), hệ thống tự động kích hoạt Amazon Rekognition để quét nhãn tổng quan và cố gắng ánh xạ sang một trong 50 món ăn trong bảng dữ liệu calo. Lưu ý: Cơ chế này chỉ có tác dụng khi món ăn thực ra thuộc một trong 50 món đã định nghĩa nhưng ONNX mất tự tin do chất lượng ảnh — nó không mở rộng khả năng nhận diện sang các món nằm ngoài phạm vi 50 món. Mọi ảnh có độ tin cậy thấp đều được lưu tự động về s3://.../ood_logs/ để kỹ sư AI xem xét và tái huấn luyện mô hình sau này.
- **Tính toán khẩu phần ăn linh hoạt**: Hệ thống cho phép người dùng tùy chọn hệ số khẩu phần trực tiếp trên Web UI (Nhỏ 0.7x, Vừa 1.0x, Lớn 1.5x, Đặc biệt 2.0x), từ đó Lambda tự động nhân tỷ lệ Calo và Macros theo lượng ăn thực tế.

#### Lợi ích và hiệu quả đầu tư (ROI)
- **Tối ưu thời gian**: Giảm đáng kể thời gian nhập liệu dinh dưỡng hàng ngày cho người tập thể hình, người ăn kiêng hoặc bệnh nhân cần theo dõi chỉ số bằng một thao tác tải ảnh.
- **Hiệu quả chi phí**: Kiến trúc Serverless chỉ phát sinh chi phí khi có request thực tế, không tốn chi phí duy trì máy chủ 24/7, giúp tối ưu hóa ngân sách vận hành tối đa.

### 3. Kiến trúc giải pháp
![NutriVision Architecture Diagram](/FCAJ-Project/images/2-Proposal/solution_architecture.drawio.png)

#### Các dịch vụ AWS sử dụng (8 Dịch vụ Core):
1. **AWS Amplify**: Hosting tĩnh giao diện Web (Frontend), tích hợp CI/CD với GitHub, phân phối qua CloudFront CDN & cấp chứng chỉ HTTPS SSL.
2. **Amazon API Gateway**: Quản lý REST API POST /predict, xử lý CORS và kiểm soát tần suất gọi (Rate Limiting 20 req/s chống spam).
3. **AWS Lambda**: Xử lý tính toán Serverless (Python 3.12 / ONNX Runtime / Boto3), giải mã ảnh, kiểm tra định dạng ảnh và tính calo theo khẩu phần (0.7x – 2.0x).
4. **Amazon ECR**: Lưu trữ và quản lý Docker Container Image của Lambda, đóng gói toàn bộ mô hình AI và các thư viện C++ native cần thiết.
5. **Amazon S3**: Lưu trữ tập trung tệp mô hình food_model.onnx (15.5 MB), bảng tra cứu calo calorie_map.json và log ảnh món lạ (ood_logs/).
6. **Amazon Rekognition**: Engine AI dự phòng (Fallback). Kích hoạt quét nhãn tổng quan khi mô hình chính có độ tin cậy < 60%.
7. **Amazon CloudWatch**: Giám sát P95 Latency, quản lý Log Groups và thiết lập Alarms phát hiện sự cố hệ thống.
8. **Amazon SNS**: Tự động gửi Email cảnh báo đến kỹ sư vận hành khi CloudWatch Alarm kích hoạt.

### 4. Triển khai kỹ thuật & Quy trình MLOps Thực tế
Quá trình xây dựng dự án áp dụng quy trình MLOps thực tế được chia làm 2 giai đoạn chính:

#### Giai đoạn 1: Xử lý dữ liệu, huấn luyện & đóng gói mô hình AI (Google Colab GPU)
- **Sàng lọc dữ liệu (Data Curation)**: Từ tập dữ liệu gốc Food-101 (101 món), dự án chọn lọc 50 nhóm món ăn phổ biến nhất (50,000 ảnh) dựa trên 3 tiêu chí kỹ thuật và thực tiễn:
  1. *Độ phổ biến & Tần suất tiêu thụ*: Ưu tiên các món ăn đại diện tiêu biểu của ẩm thực Á Đông (Phở, Cơm chiên, Pad Thai, Sushi, Bibimbap, Gyoza) và ẩm thực Âu - Mỹ (Hamburger, Pizza, Steak, Spaghetti, Club Sandwich, Mac & Cheese).
  2. *Cân đối đa dạng nhóm dinh dưỡng*: Phân bổ đều qua 6 nhóm chính gồm: Món giàu Protein (Steak, Sườn BBQ, Cánh gà), Cơm & Mì (Phở, Cơm chiên, Pad Thai), Fast Food (Burger, Pizza, Hot Dog), Salad ăn kiêng (Caesar, Greek, Caprese), Bữa sáng (Omelette, Pancakes, Waffles), và Tráng miệng (Chocolate Cake, Cheesecake, Apple Pie, Ice Cream).
  3. *Độ phân biệt thị giác*: Loại bỏ các món quá tương đồng nhau để mô hình đạt độ chính xác cao và nén dung lượng gọn nhẹ.
  - Tập dữ liệu 50 món được phân chia chuẩn: 37,500 ảnh Train (750 ảnh/lớp), 7,500 ảnh Validation (150 ảnh/lớp) và 5,000 ảnh Test độc lập (100 ảnh/lớp).
- **Huấn luyện & Đóng gói ONNX**: Sử dụng mạng EfficientNet-B0 với quy trình Fine-Tuning 2 giai đoạn (Freeze & Unfreeze Backbone). Kết quả đạt Test Top-1 Accuracy 85.62% trên 5,000 ảnh test trong môi trường thử nghiệm. Mô hình PyTorch (.pth) được đóng gói sang định dạng tĩnh food_model.onnx (15.5 MB) giúp tối ưu hóa dung lượng lưu trữ và tốc độ khởi chạy.

#### Giai đoạn 2: Triển khai Cloud & Giám sát Vận hành
- **Tự động hóa CI/CD cho Frontend**: Kết nối trực tiếp mã nguồn Web UI từ GitHub Repository lên AWS Amplify Hosting. Mỗi khi có thay đổi code giao diện được git push lên nhánh main, AWS Amplify sẽ tự động trigger quy trình Build & Deploy phiên bản mới nhất chỉ trong vài giây.
- **Quản lý Hạ tầng & Triển khai Backend**: Đóng gói Lambda Function dưới dạng Docker Image, đẩy lên Amazon ECR và cấu hình toàn bộ tài nguyên AWS (Lambda, API Gateway, S3, Rekognition, CloudWatch, SNS) thông qua AWS Console GUI kết hợp AWS CLI.
- **Giám sát & Xử lý trường hợp biên**:
  - Hệ thống ghi nhận toàn bộ log thời gian suy luận và nhật ký thực thi tại Amazon CloudWatch Log Groups.
  - Khi phát hiện hình ảnh nằm ngoài 50 lớp đã học (độ tin cậy < 60%), hệ thống tự động lưu hình ảnh vào s3://.../ood_logs/ để phục vụ kiểm tra sau.
  - *Hiện tại*: Việc rà soát dữ liệu OOD, gán nhãn và tái huấn luyện mô hình được thực hiện thủ công — kỹ sư AI tự truy cập S3, kiểm tra ngoại quan và chạy lại script huấn luyện trên Colab GPU.

> [!NOTE] Hướng cải tiến trong tương lai: Tích hợp pipeline tái huấn luyện tự động sử dụng Amazon SageMaker Pipelines (tự động gán nhãn qua SageMaker Ground Truth, trigger tái huấn luyện khi đủ ngưỡng dữ liệu mới, đồng bộ model mới lên S3 và cập nhật Lambda tự động) — biến quy trình MLOps hiện tại từ mức bán tự động thành toàn tự động.

### 5. Lộ trình & Mốc thực hiện
Dự án được triển khai trong thời gian 2 tháng với 4 giai đoạn cụ thể:
- **Tháng 1 (Nửa đầu - Tuần 1-2)**: Phân tích tập dữ liệu Food-101, dọn dẹp nhãn nhiễu, lọc 50 lớp món ăn phổ biến, thiết kế sơ đồ kiến trúc bằng Draw.io và lập dự toán chi phí AWS.
- **Tháng 1 (Nửa sau - Tuần 3-4)**: Huấn luyện mô hình EfficientNet-B0 trên Colab GPU, xuất mô hình sang định dạng ONNX, viết mã nguồn Lambda và kiểm thử cục bộ.
- **Tháng 2 (Nửa đầu - Tuần 5-6)**: Triển khai toàn bộ hạ tầng AWS Serverless tại vùng ap-southeast-1 (Singapore) bằng Docker Container / AWS CLI, kết nối Frontend lên AWS Amplify Hosting và đấu nối giao diện Web với API Gateway & Lambda.
- **Tháng 2 (Nửa sau - Tuần 7-8)**: Đánh giá hiệu năng hệ thống trên CloudWatch (phân tích Cold Start vs Warm Start), cấu hình SNS Email Alarms, kiểm thử toàn diện và hoàn thiện tài liệu Workshop step-by-step.

### 6. Ước tính ngân sách & Quản lý chi phí (Budget Estimation)

| Dịch vụ AWS | Chi phí ước tính |
|---|---|
| AWS Amplify Hosting | ~$0.50/tháng |
| Amazon API Gateway | ~$0.35/tháng |
| AWS Lambda | ~$0.00/tháng |
| Amazon ECR | ~$0.03/tháng |
| Amazon S3 (Storage & Requests) | ~$0.15/tháng |
| Amazon Rekognition (Fallback AI) | ~$5.00/tháng |
| Amazon CloudWatch | ~$0.02/tháng |
| Amazon SNS | ~$0.00/tháng |
| **Tổng chi phí ước tính** | **~$6.05 USD/tháng** |

> [!NOTE] 
> Chi phí tiêu chuẩn (Pay-as-you-go) là khoảng ~$6.05 USD/tháng cho quy mô 100,000 lượt nhận diện. Trong thời gian áp dụng AWS Free Tier (năm đầu), hầu hết dịch vụ hạ tầng đều được miễn phí nên chi phí thực tế chỉ từ $0.00 – $5.00 USD/tháng (chỉ phát sinh nếu gọi Rekognition vượt 5,000 lượt).

### 7. Đánh giá rủi ro & Phương án xử lý
- **Rủi ro món ăn lạ (Out-of-Distribution)**: Món ăn không nằm trong 50 món đã học ➔ *Xử lý*: Nếu confidence < 60%, tự động kích hoạt Amazon Rekognition và thử ánh xạ sang món gần nhất trong 50 món. Nếu không ánh xạ được, hệ thống chủ động trả về HTTP 422 thay vì đưa ra kết quả sai lệch, đảm bảo tính trung thực về dữ liệu dinh dưỡng. Toàn bộ ảnh OOD được lưu về s3://.../ood_logs/ để phục vụ tái huấn luyện.
- **Rủi ro Cold Start của Lambda**: Lần gọi đầu tiên sau khi container nghỉ có thể phát sinh độ trễ ➔ *Xử lý*: Tối ưu hóa mã nguồn Python siêu nhẹ giúp giảm thời gian cold start. Đối với môi trường sản xuất cần cam kết độ trễ nhất quán, có thể cân nhắc kích hoạt AWS Lambda Provisioned Concurrency.
- **Rủi ro chất lượng ảnh đầu vào**: Ảnh chụp quá mờ, quá tối hoặc file hỏng ➔ *Xử lý*: Hàm kiểm tra định dạng và chất lượng ảnh trên Lambda sẽ chặn ngay lập tức và trả về thông báo lỗi hướng dẫn người dùng tải lại ảnh hợp lệ.
- **Rủi ro vượt ngưỡng chi phí & Sự cố hệ thống**: Bị phát sinh lượng lớn request hoặc Lambda gặp lỗi ➔ *Xử lý*: Thiết lập Throttling trên API Gateway (giới hạn 20 req/s) và cài đặt CloudWatch Budget Alarm tự động kích hoạt Amazon SNS gửi Email thông báo sự cố cho quản trị viên.

### 8. Kết quả kỳ vọng
1. **Về kỹ thuật**: Xây dựng thành công hệ thống Serverless hoàn chỉnh kết hợp AI Computer Vision, hosting toàn bộ giao diện Web trên AWS Amplify Hosting, hỗ trợ mở rộng (Auto-scaling) và xử lý các trường hợp biên phổ biến. Mô hình phân loại 50 nhóm món ăn phổ biến đạt Test Top-1 Accuracy 85.62% (Weighted F1-Score 0.86) trên tập test thử nghiệm.
2. **Về kinh tế & vận hành**: Tiết kiệm 100% chi phí máy chủ nhàn rỗi so với EC2 truyền thống nhờ kiến trúc Serverless, thời gian phản hồi nhanh và chi phí vận hành chỉ khoảng vài USD/tháng cho 100,000 lượt phân tích.
3. **Về tài liệu**: Cung cấp bộ tài liệu hướng dẫn thực hành (Workshop) chi tiết từng bước, kèm hình ảnh minh họa thực tế trên AWS Console GUI, giúp người học dễ dàng tự triển khai lại giải pháp.