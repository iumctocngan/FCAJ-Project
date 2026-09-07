---
title: "Blog 3"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# Tối ưu hạ tầng AI Agent trên AWS: Hiểu đúng mô hình chi phí trước khi tối ưu

> Khi AI agent chuyển từ giai đoạn thử nghiệm (demo/PoC) sang production, vấn đề không chỉ dừng lại ở việc agent có hoạt động thông minh hay không, mà còn là **mỗi phiên chạy bao lâu, sử dụng bao nhiêu tài nguyên và chi phí thực tế phát sinh từ đâu**.

Điểm đặc thù của AI agent là phần lớn thời gian thực thi dành cho việc **chờ I/O** (chờ phản hồi từ LLM, API bên ngoài, cơ sở dữ liệu hoặc tool thực thi). Vì vậy, hiểu đúng mô hình tính phí của **Amazon Bedrock AgentCore** là bước tiên quyết trước khi bắt tay vào tối ưu hóa hạ tầng.

---

### 1. AgentCore Runtime Tính Phí Như Thế Nào?

**AgentCore Runtime** là môi trường *serverless runtime* được thiết kế chuyên biệt cho AI Agent. Mỗi phiên làm việc (session) được khởi chạy bên trong một **microVM độc lập** nhằm cách ly hoàn toàn về CPU, memory và hệ thống tập tin (filesystem).

Runtime áp dụng mô hình **Active Resource Consumption** (tiêu thụ tài nguyên thực tế): thay vì phải trả tiền cho toàn bộ tài nguyên cấp phát trước (provisioned capacity), chi phí được tính chính xác dựa trên lượng tài nguyên CPU và RAM thực sự hoạt động trong session.

#### Bảng Biểu Phí Cơ Bản (Tham Khảo):

| Thành phần tài nguyên | Mức giá niêm yết (USD) | Đơn vị tính & Quy chuẩn |
| :--- | :--- | :--- |
| **vCPU** | **$0.0895** / vCPU-hour | Tính theo giây (tối thiểu 1 giây) |
| **Memory (RAM)** | **$0.00945** / GB-hour | Tính theo giây, mức tối thiểu tính phí: 128 MB |

Theo thống kê từ AWS, các tác vụ AI Agent thường dành từ **30% đến 70% tổng thời gian phiên cho việc chờ I/O** (*I/O wait*). Trong suốt khoảng thời gian chờ này, **CPU hoàn toàn không bị tính phí** nếu không có tiến trình chạy ngầm (background process) nào tiêu thụ CPU.

Cơ chế này cực kỳ phù hợp với bản chất hoạt động ngắt quãng của Agent:
```text
Suy luận logic → [Chờ LLM sinh token] → Thực thi Tool → [Chờ API ngoài phản hồi] → Xử lý kết quả
```

---

### 2. Quản Lý và Kiểm Soát Session Lifetime

AgentCore Runtime cung cấp các tham số vòng đời (lifecycle settings) quan trọng để kiểm soát thời gian tồn tại của session:

1. **`idleRuntimeSessionTimeout`**: Khoảng thời gian session được phép duy trì trạng thái nhàn rỗi (idle) khi không có request mới trước khi tự động giải phóng (mặc định khoảng **15 phút**).
2. **`maxLifetime`**: Thời gian tồn tại tối đa tuyệt đối của một session (có thể cấu hình tối đa lên tới **8 giờ**).

> [!TIP]
> **Khuyến nghị thực tế:** Không nên giữ session tồn tại lâu hơn nhu cầu thực tế của nghiệp vụ. Với các agent xử lý tác vụ ngắn (short-lived tasks), việc cấu hình timeout phù hợp giúp hệ thống giải phóng microVM sớm hơn, tránh lãng phí tài nguyên và không giữ lại trạng thái dư thừa trong bộ nhớ.

---

### 3. Tối Ưu CPU và Memory Theo Hành Vi Thực Tế Của Agent

Với mô hình *active-consumption pricing*, tối ưu hóa chi phí không chỉ đơn giản là cố gắng rút ngắn tổng thời lượng session mà là **giảm thiểu thời gian CPU active và giảm dung lượng RAM thực tế**:

#### 1. Giảm Active CPU Time
Trong lúc Agent đang ở trạng thái chờ LLM hoặc API ngoài trả kết quả, cần triệt để tránh:
1. Cơ chế **polling liên tục** (busy-waiting loops).
2. Các **vòng lặp retry quá dày** với khoảng cách thời gian (backoff) quá ngắn.
3. Các **background threads/tasks** chạy ngầm không cần thiết.
4. Việc **parse/serialize dữ liệu lặp đi lặp lại** nhiều lần.

*(Nếu có tiến trình chạy ngầm làm CPU hoạt động trong lúc chờ I/O, thời gian đó vẫn bị tính phí vCPU bình thường).*

#### 2. Giảm Memory Footprint
Chi phí RAM được tính dựa trên mức dung lượng thực tế sử dụng theo thời gian:
1. Tránh lưu giữ dữ liệu trung gian (intermediate results), tệp tin tải về tạm thời hoặc context cũ trong RAM suốt chiều dài session.
2. Chủ động giải phóng các biến lớn hoặc đẩy dữ liệu tạm sang lưu trữ ngoài (như Amazon S3 hoặc Cache) khi đã xử lý xong.

---

### 4. Runtime Không Phải Toàn Bộ Hóa Đơn (Total Cost of Ownership)

Một sai lầm rất phổ biến là đội ngũ kỹ sư chỉ tập trung tối ưu CPU và RAM của Runtime mà bỏ qua các cấu phần chi phí khác. 

Trên thực tế, tổng chi phí vận hành một AI Agent gồm **6 thành phần chính**:

```text
Tổng Chi Phí AI Agent =
    1. Runtime (vCPU & Memory thực tế)
  + 2. Model Inference (Input/Output Tokens & số lần gọi Foundation Model)
  + 3. Gateway (Số lượng tool calls qua Gateway)
  + 4. Memory Storage (Chi phí lưu trữ & truy vấn short-term / long-term memory)
  + 5. Sandboxed Tools (Browser instances / Code Interpreter compute)
  + 6. Observability (Logs, Metrics, Traces qua Amazon CloudWatch)
```

> [!IMPORTANT]
> **Trọng số chi phí thực tế:** Trong phần lớn các bài toán thực tế, chi phí **Model Inference (Token & API calls)** thường chiếm tỷ trọng lớn hơn rất nhiều so với chi phí Compute Runtime.

Nếu một Agent thiết kế luồng suy luận kém hiệu quả và lặp lại quá nhiều bước reasoning không cần thiết:
```text
User Prompt → LLM Call 1 → Tool Call 1 → LLM Call 2 → Tool Call 2 → LLM Call 3 → Final Response
```

Mỗi bước lặp sẽ nhân thêm lượng token và chi phí gọi mô hình. Do đó, việc **tối ưu prompt, thu gọn context window, giới hạn số bước reasoning và chọn model kích cỡ phù hợp** (ví dụ: dùng model nhỏ cho bước routing, model lớn cho bước tổng hợp) mang lại hiệu quả tiết kiệm chi phí vượt trội hơn nhiều so với việc chỉ giảm vài mili-giây runtime.

---

### 5. Phân Tích Ví Dụ Về Active-Consumption Pricing

AWS đưa ra ví dụ minh họa trực quan với một Agent Session kéo dài trong tổng cộng 60 giây. Trong đó, 70% thời gian (42 giây) là thời gian chờ I/O (chờ LLM inference và API bên thứ ba), còn thời gian CPU thực sự xử lý logic là 18 giây (chiếm 30%).

```text
Tổng thời gian Session: 60 giây
├──────────────────────────────┬─────────────────────────────────────────┤
│    Active CPU: 18 giây (30%) │          I/O Wait: 42 giây (70%)         │
│     (Bị tính phí vCPU)       │     (KHÔNG bị tính phí vCPU active)     │
└──────────────────────────────┴─────────────────────────────────────────┘
```

Nếu agent sử dụng 1 vCPU, hệ thống chỉ tính tiền tiêu thụ vCPU cho đúng 18 giây active thay vì tính toàn bộ 60 giây như mô hình máy ảo truyền thống. Đồng thời, Memory cũng được đo lường linh hoạt theo mức sử dụng thực tế ở từng giai đoạn thay vì áp dụng mức đỉnh (peak memory) cho toàn bộ 60 giây.

*(Lưu ý: Đây là số liệu minh họa theo kịch bản chuẩn của AWS. Chi phí thực tế sẽ phụ thuộc cụ thể vào đặc thù thuật toán và tần suất gọi I/O của từng ứng dụng).*

---

### 6. Checklist Tối Ưu Hạ Tầng AI Agent Trước Khi Lên Production

Trước khi triển khai chính thức lên môi trường production, hãy rà soát theo 7 tiêu chí cốt lõi:

1. [ ] **Session Timeout:** Đã cấu hình `idleRuntimeSessionTimeout` và `maxLifetime` sát với hành vi người dùng chưa?
2. [ ] **Active CPU:** Có tiến trình chạy ngầm, polling liên tục hoặc vòng lặp retry không cần thiết trong lúc chờ I/O không?
3. [ ] **Memory Footprint:** Có giải phóng bộ nhớ cho các payload/tệp tin trung gian lớn sau khi xử lý xong không?
4. [ ] **Model Inference:** Có đang gửi context quá dài hoặc gọi foundation model nhiều lần dư thừa không?
5. [ ] **Tool Calling:** Agent có gọi các tool trung gian không hiệu quả hoặc trùng lặp dữ liệu không?
6. [ ] **Long-term Memory:** Đã thiết lập chính sách lưu giữ dữ liệu (retention policy / TTL) cho bộ nhớ dài hạn chưa?
7. [ ] **Observability:** Mức độ log (log level) và telemetry gửi về CloudWatch có bị quá mức cần thiết không?

> [!TIP]
> **Chỉ số đo lường chuẩn:** Hãy đo lường chi phí dựa trên các chỉ số nghiệp vụ cụ thể như Cost per agent session (chi phí trên mỗi phiên tương tác) và Cost per completed task (chi phí trên mỗi tác vụ hoàn thành). Thay vì chỉ nhìn vào tổng hóa đơn hàng tháng, các chỉ số này giúp xác định chính xác thành phần nào đang gây lãng phí ngân sách.

---

### 7. Kết Luận

Tối ưu hóa hạ tầng AI Agent trên AWS không đơn thuần là việc tìm kiếm cấu hình compute có giá rẻ nhất.

Chi phí thực tế của hệ thống là tổng hòa của nhiều lớp:
```text
Runtime vCPU/RAM + Model Tokens + Tools/Gateway + Memory Storage + Observability
```

Amazon Bedrock AgentCore với mô hình **active-consumption pricing** mang lại lợi thế tài chính vượt trội cho các tác vụ I/O-heavy. Tuy nhiên, lợi thế này chỉ được phát huy tối đa khi Agent được thiết kế bài bản: loại bỏ xử lý ngầm dư thừa, kiểm soát chặt chẽ vòng đời session, quản lý memory thông minh và tối ưu hóa số bước gọi mô hình.

Hãy tiếp cận việc tối ưu hóa bằng dữ liệu đo lường cụ thể (*Cost per Session/Task*) để xác định đúng "điểm nghẽn chi phí" và giải quyết triệt để.

---

### Nguồn Tham Khảo

1. [Amazon Bedrock AgentCore — Overview](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/what-is-bedrock-agentcore.html)
2. [Amazon Bedrock AgentCore — Pricing](https://aws.amazon.com/bedrock/agentcore/pricing/)
3. [Amazon Bedrock AgentCore — Runtime Concepts](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/agents-tools-runtime.html)
4. [Amazon Bedrock AgentCore — Runtime Lifecycle Settings](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/runtime-lifecycle-settings.html)

> *Lưu ý: Mức giá có thể thay đổi theo thời gian và từng AWS Region. Hãy tham khảo trang Pricing chính thức của AWS khi lập dự toán ngân sách.*  