---
title: "Worklog Tuần 10"
date: 2024-01-01
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---


### Mục tiêu tuần 10:

- Chuẩn bị dữ liệu và xây dựng mô hình nhận diện món ăn cho NutriVision.
- Đánh giá mô hình, tối ưu kích thước và chuẩn bị artefact để tích hợp vào backend.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành |
| --- | --- | --- | --- |
| 2 | - Thu thập, khảo sát và chọn 50 lớp món ăn phổ biến từ bộ dữ liệu Food-101 | 14/09/2026 | 14/09/2026 |
| 3 | - Làm sạch dữ liệu, chia tập train/validation/test và xây dựng pipeline tiền xử lý ảnh | 15/09/2026 | 15/09/2026 |
| 4 | - Fine-tune EfficientNet-B0 trên Google Colab GPU và theo dõi quá trình huấn luyện | 16/09/2026 | 16/09/2026 |
| 5 | - Đánh giá Accuracy, Precision, Recall, F1-score; phân tích confusion matrix và lỗi dự đoán | 17/09/2026 | 17/09/2026 |
| 6 | - Xuất mô hình sang ONNX, kiểm thử suy luận cục bộ và hoàn thiện calorie_map.json | 18/09/2026 | 18/09/2026 |

### Kết quả đạt được tuần 10:

- Hoàn thành tập dữ liệu 50 lớp món ăn và pipeline tiền xử lý dùng thống nhất khi huấn luyện, kiểm thử.
- Fine-tune thành công EfficientNet-B0 và đánh giá mô hình trên tập test độc lập.
- Xuất mô hình ONNX dung lượng 15.5 MB, đạt Test Top-1 Accuracy 85.62% và Weighted F1-Score 0.86.
- Hoàn thiện tệp ánh xạ dinh dưỡng calorie_map.json và xác nhận kết quả suy luận cục bộ sẵn sàng tích hợp.
