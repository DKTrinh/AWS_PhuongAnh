---
title: "Blog 1"
date: 2026-07-26
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# QUẢN LÝ VÀ KIỂM SOÁT TRUY CẬP AMAZON BEDROCK TẬP TRUNG VỚI MÔ HÌNH AI GATEWAY TRÊN AWS

Xin chào mọi người!

Khi triển khai Generative AI trong doanh nghiệp, bài toán governance (quản trị) luôn là thử thách lớn: làm sao để phân quyền (authorization), kiểm soát chi phí, giới hạn lưu lượng (rate limiting) và đảm bảo tính cô lập dữ liệu giữa các tenant?

Hôm nay mình muốn chia sẻ về mô hình AI Gateway do Dynatrace phát triển và được AWS chuẩn hóa thành reference pattern. Kiến trúc này giúp quản lý tập trung toàn bộ truy cập vào Amazon Bedrock bằng cách kết hợp các dịch vụ serverless quen thuộc.

## Bối Cảnh & Vấn Đề

Nếu ứng dụng client gọi trực tiếp tới Amazon Bedrock qua AWS SDK, hệ thống sẽ gặp một số hạn chế:

- Khó tích hợp với các hệ thống identity có sẵn (như OAuth2/JWT).
- Khó quản lý quota và chi phí theo từng team/tenant trong kiến trúc multi-tenant.
- Phải cập nhật code trung gian mỗi khi Bedrock ra mắt tính năng hay API mới.

**Giải pháp:** Xây dựng một lớp AI Gateway đứng trước Bedrock. Lớp này trong suốt (transparent) với client — lập trình viên vẫn dùng Boto3/AWS SDK bình thường — trong khi toàn bộ luồng auth, quota và streaming được xử lý tập trung phía sau.

## Những Điểm Nổi Bật Của Kiến Trúc

- **Thiết kế Future-Proof với Dynamic Forwarder:** Lambda Integration đóng vai trò ký AWS SigV4 và chuyển tiếp request. Bất kể Bedrock thêm model hay API mới, AI Gateway vẫn xử lý mượt mà mà không cần sửa code.
- **Tích hợp Identity linh hoạt:** Dùng Lambda Authorizer để xác thực JWT token từ Auth0, Keycloak hay Okta trước khi request chạm tới Bedrock.
- **Hỗ trợ Response Streaming real-time:** Tận dụng tính năng API Gateway Response Streaming để truyền trực tiếp từng "chunk" dữ liệu từ LLM về client với độ trễ tối thiểu.
- **Chống Noisy Neighbor & Kiểm soát chi phí:** Dễ dàng áp dụng Usage Plans, API Keys và rate limiting trên API Gateway để kiểm soát lưu lượng từng team.

## Tình Huống Thực Tế: Luồng Xử Lý Request

Khi client gọi API ConverseStream của mô hình Claude 3.5 Haiku qua AI Gateway, luồng xử lý diễn ra như sau:

```text
Client (Boto3 SDK)
  │ (API call unsigned + JWT header)
  ▼
Amazon API Gateway
  │ (Gọi Lambda Authorizer kiểm tra JWT)
  ▼
AWS Lambda Integration
  │ (Giữ nguyên request -> ký AWS SigV4)
  ▼
Amazon Bedrock
  │ (Stream response real-time)
  ▼
Client nhận dữ liệu dạng stream
````

Khởi tạo client: client trỏ `endpoint_url` về API Gateway, tắt chế độ tự ký client-side (`signature_version = UNSIGNED`) và đính kèm JWT token trong header.

Xác thực tại API Gateway: API Gateway gọi Lambda Authorizer để kiểm tra tính hợp lệ của JWT token.

Ký chữ ký số & chuyển tiếp: Lambda Integration tiếp nhận request, giữ nguyên payload/parameters, ký chữ ký AWS SigV4 bằng IAM role của Lambda rồi forward tới Amazon Bedrock.

Phản hồi dạng stream: API Gateway stream trực tiếp từng chunk dữ liệu (`contentBlockDelta`) từ Bedrock về client.

## Khả Năng Mở Rộng

Nhờ đặt ở tầng API Gateway, bạn có thể dễ dàng bổ sung các tính năng nâng cao:

* Prompt & Response Caching: cache các câu hỏi phổ biến để giảm chi phí API và độ trễ.
* AWS WAF Integration: thêm các rule bảo mật chống SQLi, XSS hoặc giới hạn IP.
* Custom Content Filtering: thêm logic lọc dữ liệu nhạy cảm (PII) tại Lambda trước khi gửi prompt tới Bedrock.

## Kết Luận

Mô hình AI Gateway cân bằng tốt giữa developer experience và enterprise governance. Developer phía client vẫn dùng SDK chính thống mượt mà, trong khi team Security & Cloud Ops hoàn toàn kiểm soát được quyền truy cập, lưu lượng và chi phí hệ thống.

## Tài liệu tham khảo

* AWS Compute Blog – Building an AI gateway to Amazon Bedrock with Amazon API Gateway
* GitHub Repository: amazon-api-gateway-ai-gateway-pattern


<img src="/images/Blogs/blog1.png" alt="Blog 1" width="1000" />