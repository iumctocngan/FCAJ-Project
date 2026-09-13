---
title: "Xuất mô hình tĩnh ONNX"
date: 2026-08-04
weight: 2
chapter: false
pre: " <b> 5.3. </b> "
---

#### Xuất mô hình ONNX

#### 1. Tại sao cần xuất mô hình sang định dạng ONNX?
- Tối ưu dung lượng cho Serverless: Thư viện PyTorch (torch + torchvision) có kích thước giải nén vượt quá 1.2 GB, không thể đóng gói trực tiếp lên AWS Lambda.
- Dung lượng nhỏ gọn và tốc độ cao: Thư viện ONNX Runtime chỉ chiếm 18 MB. Khi đóng gói cùng file mô hình food_model.onnx (15.52 MB), tổng dung lượng gói Lambda chỉ khoảng ~34 MB, giúp xử lý suy luận nhanh chóng và tối ưu thời gian khởi chạy.

#### 2. Thực thi Mã nguồn Export ONNX (Cell 8):

![onnx-export-success](/FCAJ-Project/images/5-Workshop/5.3-S3-vpc/onnx_export_success.png)

#### 3. Nghiệm thu & Tải tệp về máy cục bộ:
- Sau khi xuất thành công, kiểm tra file food_model.onnx đạt dung lượng chuẩn 15.52 MB.
- Tải 2 tệp food_model.onnx (15.52 MB) và calorie_map.json về máy cục bộ để sẵn sàng đóng gói Docker và lưu trữ lên Amazon S3.
