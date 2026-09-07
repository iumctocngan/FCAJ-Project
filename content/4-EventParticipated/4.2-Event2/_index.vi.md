---
title: "Event 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 4.2. </b> "
---


# Bài thu hoạch “GenAI-powered App-DB Modernization workshop”

### Mục Đích Của Sự Kiện

1. Chia sẻ best practices trong thiết kế ứng dụng hiện đại
2. Giới thiệu phương pháp DDD và event-driven architecture
3. Hướng dẫn lựa chọn compute services phù hợp
4. Giới thiệu công cụ AI hỗ trợ development lifecycle

### Danh Sách Diễn Giả

1. **Jignesh Shah** - Director, Open Source Databases
2. **Erica Liu** - Sr. GTM Specialist, AppMod
3. **Fabrianne Effendi** - Assc. Specialist SA, Serverless Amazon Web Services

### Nội Dung Nổi Bật

#### Đưa ra các ảnh hưởng tiêu cực của kiến trúc ứng dụng cũ

1. Thời gian release sản phẩm lâu → Mất doanh thu/bỏ lỡ cơ hội
2. Hoạt động kém hiệu quả → Mất năng suất, tốn kém chi phí
3. Không tuân thủ các quy định về bảo mật → Mất an ninh, uy tín

#### Chuyển đổi sang kiến trúc ứng dụng mới - Microservice Architecture

Chuyển đổi thành hệ thống modular – từng chức năng là một **dịch vụ độc lập** giao tiếp với nhau qua **sự kiện** với 3 trụ cột cốt lõi:

1. **Queue Management**: Xử lý tác vụ bất đồng bộ
2. **Caching Strategy:** Tối ưu performance
3. **Message Handling:** Giao tiếp linh hoạt giữa services

#### Domain-Driven Design (DDD)

1. **Phương pháp 4 bước**: Xác định domain events → sắp xếp timeline → identify actors → xác định bounded contexts
2. **Case study bookstore**: Minh họa cách áp dụng DDD thực tế
3. **Context mapping**: 7 patterns tích hợp bounded contexts

#### Event-Driven Architecture

1. **3 patterns tích hợp**: Publish/Subscribe, Point-to-point, Streaming
2. **Lợi ích**: Loose coupling, scalability, resilience
3. **So sánh sync vs async**: Hiểu rõ trade-offs (sự đánh đổi)

#### Compute Evolution

1. **Shared Responsibility Model**: Từ EC2 → ECS → Fargate → Lambda
2. **Serverless benefits**: No server management, auto-scaling, pay-for-value
3. **Functions vs Containers**: Criteria lựa chọn phù hợp

#### Amazon Q Developer

1. **SDLC automation**: Từ planning đến maintenance
2. **Code transformation**: Java upgrade, .NET modernization
3. **AWS Transform agents**: VMware, Mainframe, .NET migration

### Những Gì Học Được

#### Tư Duy Thiết Kế

1. **Business-first approach**: Luôn bắt đầu từ business domain, không phải technology
2. **Ubiquitous language**: Importance của common vocabulary giữa business và tech teams
3. **Bounded contexts**: Cách identify và manage complexity trong large systems

#### Kiến Trúc Kỹ Thuật

1. **Event storming technique**: Phương pháp thực tế để mô hình hóa quy trình kinh doanh
2. Sử dụng **Event-driven communication** thay vì synchronous calls
3. **Integration patterns**: Hiểu khi nào dùng sync, async, pub/sub, streaming
4. **Compute spectrum**: Criteria chọn từ VM → containers → serverless

#### Chiến Lược Hiện Đại Hóa

1. **Phased approach**: Không rush, phải có roadmap rõ ràng
2. **7Rs framework**: Nhiều con đường khác nhau tùy thuộc vào đặc điểm của mỗi ứng dụng
3. **ROI measurement**: Cost reduction + business agility

### Ứng Dụng Vào Công Việc

1. **Áp dụng DDD** cho project hiện tại: Event storming sessions với business team
2. **Refactor microservices**: Sử dụng bounded contexts để identify service boundaries
3. **Implement event-driven patterns**: Thay thế một số sync calls bằng async messaging
4. **Serverless adoption**: Pilot AWS Lambda cho một số use cases phù hợp
5. **Try Amazon Q Developer**: Integrate vào development workflow để boost productivity

### Trải nghiệm trong event

Tham gia workshop **“GenAI-powered App-DB Modernization”** là một trải nghiệm rất bổ ích, giúp tôi có cái nhìn toàn diện về cách hiện đại hóa ứng dụng và cơ sở dữ liệu bằng các phương pháp và công cụ hiện đại. Một số trải nghiệm nổi bật:

#### Học hỏi từ các diễn giả có chuyên môn cao
1. Các diễn giả đến từ AWS và các tổ chức công nghệ lớn đã chia sẻ **best practices** trong thiết kế ứng dụng hiện đại.
2. Qua các case study thực tế, tôi hiểu rõ hơn cách áp dụng **Domain-Driven Design (DDD)** và **Event-Driven Architecture** vào các project lớn.

#### Trải nghiệm kỹ thuật thực tế
1. Tham gia các phiên trình bày về **event storming** giúp tôi hình dung cách **mô hình hóa quy trình kinh doanh** thành các domain events.
2. Học cách **phân tách microservices** và xác định **bounded contexts** để quản lý sự phức tạp của hệ thống lớn.
3. Hiểu rõ trade-offs giữa **synchronous và asynchronous communication** cũng như các pattern tích hợp như **pub/sub, point-to-point, streaming**.

#### Ứng dụng công cụ hiện đại
1. Trực tiếp tìm hiểu về **Amazon Q Developer**, công cụ AI hỗ trợ SDLC từ lập kế hoạch đến maintenance.
2. Học cách **tự động hóa code transformation** và pilot serverless với **AWS Lambda**, từ đó nâng cao năng suất phát triển.

#### Kết nối và trao đổi
1. Workshop tạo cơ hội trao đổi trực tiếp với các chuyên gia, đồng nghiệp và team business, giúp **nâng cao ngôn ngữ chung (ubiquitous language)** giữa business và tech.
2. Qua các ví dụ thực tế, tôi nhận ra tầm quan trọng của **business-first approach**, luôn bắt đầu từ nhu cầu kinh doanh thay vì chỉ tập trung vào công nghệ.

#### Bài học rút ra
1. Việc áp dụng DDD và event-driven patterns giúp giảm **coupling**, tăng **scalability** và **resilience** cho hệ thống.
2. Chiến lược hiện đại hóa cần **phased approach** và đo lường **ROI**, không nên vội vàng chuyển đổi toàn bộ hệ thống.
3. Các công cụ AI như Amazon Q Developer có thể **boost productivity** nếu được tích hợp vào workflow phát triển hiện tại.

#### Một số hình ảnh khi tham gia sự kiện
*(Hình ảnh ghi nhận từ workshop)*
> Tổng thể, sự kiện không chỉ cung cấp kiến thức kỹ thuật mà còn giúp tôi thay đổi cách tư duy về thiết kế ứng dụng, hiện đại hóa hệ thống và phối hợp hiệu quả hơn giữa các team.
