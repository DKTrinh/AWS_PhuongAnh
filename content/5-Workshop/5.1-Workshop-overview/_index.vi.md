---
title : "Giới thiệu"
date : 2026-07-26
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

### Triển khai hạ tầng Web trên Cloud và thiết lập bảo mật hệ thống

<div style="text-align: center; margin: 20px 0;">

  ![Kiến trúc tổng thể](/images/5-Workshop/5.1-Introduction/architecture.png?classes=shadow)

  <div style="font-weight: bold; margin-top: 8px; color: #555;">Hình 1. Kiến trúc tổng thể của cơ sở hạ tầng mạng và sự tích hợp dịch vụ AWS cho dự án Tracker Maintenance.</div>
</div>

<br>

### Giới thiệu Tổng quan

Bài thực hành này ghi lại đầy đủ quy trình xây dựng, cấu hình và thử nghiệm cơ sở hạ tầng đám mây cho dự án Hệ thống Quản lý Bảo trì (Tracker Maintenance).

Cấu hình tập trung vào việc triển khai kiến trúc Web đa lớp (Multi-tier) trên AWS, tích hợp luồng xử lý sự kiện (Event-Driven) và bảo mật chuyên sâu, bao gồm:
* Thiết lập mạng riêng ảo (Amazon VPC) phân tách rõ ràng luồng truy cập giữa Public Subnet (để tiếp nhận request) và Private Subnet (cô lập và bảo vệ cơ sở dữ liệu Amazon RDS).
* Triển khai máy chủ Backend cốt lõi trên Amazon EC2 (FastAPI/Node.js) xử lý logic nghiệp vụ, đồng thời xây dựng hệ thống cấp phát token xác thực JWT độc lập để tối ưu kiểm soát quyền truy cập.
* Tối ưu hóa phân phối nội dung tĩnh của Frontend (React/Vue) lưu trữ trên Amazon S3 thông qua mạng phân phối nội dung Amazon CloudFront và dịch vụ phân giải tên miền Route 53.
* Xây dựng luồng tải file an toàn bằng cơ chế cấp quyền tạm thời (S3 Pre-signed URL), kết hợp kiến trúc Hướng sự kiện sử dụng AWS Lambda và Amazon SNS để tự động xử lý hình ảnh và phát thông báo đến kỹ thuật viên ngay khi có dữ liệu mới.
* Triển khai tính năng bảo vệ hệ thống chống tấn công dò mật khẩu (Brute-force protection) và thu thập, giám sát log tập trung thông qua Amazon CloudWatch.

Thông qua việc triển khai thực tế này, hệ thống Tracker Maintenance đã áp dụng và tuân thủ nghiêm ngặt các tiêu chuẩn của Khung kiến trúc AWS Well-Architected, tập trung mạnh mẽ vào ba trụ cột cốt lõi: Bảo mật (Security), Hiệu suất (Performance Efficiency) và Độ tin cậy (Reliability).