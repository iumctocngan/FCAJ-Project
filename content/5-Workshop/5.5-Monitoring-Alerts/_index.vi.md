---
title: "Bước 3: Giám sát CloudWatch & Cảnh báo SNS"
date: 2026-08-04
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

#### Cấu hình Amazon CloudWatch & Cảnh báo Amazon SNS

Trong phần này, bạn sẽ cấu hình Amazon CloudWatch để giám sát backend Serverless chạy trên AWS Lambda, đồng thời tích hợp Amazon SNS để tự động gửi thông báo cảnh báo thời gian thực khi phát sinh sự cố.

Amazon CloudWatch cung cấp khả năng giám sát toàn diện bằng cách thu thập số liệu (Metrics) và nhật ký thực thi (Logs) từ các tài nguyên AWS, sau đó đánh giá theo các ngưỡng do người dùng thiết lập. Điều này giúp quản trị viên phát hiện lỗi thực thi, theo dõi độ trễ suy luận AI và phản ứng kịp thời khi có sự cố xảy ra.

Trong dự án này, CloudWatch được sử dụng để giám sát lỗi thực thi (Errors) của hàm Lambda NutriVisionPredictor thông qua việc tạo CloudWatch Alarm kết hợp kênh thông báo Amazon SNS.

---

#### 1. Cấu hình & Thu thập Nhật ký (CloudWatch Logging)

AWS Lambda tự động tích hợp với Amazon CloudWatch để ghi lại toàn bộ nhật ký hoạt động vào Log Group /aws/lambda/NutriVisionPredictor.

Kiểm tra nhật ký thực thi gần nhất của hàm Lambda qua AWS CLI (lọc trong 7 ngày qua):
```bash
aws logs tail /aws/lambda/NutriVisionPredictor --region ap-southeast-1 --since 7d
```

Đoạn log mẫu thực tế từ CloudWatch ghi nhận quá trình nạp mô hình, độ trễ và nhãn AI Rekognition Fallback:
```text
[INFO] Using bundled food_model.onnx file.
[INFO] Loading calorie_map.json from local bundle.
[INFO] ONNX model initialized with 50 classes.
[INFO] Fallback labels: [('Food', 100.0), ('Meal', 100.0), ('Dish', 99.8), ('Cooking', 97.5), ('Bowl', 95.4), ('Soup', 55.3), ('Noodle', 52.8)]
REPORT RequestId: fe178352-12fe-4fdd-b73f-10ab4dfab1fb	Duration: 1359.21 ms	Billed Duration: 2482 ms	Memory Size: 512 MB	Max Memory Used: 233 MB	Init Duration: 1108.15 ms
```

Các thông số telemetry chính được ghi nhận:
- **Duration**: Thời gian thực thi suy luận AI thực tế (mili-giây).
- **Billed Duration**: Thời gian bị tính cước của Lambda.
- **Memory Size & Max Memory Used**: Dung lượng RAM được cấp phát (512 MB) và dung lượng RAM thực tế tiêu thụ.
- **Application Logs**: Nhật ký suy luận mô hình ONNX, thông tin kích hoạt Fallback của Amazon Rekognition và lưu vết ảnh Out-of-Distribution (OOD).

---

#### 2. Cấu hình Kênh Thông báo Amazon SNS (Notification Topic)

Để gửi thông báo tức thì đến quản trị viên khi xảy ra sự cố, thiết lập một SNS Topic và đăng ký địa chỉ Email nhận cảnh báo.

Các thông số cấu hình của Amazon SNS Topic:

| Thuộc tính (Property) | Giá trị cấu hình (Value) |
|---|---|
| Topic type | Standard |
| Name | NutriVision-AlarmNotifications |
| Display name | NutriVision Alerts |
| Subscription Protocol | Email |
| Endpoint | chuasheef3@gmail.com |

Thực thi lệnh khởi tạo Topic và đăng ký Email qua AWS CLI:
```bash
aws sns create-topic --name NutriVision-AlarmNotifications --attributes DisplayName="NutriVision Alerts" --region ap-southeast-1

aws sns subscribe --topic-arn arn:aws:sns:ap-southeast-1:110359221458:NutriVision-AlarmNotifications --protocol email --notification-endpoint chuasheef3@gmail.com --region ap-southeast-1
```

> [!IMPORTANT]
> Sau khi chạy lệnh đăng ký, mở hòm thư Email cá nhân và nhấp vào liên kết Confirm subscription trong thư gửi từ AWS Notifications để kích hoạt nhận thông báo.

![Email Thông báo từ Amazon SNS](/FCAJ-Project/images/5-Workshop/5.5-Monitoring-Alerts/sns_email_notification_confirmed.png)

---

#### 3. Tạo Amazon CloudWatch Alarm

Để liên tục giám sát sức khỏe của hệ thống backend, một CloudWatch Alarm được thiết lập cho hàm Lambda NutriVisionPredictor.

Cảnh báo đánh giá tổng số lỗi phát sinh (Sum of Errors) trong mỗi chu kỳ 1 phút. Nếu số lỗi đạt hoặc vượt quá ngưỡng quy định (Errors >= 1), CloudWatch sẽ tự động chuyển trạng thái cảnh báo từ OK sang In alarm và kích hoạt SNS Topic để gửi Email cảnh báo cho quản trị viên.

Bảng thông số cấu hình của CloudWatch Alarm:

| Thuộc tính (Property) | Giá trị cấu hình (Value) |
|---|---|
| Alarm name | NutriVision-HighLambdaErrors |
| Alarm description | Cảnh báo khi Lambda NutriVisionPredictor phát sinh lỗi thực thi |
| Namespace | AWS/Lambda |
| Metric name | Errors |
| Statistic | Sum |
| Period | 1 minute |
| Threshold | >= 1 |
| Function name | NutriVisionPredictor |
| Alarm Action | Gửi thông báo đến NutriVision-AlarmNotifications (SNS) |

Khởi tạo CloudWatch Alarm bằng lệnh AWS CLI:
```bash
aws cloudwatch put-metric-alarm --alarm-name NutriVision-HighLambdaErrors --alarm-description "Canh bao khi Lambda NutriVisionPredictor phat sinh loi" --metric-name Errors --namespace AWS/Lambda --statistic Sum --dimensions Name=FunctionName,Value=NutriVisionPredictor --period 60 --evaluation-periods 1 --threshold 1 --comparison-operator GreaterThanOrEqualToThreshold --treat-missing-data notBreaching --alarm-actions arn:aws:sns:ap-southeast-1:110359221458:NutriVision-AlarmNotifications --region ap-southeast-1
```

---

#### 4. Xác minh Trạng thái Cảnh báo (Verify Alarm Status)

Sau khi cảnh báo được khởi tạo, Amazon CloudWatch sẽ liên tục theo dõi tình trạng hoạt động của hàm Lambda NutriVisionPredictor.

![Trạng thái CloudWatch Alarm OK](/FCAJ-Project/images/5-Workshop/5.5-Monitoring-Alerts/cloudwatch_alarm_ok_status.png)

- **Trạng thái bình thường (OK)**: Cảnh báo duy trì ở trạng thái OK màu xanh khi hàm xử lý thành công và không phát sinh ngoại lệ chưa xử lý (Errors < 1).
- **Trạng thái cảnh báo (In alarm)**: Nếu phát sinh lỗi thực thi (Errors >= 1), CloudWatch tự động chuyển sang trạng thái In alarm màu đỏ và gửi Email cảnh báo thời gian thực.

Trang chi tiết của Alarm trên Console cũng cung cấp đầy đủ các thông tin vận hành:
- Trạng thái hiện tại của Alarm (OK / In alarm).
- Biểu đồ biểu diễn số lượng lỗi theo thời gian (Errors metric graph).
- Cấu hình ngưỡng cảnh báo và chu kỳ đánh giá (Period).
- Tên hàm Lambda và Namespace đang giám sát.
- Lịch sử kích hoạt thông báo gửi qua Amazon SNS.

Thông tin này giúp quản trị viên nắm bắt nhanh chóng tình trạng hoạt động của ứng dụng và khoanh vùng sự cố kịp thời.
