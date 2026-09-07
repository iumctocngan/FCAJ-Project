---
title: "Blog 2"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Amazon Bedrock: Kiến trúc nền tảng Generative AI managed và những đánh đổi cần hiểu

> Nhiều người khi mới tiếp cận **Amazon Bedrock** thường hiểu đơn giản đây là dịch vụ để gọi các foundation model trên AWS. Cách hiểu này đúng nhưng chưa đủ.

Bedrock cung cấp nhiều lớp khả năng toàn diện cho ứng dụng Generative AI: từ **Model Inference**, **RAG với Knowledge Bases**, **Guardrails** cho đến các công cụ xây dựng hệ thống **Agentic (AgentCore)**. Hiểu đúng vai trò và đánh đổi của từng lớp giúp kỹ sư xây dựng kiến trúc tối ưu, tránh over-engineering hoặc bị phụ thuộc (vendor lock-in) vào một model/provider không cần thiết.

---

### 1. Amazon Bedrock Là Gì?

**Amazon Bedrock** là dịch vụ *fully managed* cho phép khai thác các Foundation Model (FM) hàng đầu (như Anthropic Claude, Meta Llama, Amazon Titan, Mistral, Cohere, AI21 Labs...) thông qua một giao diện API duy nhất mà **không cần tự quản lý hạ tầng GPU hay cụm máy chủ**.

Bedrock không đơn thuần là nơi "huấn luyện LLM từ đầu" (pre-training). Thay vào đó, nó tập trung vào việc cung cấp **Model Inference** tốc độ cao và các năng lực mở rộng (capabilities) phục vụ việc phát triển ứng dụng GenAI thực tế: RAG, Model Customization (Fine-tuning, Continued Pre-training) và Guardrails an toàn.

#### Các API Cốt Lõi Cần Nắm:
1. **`Converse API`**: Giao diện Bedrock-native thống nhất, chuẩn hóa định dạng tin nhắn cho tất cả các mô hình hỗ trợ hội thoại (multi-turn conversation) và Tool Use (Function Calling). Đây là lựa chọn tối ưu để dễ dàng hoán đổi mô hình (model switching) mà không cần sửa đổi mã nguồn xử lý prompt.
2. **`InvokeModel` / `InvokeModelWithResponseStream`**: Gọi mô hình trực tiếp theo định dạng payload riêng của từng nhà cung cấp, phù hợp khi cần tận dụng các tính năng đặc thù hoặc tối ưu hóa luồng streaming.
3. **`Responses API` / `Chat Completions API`**: Các API tương thích với chuẩn OpenAI SDK thông qua `bedrock-mantle`, rất thuận tiện khi cần migrate các ứng dụng viết sẵn bằng thư viện OpenAI sang hạ tầng bảo mật của AWS.

> [!TIP]
> **Nguyên tắc thiết kế:** Không nên hard-code ứng dụng vào API riêng của một provider cụ thể. Hãy ưu tiên sử dụng `Converse API` hoặc các open-standard API để giữ cho hệ thống có tính linh hoạt cao nhất khi đổi mô hình.

---

### 2. RAG Với Knowledge Bases for Amazon Bedrock

**Retrieval Augmented Generation (RAG)** là phương pháp bổ sung dữ liệu nội bộ doanh nghiệp vào ngữ cảnh (context window) của mô hình để sinh phản hồi chính xác mà **không cần tốn kém chi phí retrain foundation model**.

**Knowledge Bases for Amazon Bedrock** cung cấp một luồng làm việc RAG được quản lý toàn diện (*Managed RAG Workflow*): tự động hóa các bước Ingest dữ liệu từ S3, chia nhỏ văn bản (Chunking), trích xuất Vector Embeddings và lưu trữ vào Vector Store (như Amazon OpenSearch Serverless, Pinecone, Amazon Aurora PostgreSQL pgvector).

Khi người dùng đặt câu hỏi, hệ thống tìm các đoạn tài liệu liên quan và cung cấp chúng làm context cho model.

#### Hai Phương Thức Sử Dụng Chính & Đánh Đổi:

| Tiêu chí | `RetrieveAndGenerate` | `Retrieve` (Chỉ trích xuất) |
| :--- | :--- | :--- |
| **Cơ chế hoạt động** | Bedrock tự động thực hiện cả 2 bước: tìm kiếm văn bản liên quan và đưa vào prompt để model sinh câu trả lời. | Chỉ thực hiện vector/hybrid search và trả về danh sách các đoạn văn bản (chunks + metadata) liên quan nhất. |
| **Ưu điểm** | Tốc độ triển khai cực nhanh, viết rất ít code, luồng xử lý trọn gói. | Toàn quyền kiểm soát logic xử lý sau retrieval (Post-processing). |
| **Khả năng tùy biến** | Hạn chế tùy biến prompt và logic trung gian. | Linh hoạt áp dụng: Re-ranking (Reranker models), Document-level authorization, Custom Prompting và Citation mapping. |
| **Phù hợp cho** | Ứng dụng hỏi đáp nội bộ đơn giản, PoC nhanh chóng. | Hệ thống Enterprise có quy định phân quyền phức tạp hoặc yêu cầu độ chính xác cao. |

---

### 3. Guardrails: An Toàn Nội Dung Không Thay Thế Authorization

**Guardrails for Amazon Bedrock** cho phép tách biệt chính sách an toàn nội dung (Safety Policies) ra khỏi system prompt và logic mô hình, giúp kiểm soát rủi ro tập trung:

1. **Bộ lọc chủ đề bị cấm (Denied Topics):** Ngăn chặn mô hình tư vấn các chủ đề ngoài phạm vi (ví dụ: cấm bot ngân hàng đưa lời khuyên đầu tư tiền điện tử).
2. **Bộ lọc nội dung độc hại (Content Filters):** Chặn ngôn từ thù địch, quấy rối, bạo lực, nội dung người lớn.
3. **Mặt nạ dữ liệu nhạy cảm (Sensitive Information Filters):** Tự động phát hiện và che giấu thông tin định danh cá nhân (**PII**) như CCCD, số thẻ tín dụng, email.
4. **Bộ lọc từ ngữ tùy chỉnh (Custom Words/Profanity):** Chặn danh sách từ khóa nội bộ không mong muốn.

> [!CAUTION]
> **Phân biệt cốt lõi: Guardrails ≠ Authorization (Phân quyền truy cập tài liệu)**
> 
> Guardrails chỉ đánh giá tính an toàn của nội dung *văn bản* (Input Prompt và Generated Output). Guardrails **không phải là hệ thống kiểm soát quyền truy cập tài liệu (Access Control)**.

Nếu Knowledge Base chứa nhiều tài liệu với các cấp độ bảo mật khác nhau (*Tài liệu nhân viên, dữ liệu quản lý, báo cáo tài chính nội bộ*), hệ thống bắt buộc phải thiết kế tầng **Identity & Authorization** riêng:
1. Xác thực danh tính người dùng (IAM / Cognito / OAuth).
2. Áp dụng **Metadata Filtering** trong lệnh gọi `Retrieve` để chỉ tìm kiếm trong phạm vi tài liệu mà người dùng đó được phép xem.
3. Sử dụng IAM Roles và Session Policies để cô lập dữ liệu.

*Nói cách khác, Guardrails bảo vệ nội dung, còn authorization quyết định người dùng được phép truy cập dữ liệu nào.*

---

### 4. Từ Model Inference Đến Agentic Systems (AgentCore)

Khi xây dựng ứng dụng Generative AI, có thể hình dung kiến trúc Bedrock theo các lớp phát triển dần:

```
Foundation Model → RAG / Knowledge Bases → Guardrails → Agentic Application / AgentCore
```

1. **Mức Inference & RAG:** Phù hợp khi ứng dụng chủ yếu dừng lại ở việc hỏi đáp thông tin, tra cứu văn bản và tương tác hội thoại có ngữ cảnh hoặc tool use ở mức cơ bản.
2. **Mức Agentic Systems (AgentCore):** Cần thiết khi hệ thống phát triển thành một AI Agent cần nhiều capability production hơn bao gồm: Runtime & Memory (vòng lặp suy luận và quản lý bộ nhớ dài hạn/ngắn hạn), Gateway & Tool Integration (tích hợp với APIs và AWS Lambda an toàn), Code Interpreter & Sandbox (môi trường thực thi code cô lập), Browser Automation (khả năng duyệt web và tương tác UI), cùng Identity & Observability (quản lý định danh và giám sát toàn diện).

*Không phải ứng dụng nào cũng cần đi đến lớp cuối cùng. Một chatbot hỏi đáp tài liệu đơn giản có thể chỉ cần Model + Knowledge Base + lớp kiểm soát phù hợp.*

---

### 5. Những Nguyên Tắc Thiết Kế Quan Trọng & Đánh Đổi

1. **Không phụ thuộc provider nếu không cần thiết:**
   Nên lựa chọn API giúp việc thử nghiệm hoặc thay đổi model dễ dàng (như `Converse API`), trừ khi ứng dụng cần capability đặc thù của một provider cụ thể.

2. **Managed RAG không có nghĩa là không cần hiểu RAG:**
   Knowledge Bases giảm công việc vận hành nhưng vẫn cần thiết kế chunking, retrieval, metadata, quyền truy cập và đánh giá chất lượng retrieval.

3. **Guardrails không phải hệ thống phân quyền:**
   Không nên sử dụng Guardrails như giải pháp thay thế cho authorization hoặc document-level access control.

4. **Không phải ứng dụng Generative AI nào cũng cần agent:**
   Nếu bài toán chỉ cần hỏi đáp hoặc RAG, kiến trúc đơn giản thường dễ vận hành và tiết kiệm chi phí hơn việc đưa AgentCore vào ngay từ đầu.

---

### 6. Kết Luận

**Amazon Bedrock** không đơn giản là một API gọi LLM mà là tập hợp nhiều building block cho Generative AI.

Điều quan trọng khi thiết kế hệ thống là xác định đúng nhu cầu:
1. **Inference**: Khi chỉ cần tương tác với foundation model.
2. **Knowledge Bases**: Khi cần RAG kết nối dữ liệu riêng.
3. **Guardrails**: Khi cần chính sách an toàn nội dung.
4. **AgentCore**: Khi thực sự cần một hệ thống agent có khả năng vận hành ở production.

Một kiến trúc Bedrock tốt không phải kiến trúc sử dụng nhiều dịch vụ nhất, mà là kiến trúc chỉ sử dụng những lớp thực sự cần cho bài toán hiện tại và vẫn đủ linh hoạt để mở rộng sau này.

---

### Nguồn Tham Khảo

1. [Amazon Bedrock — Overview & Core Concepts](https://docs.aws.amazon.com/bedrock/latest/userguide/what-is-bedrock.html)
2. [Amazon Bedrock — Runtime Endpoints and APIs](https://docs.aws.amazon.com/bedrock/latest/userguide/endpoints.html)
3. [Amazon Bedrock — Knowledge Bases for RAG](https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html)
4. [Amazon Bedrock — How Knowledge Base Retrieval Works](https://docs.aws.amazon.com/bedrock/latest/userguide/kb-how-retrieval.html)
5. [Amazon Bedrock — Agent Architecture & AgentCore](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-classic-maintenance-mode.html)

