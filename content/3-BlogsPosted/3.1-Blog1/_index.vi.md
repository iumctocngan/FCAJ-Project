---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# MLOps trên AWS SageMaker: Từ huấn luyện đến giám sát mô hình

> Huấn luyện được một mô hình có độ chính xác cao chỉ là bước đầu. Khi đưa model lên production, chúng ta còn phải biết model được train từ dữ liệu nào, bằng phiên bản code nào, model nào đang được triển khai và chất lượng model có thay đổi theo thời gian hay không. Đây chính là bài toán mà **MLOps** giải quyết.

Trên **AWS**, một quy trình MLOps hoàn chỉnh có thể được xây dựng xoay quanh **SageMaker Pipelines**, **Model Registry**, **Baseline/Drift checking** và các công cụ **Model Monitoring**.

---

### 1. MLOps Cần Kiểm Soát Những Gì?

Khác với các ứng dụng phần mềm truyền thống (nơi hành vi chủ yếu chỉ phụ thuộc vào source code), hệ thống **Machine Learning (ML)** phức tạp hơn nhiều vì nó liên tục thay đổi ở ba yếu tố: **mã nguồn**, **dữ liệu** và **hành vi thực tế**.

Có ba vấn đề cốt lõi cần phải kiểm soát chặt chẽ:

1. **Code & Configuration Change:** Các thay đổi trong khâu tiền xử lý (preprocessing), siêu tham số (hyperparameters), thư viện phụ thuộc (dependencies) hoặc kiến trúc mô hình. Việc kết hợp Git, versioning và metadata lineage giúp xác định chính xác phiên bản code nào đã tạo ra từng model artifact.
2. **Data Drift (Trôi dạt dữ liệu):** Phân phối thống kê của dữ liệu đầu vào trong môi trường production thay đổi so với phân phối dữ liệu huấn luyện dùng làm mốc tham chiếu.
3. **Concept Drift (Trôi dạt khái niệm):** Mối quan hệ bản chất giữa đầu vào (input) và đầu ra (output target) thay đổi theo thời gian, khiến cho dù dữ liệu đầu vào không đổi nhưng khả năng suy luận của mô hình vẫn bị suy giảm nghiêm trọng.

> [!IMPORTANT]
> **Lưu ý quan trọng:** Sự xuất hiện của *drift* không đồng nghĩa với việc mô hình chắc chắn đã bị hỏng. Drift là một **tín hiệu cảnh báo sớm** để đội ngũ kỹ sư kiểm tra lại chất lượng dữ liệu và đánh giá hiệu năng thực tế trước khi đưa ra quyết định tái huấn luyện (retrain).

---

### 2. SageMaker Pipelines và Thiết Lập Baseline

**Amazon SageMaker Pipelines** là công cụ CI/CD chuyên biệt dành cho ML, giúp tự động hóa và tổ chức các bước trong workflow: tiền xử lý dữ liệu (Data Preprocessing), huấn luyện (Training), đánh giá (Evaluation) và đăng ký mô hình (Model Registration).

Một thành phần then chốt trong pipeline là **Baseline**. Đây là tập hợp các chỉ số thống kê (statistics) và ràng buộc dữ liệu (constraints) được trích xuất từ tập dữ liệu tham chiếu (thường là tập training/validation) nhằm làm thước đo chuẩn để các lần chạy sau so sánh và phát hiện biến động.

SageMaker cung cấp các bước chuyên biệt như:
1. **`QualityCheckStep`**: Tính toán baseline và kiểm tra chất lượng dữ liệu (Data Quality) cũng như chất lượng dự đoán của mô hình (Model Quality).
2. **`ClarifyCheckStep`**: Kiểm tra độ lệch/thiên vị dữ liệu (Data Bias) và phân tích tính giải thích được của mô hình (Explainability).

Hai tham số cấu hình quan trọng trong pipeline:
1. **`skip_check`**: Quyết định có bỏ qua bước so sánh với baseline tham chiếu hiện tại hay không.
2. **`register_new_baseline`**: Quyết định xem baseline vừa tính toán có được lưu lại làm điểm chuẩn mới cho các lần vận hành tiếp theo hay không.

*(Ở lần chạy đầu tiên khi chưa có baseline lịch sử, pipeline được cấu hình tạo baseline ban đầu. Các lần thực thi sau đó sẽ dùng baseline này làm điểm đối chiếu).*

---

### 3. SageMaker Model Registry và Quản Lý Phiên Bản Model

Sau khi hoàn tất quá trình training và evaluation, mô hình không nên được triển khai thẳng lên production mà cần trải qua khâu phê duyệt và quản lý phiên bản nghiêm ngặt.

**SageMaker Model Registry** cho phép quản lý tập trung các phiên bản model, siêu dữ liệu (metadata), chỉ số đánh giá (metrics) và liên kết nguồn gốc (lineage). Mỗi phiên bản mô hình được gán một trong các trạng thái phê duyệt:

1. `PendingManualApproval`: Trạng thái mặc định sau khi train xong, chờ chuyên gia ML hoặc QA rà soát.
2. `Approved`: Mô hình đáp ứng đầy đủ tiêu chuẩn nghiệp vụ và kỹ thuật, sẵn sàng để deployment pipeline tự động kích hoạt triển khai.
3. `Rejected`: Mô hình không đạt chuẩn, bị từ chối đưa vào sử dụng.

> [!TIP]
> **Lineage & Traceability:** Baseline và báo cáo đánh giá có thể được liên kết trực tiếp vào từng model version trong Model Registry. Điều này giúp đội ngũ kỹ sư luôn trả lời được chính xác: *Model trên production được sinh ra từ đâu, dùng code nào, dữ liệu nào và baseline chuẩn tương ứng là gì.*

---

### 4. Continuous Monitoring và Xử Lý Drift

Sau khi triển khai lên Endpoint hoặc Serverless Inference, hệ thống cần duy trì cơ chế giám sát liên tục (**Amazon SageMaker Model Monitor**):

1. **Thu thập Real-time Logs:** Ghi nhận toàn bộ payload đầu vào và kết quả suy luận (Inference input/output) lưu trữ an toàn trên Amazon S3.
2. **So sánh với Baseline:** Định kỳ chạy các job kiểm tra để phát hiện xem có sự sai lệch phân phối (Data Drift) hay không.
3. **Đánh giá Ground-Truth:** Khi có nhãn thực tế sau một khoảng thời gian vận hành, hệ thống tự động tính lại các chỉ số hiệu năng (*Accuracy, Precision, Recall, F1-Score*) để phát hiện Model Drift.

**Quy trình xử lý chuẩn khi phát hiện Drift:**
```
Phát hiện Drift / Cảnh báo
       │
       ▼
Kiểm tra nguyên nhân gốc rễ (Mùa vụ, lỗi pipeline, thay đổi hành vi...)
       │
       ▼
Thu thập & Xác thực tập dữ liệu mới
       │
       ▼
Tái huấn luyện (Retrain Pipeline)
       │
       ▼
Đánh giá & Phê duyệt (Model Registry Approval)
       │
       ▼
Triển khai phiên bản mới (Safe Deployment / Blue-Green)
```

> [!CAUTION]
> **Không nên tự động retrain mù quáng:** Khi phát hiện drift, không nên ngay lập tức tự động kích hoạt retrain và deploy mô hình mới. Drift có thể bắt nguồn từ lỗi hệ thống ingest dữ liệu, biến động mùa vụ tạm thời (Black Friday, Lễ Tết). Cần có bước xác thực nguyên nhân trước khi tiến hành retrain.

---

### 5. Amazon SageMaker Feature Store Có Bắt Buộc Không?

**SageMaker Feature Store** là kho lưu trữ trung tâm dành cho các feature kỹ thuật (feature engineering), cung cấp hai tầng lưu trữ:
1. **Online Store:** Độ trễ cực thấp (vài mili-giây), phục vụ cho real-time inference.
2. **Offline Store:** Lưu trữ dữ liệu lịch sử trên Amazon S3 phục vụ huấn luyện mô hình và batch scoring.

#### Khi nào nên dùng?
Feature Store rất hữu ích với các hệ thống phức tạp dựa nhiều vào feature engineering dạng bảng (tabular data), chẳng hạn như: *Phát hiện gian lận (Fraud Detection)*, *Hệ thống gợi ý (Recommendation System)*, *Dự báo nhu cầu (Demand Forecasting)*.

#### Lưu ý thực tế:
Feature Store không tự động giải quyết bài toán Training-Serving Skew; kỹ sư vẫn phải thiết kế logic tính toán feature nhất quán giữa quá trình training và serving. Ngoài ra, giải pháp này không bắt buộc cho mọi bài toán: đối với các bài toán thị giác máy tính hay xử lý ngôn ngữ tự nhiên lưu trữ trực tiếp trên Amazon S3, việc đưa thêm Feature Store vào kiến trúc có thể làm tăng chi phí và độ phức tạp không cần thiết.

---

### 6. Toàn Bộ Vòng Đời MLOps (End-to-End MLOps Lifecycle)

Có thể tóm tắt toàn bộ kiến trúc bằng một luồng duy nhất:

```
Data → Processing → Training → Evaluation → Quality/Drift Check
  ↓
Model Registry
  ↓
Approval
  ↓
Deployment
  ↓
Monitoring
  ↓
Collect New Data
  ↓
Retraining
```

Như vậy, **MLOps không chỉ đơn thuần là tự động hóa Train → Deploy**. Mục tiêu cốt lõi của MLOps là **quản lý toàn diện vòng đời của mô hình**: từ dữ liệu, mã nguồn, phiên bản, đánh giá, phê duyệt, triển khai cho đến giám sát liên tục và phản hồi.

---

### 7. Kết Luận

Một hệ thống MLOps đạt chuẩn cần trả lời chính xác **ba câu hỏi then chốt**:
1. *Mô hình đang chạy trên production đến từ đâu (nguồn gốc dữ liệu, phiên bản code)?*
2. *Tại sao mô hình đó được phép triển khai (ai duyệt, tiêu chuẩn đánh giá nào)?*
3. *Mô hình có còn hoạt động tốt và chính xác sau khi triển khai hay không?*

AWS SageMaker cung cấp một hệ sinh thái toàn diện để giải quyết trọn vẹn các yêu cầu này. Hãy bắt đầu với một pipeline tinh gọn, đáp ứng đúng nhu cầu hiện tại, sau đó từng bước tích hợp Model Registry, Drift Detection, Feature Store khi quy mô hệ thống đòi hỏi.

---

### Nguồn Tham Khảo

1. [Amazon SageMaker AI — Pipelines Overview](https://docs.aws.amazon.com/sagemaker/latest/dg/pipelines-overview.html)
2. [Amazon SageMaker AI — Baseline & Drift Detection](https://docs.aws.amazon.com/sagemaker/latest/dg/pipelines-quality-clarify-baseline-lifecycle.html)
3. [Amazon SageMaker AI — Model Registry](https://docs.aws.amazon.com/sagemaker/latest/dg/model-registry.html)
4. [Amazon SageMaker AI — Feature Store Concepts](https://docs.aws.amazon.com/sagemaker/latest/dg/feature-store-concepts.html)

