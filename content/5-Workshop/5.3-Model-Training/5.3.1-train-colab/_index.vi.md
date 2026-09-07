---
title: "Huấn luyện mô hình trên Colab"
date: 2026-08-04
weight: 1
chapter: false
pre: " <b> 5.3.1 </b> "
---

#### Các bước Huấn luyện AI trên Google Colab GPU (Stage 1)

1. **Mở Notebook & Kiểm tra Môi trường (Cell 1):**
   - Mở file notebook [Food101_FineTuning_Colab.ipynb](https://colab.research.google.com/drive/1sLn-Jui3FGhMCUTgPHQDb6mu1PsESRzX?usp=sharing) trên Google Colab.
   - Thực thi Cell 1 để kiểm tra thông số GPU T4 và cài đặt các thư viện cần thiết.

2. **Tải Dataset & Tạo Calorie Map (Cell 2 & 3):**
   - Thực thi Cell 2 để tải dữ liệu Food-101 và trích xuất 50 nhóm món ăn phổ biến nhất (50,000 ảnh: Train 37,500 | Val 7,500 | Test 5,000).
   - Thực thi Cell 3 để tự động sinh tệp tra cứu dinh dưỡng calorie_map.json.

![colab-training](/FCAJ-Project/images/5-Workshop/5.3-S3-vpc/dataset.png)

3. **Cấu hình & Fine-Tuning EfficientNet-B0 (Cell 4, 5 & 6):**
   - Cấu hình đầu vào 288x288 + TrivialAugmentWide tăng cường dữ liệu.
   - Giai đoạn 1 (Warmup): Freeze Backbone 3 epochs (Lr = 1e-3).
   - Giai đoạn 2 (Full Unfreeze): Huấn luyện toàn bộ mô hình 7 epochs với Discriminative LR (Backbone 1e-4, Classifier 1e-3) và Cosine Annealing.

![colab-training](/FCAJ-Project/images/5-Workshop/5.3-S3-vpc/colab_training.png)

4. **Nghiệm thu kết quả (Cell 7):**
   - Kết quả đánh giá trên 5,000 ảnh tập Test: Đạt **Test Top-1 Accuracy 85.62%** và **F1-score 0.86**.
   - Checkpoint được lưu tự động thành tệp best_food_model.pth (15.8 MB).

![colab-training](/FCAJ-Project/images/5-Workshop/5.3-S3-vpc/accuracy.png)
![colab-training](/FCAJ-Project/images/5-Workshop/5.3-S3-vpc/f1.png)