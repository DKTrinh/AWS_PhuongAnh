---
title : "Khó khăn & Hướng phát triển"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---
### Khó khăn & Hướng phát triển

#### Khó khăn Gặp phải

* **Lỗi phân quyền mạng ẩn (ENI)**: Trong giai đoạn đầu đưa hàm Lambda `Process_ESP32_Tracker_Telemetry` vào mạng con Private VPC, hệ thống đã từ chối quyền truy cập mạng. Sau khi tra cứu tài liệu lỗi `CreateNetworkInterface` trên AWS Knowledge Center, tác giả nhận ra hàm thiếu đặc quyền tạo giao diện mạng ảo nội bộ và đã khắc phục bằng cách gắn thêm policy `AWSLambdaVPCAccessExecutionRole` cho IAM Role.
* **Xung đột chính sách bảo mật (Bucket Policy vs. Endpoint Policy)**: Khi thiết lập bảo mật cho bucket `tracker-maintenance-storage`, việc khóa toàn bộ truy cập public quá sớm—trước khi chỉ định chính xác định danh ARN của VPC Endpoint—đã làm cho chính hàm Lambda nội bộ bị từ chối quyền cấp S3 Presigned URL (Access Denied). Tác giả đã phải lần vết qua CloudWatch logs và dùng AWS CLI để gỡ rối, từ đó điều chỉnh lại trình tự cấu hình.
* **Xử lý dữ liệu cảm biến không đồng nhất**: Các gói tin (payload) gửi từ mạch ESP32 đôi khi bị gián đoạn hoặc sai định dạng JSON do nhiễu sóng mạng hoặc pin yếu. Điều này ban đầu gây ra lỗi crash cho hệ thống Backend. Tác giả đã phải bổ sung cơ chế kiểm tra tính hợp lệ của dữ liệu đầu vào (data validation) và các khối `try-catch` nghiêm ngặt hơn cho hàm Lambda.

#### Hướng phát triển Tương lai

* **Triển khai Hạ tầng dưới dạng mã (IaC)**: Chuyển đổi toàn bộ quy trình thiết lập thủ công trên Web Console (VPC, Subnets, Lambda, Endpoint, IAM, S3) sang dạng mã nguồn quản lý tập trung bằng AWS CloudFormation hoặc Terraform. Điều này giúp dễ dàng nhân bản môi trường để phục vụ hàng ngàn thiết bị IoT chỉ bằng một câu lệnh triển khai.
* **Tích hợp giao thức MQTT với AWS IoT Core**: Nhằm tối ưu hóa thời lượng pin cho thiết bị ESP32 và đảm bảo kết nối ổn định trong điều kiện mạng yếu, hệ thống dự kiến sẽ chuyển từ việc gọi HTTP REST API (qua API Gateway) sang giao thức MQTT hạng nhẹ chuyên dụng cho IoT thông qua dịch vụ AWS IoT Core.
* **Xây dựng Dashboard Quản trị Thiết bị (Fleet Management)**: Đồng bộ các chỉ số rời rạc từ CloudWatch Alarms và log hệ thống lên một giao diện giám sát tổng thể. Giao diện này sẽ hiển thị trực quan bản đồ vị trí, dung lượng pin, và cảnh báo lỗi phần cứng của toàn bộ cụm thiết bị tracker, giúp đội ngũ kỹ thuật dễ dàng điều phối công tác bảo trì ngoài thực địa.