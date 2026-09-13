---
title: "Worklog Tuần 8"
date: 2024-01-01
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8:
- Khảo sát bài toán, xác định yêu cầu MVP và thiết kế kiến trúc AWS Serverless cho dự án NutriVision.
- Chuẩn bị dữ liệu Food-101, fine-tune mô hình EfficientNet-B0 và xuất mô hình sang ONNX.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành |
| --- | --- | --- | --- |
| 2 | - Khảo sát bài toán theo dõi dinh dưỡng, xác định đối tượng người dùng và phạm vi MVP NutriVision | 31/08/2026 | 31/08/2026 |
| 3 | - Thiết kế kiến trúc tổng thể AWS Serverless và luồng dữ liệu giữa Web UI, API Gateway, Lambda và AI model | 01/09/2026 | 01/09/2026 |
| 4 | - Thu thập, chọn lọc 50 lớp món ăn từ Food-101, chia tập train/val/test và xây dựng pipeline tiền xử lý ảnh | 02/09/2026 | 02/09/2026 |
| 5 | - Fine-tune EfficientNet-B0 trên Google Colab GPU và đánh giá mô hình (Accuracy, F1-Score) | 03/09/2026 | 03/09/2026 |
| 6 | - Xuất mô hình sang định dạng ONNX (15.5 MB), kiểm thử suy luận cục bộ và xây dựng calorie_map.json | 04/09/2026 | 04/09/2026 |

### Kết quả đạt được tuần 8:
- Hoàn thành tài liệu thiết kế kiến trúc Serverless và kế hoạch triển khai 3 tuần cho dự án NutriVision.
- Xây dựng thành công tập dữ liệu 50 lớp món ăn và pipeline tiền xử lý ảnh chuẩn hóa.
- Fine-tune EfficientNet-B0 đạt Test Top-1 Accuracy 85.62% và Weighted F1-Score 0.86.
- Xuất thành công mô hình ONNX dung lượng tối ưu (15.5 MB) và hoàn thiện tệp tra cứu dinh dưỡng calorie_map.json.
