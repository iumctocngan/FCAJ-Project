---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# MLOps trên AWS SageMaker: Từ huấn luyện đến giám sát mô hình

> Huấn luyện được một mô hình có độ chính xác cao chỉ là bước đầu. Khi đưa model lên production, chúng ta còn phải biết model được train từ dữ liệu nào, bằng phiên bản code nào, model nào đang được triển khai và chất lượng model có thay đổi theo thời gian hay không. Đây chính là bài toán mà MLOps giải quyết.

Trên AWS, một quy trình MLOps có thể được xây dựng xoay quanh SageMaker Pipelines, Model Registry, cơ chế kiểm tra baseline/drift và công cụ Model Monitor.

---

### 1. MLOps Cần Kiểm Soát Những Gì?

Khác với các ứng dụng phần mềm truyền thống (nơi hành vi chủ yếu chỉ phụ thuộc vào mã nguồn), hệ thống Machine Learning phức tạp hơn vì nó liên tục chịu tác động từ ba yếu tố: mã nguồn, dữ liệu và hành vi môi trường thực tế.

Ba vấn đề chính cần kiểm soát gồm:

- Thay đổi mã nguồn và cấu hình (Code & Configuration): Các thay đổi trong khâu tiền xử lý, siêu tham số, thư viện phụ thuộc hoặc kiến trúc mô hình. Việc kết hợp Git, quản lý phiên bản và metadata lineage giúp xác định chính xác phiên bản code nào đã tạo ra từng model artifact.
- Trôi dạt dữ liệu (Data Drift): Phân phối thống kê của dữ liệu đầu vào trong môi trường production thay đổi so với phân phối dữ liệu huấn luyện dùng làm mốc tham chiếu.
- Trôi dạt khái niệm (Concept Drift): Mối quan hệ bản chất giữa đầu vào và đầu ra thay đổi theo thời gian, khiến khả năng suy luận của mô hình suy giảm dù dữ liệu đầu vào trông vẫn tương tự.

> [!IMPORTANT]
> Sự xuất hiện của drift không đồng nghĩa với việc mô hình chắc chắn đã bị hỏng. Drift đóng vai trò như một tín hiệu cảnh báo sớm để đội ngũ kỹ sư kiểm tra lại chất lượng dữ liệu và đánh giá hiệu năng thực tế trước khi đưa ra quyết định tái huấn luyện.

---

### 2. SageMaker Pipelines và Thiết Lập Baseline

Amazon SageMaker Pipelines là công cụ CI/CD chuyên biệt dành cho ML, giúp tổ chức và tự động hóa các bước trong workflow: tiền xử lý dữ liệu, huấn luyện, đánh giá và đăng ký mô hình.

Một thành phần then chốt trong pipeline là baseline. Đây là tập hợp các chỉ số thống kê và ràng buộc dữ liệu được trích xuất từ tập dữ liệu tham chiếu (thường là tập training/validation) nhằm làm thước đo chuẩn để các lần chạy sau so sánh và phát hiện biến động.

SageMaker cung cấp các bước kiểm tra chuyên biệt:
- QualityCheckStep: Tính toán baseline và kiểm tra chất lượng dữ liệu cũng như chất lượng dự đoán của mô hình.
- ClarifyCheckStep: Kiểm tra độ lệch dữ liệu và phân tích tính giải thích được của mô hình.

Hai tham số cấu hình thường dùng trong pipeline:
- skip_check: Quyết định có bỏ qua bước so sánh với baseline tham chiếu hiện tại hay không.
- register_new_baseline: Quyết định xem baseline vừa tính toán có được lưu lại làm điểm chuẩn mới cho các lần vận hành tiếp theo hay không.

*(Ở lần chạy đầu tiên khi chưa có baseline lịch sử, pipeline được cấu hình tạo baseline ban đầu. Các lần thực thi sau đó sẽ dùng baseline này làm điểm đối chiếu).*

---

### 3. SageMaker Model Registry và Quản Lý Phiên Bản Model

Sau khi hoàn tất quá trình training và evaluation, mô hình cần trải qua khâu phê duyệt và quản lý phiên bản trước khi đưa ra sử dụng thực tế.

SageMaker Model Registry cho phép quản lý tập trung các phiên bản model, siêu dữ liệu, chỉ số đánh giá và liên kết nguồn gốc. Mỗi phiên bản mô hình được gán một trong các trạng thái:

- PendingManualApproval: Trạng thái mặc định sau khi train xong, chờ người phụ trách rà soát.
- Approved: Mô hình đáp ứng đầy đủ tiêu chuẩn, sẵn sàng để deployment pipeline kích hoạt triển khai.
- Rejected: Mô hình không đạt chuẩn, bị từ chối đưa vào sử dụng.

> [!TIP]
> Theo dõi nguồn gốc (Lineage & Traceability): Baseline và báo cáo đánh giá có thể được liên kết trực tiếp vào từng model version trong Model Registry. Điều này giúp đội ngũ kỹ sư biết rõ model trên production được sinh ra từ đâu, dùng code nào, dữ liệu nào và baseline chuẩn tương ứng là gì.

---

### 4. Continuous Monitoring và Xử Lý Drift

Sau khi triển khai lên Endpoint hoặc Serverless Inference, hệ thống duy trì cơ chế giám sát liên tục qua Amazon SageMaker Model Monitor:

- Thu thập log thời gian thực: Ghi nhận payload đầu vào và kết quả suy luận lưu trữ trên Amazon S3.
- So sánh với baseline: Định kỳ chạy các job kiểm tra để phát hiện xem có sự sai lệch phân phối (Data Drift) hay không.
- Đánh giá theo nhãn thực tế: Khi có nhãn thực tế sau một khoảng thời gian vận hành, hệ thống tính lại các chỉ số hiệu năng (Accuracy, Precision, Recall, F1-Score) để phát hiện Model Drift.

Quy trình xử lý khi phát hiện Drift:
Phát hiện Drift hoặc cảnh báo ➔ Kiểm tra nguyên nhân gốc rễ (mùa vụ, lỗi pipeline, dữ liệu bất thường) ➔ Thu thập và xác thực tập dữ liệu mới ➔ Kích hoạt pipeline tái huấn luyện ➔ Đánh giá và phê duyệt trên Model Registry ➔ Triển khai phiên bản mới.

> [!CAUTION]
> Không nên tự động retrain ngay khi có cảnh báo: Khi phát hiện drift, không nên ngay lập tức tự động kích hoạt retrain và deploy mô hình mới. Drift có thể bắt nguồn từ lỗi hệ thống thu thập dữ liệu hoặc biến động mùa vụ tạm thời. Cần có bước xác thực nguyên nhân trước khi tiến hành retrain.

---

### 5. Amazon SageMaker Feature Store Có Bắt Buộc Không?

SageMaker Feature Store là kho lưu trữ trung tâm dành cho các feature kỹ thuật, cung cấp hai tầng lưu trữ:
- Online Store: Độ trễ thấp (vài mili-giây), phục vụ cho real-time inference.
- Offline Store: Lưu trữ dữ liệu lịch sử trên Amazon S3 phục vụ huấn luyện mô hình và batch scoring.

#### Khi nào nên dùng?
Feature Store phù hợp với các hệ thống phức tạp dựa nhiều vào feature engineering dạng bảng (tabular data), chẳng hạn như phát hiện gian lận, hệ thống gợi ý hoặc dự báo nhu cầu.

#### Lưu ý thực tế:
Feature Store không tự động giải quyết bài toán sai lệch giữa huấn luyện và phục vụ (Training-Serving Skew); kỹ sư vẫn phải thiết kế logic tính toán feature nhất quán. Ngoài ra, giải pháp này không bắt buộc cho mọi bài toán: đối với bài toán thị giác máy tính hay xử lý ngôn ngữ tự nhiên lưu trữ trực tiếp trên Amazon S3, việc đưa thêm Feature Store vào kiến trúc có thể làm tăng chi phí và độ phức tạp không cần thiết.

---

### 6. Toàn Bộ Vòng Đời MLOps

Có thể tóm tắt toàn bộ luồng vận hành MLOps qua các bước:

Dữ liệu ➔ Tiền xử lý ➔ Huấn luyện ➔ Đánh giá & Kiểm tra Baseline/Drift ➔ Đăng ký Model Registry ➔ Phê duyệt ➔ Triển khai ➔ Giám sát vận hành ➔ Thu thập dữ liệu mới ➔ Tái huấn luyện.

Như vậy, MLOps không chỉ đơn thuần là tự động hóa khâu huấn luyện và triển khai. Mục tiêu của MLOps là quản lý toàn diện vòng đời của mô hình: từ dữ liệu, mã nguồn, phiên bản, đánh giá, phê duyệt, triển khai cho đến giám sát liên tục và tiếp nhận phản hồi.

---

### 7. Kết Luận

Một hệ thống MLOps hoàn chỉnh cần giải quyết được ba câu hỏi:
1. Mô hình đang chạy trên production đến từ đâu (nguồn gốc dữ liệu, phiên bản code)?
2. Tại sao mô hình đó được phép triển khai (ai duyệt, tiêu chuẩn đánh giá nào)?
3. Mô hình có còn hoạt động tốt và chính xác sau khi triển khai hay không?

AWS SageMaker cung cấp đầy đủ các thành phần để hỗ trợ các yêu cầu này. Trong thực tế triển khai, chúng ta có thể bắt đầu với một pipeline cơ bản, sau đó bổ sung dần Model Registry, Drift Detection hoặc Feature Store khi hệ thống mở rộng và có nhu cầu cụ thể.

---

### Nguồn Tham Khảo

1. [Amazon SageMaker AI — Pipelines Overview](https://docs.aws.amazon.com/sagemaker/latest/dg/pipelines-overview.html)
2. [Amazon SageMaker AI — Baseline & Drift Detection](https://docs.aws.amazon.com/sagemaker/latest/dg/pipelines-quality-clarify-baseline-lifecycle.html)
3. [Amazon SageMaker AI — Model Registry](https://docs.aws.amazon.com/sagemaker/latest/dg/model-registry.html)
4. [Amazon SageMaker AI — Feature Store Concepts](https://docs.aws.amazon.com/sagemaker/latest/dg/feature-store-concepts.html)

