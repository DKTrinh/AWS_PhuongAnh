---
title: "Bản đề xuất"
date: 2026-07-26
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

## Tracker Maintenance – Hệ thống quản lý bảo trì trên nền tảng AWS
### Giải pháp vận hành an toàn với kiến trúc Cloud Web Đa lớp và xác thực JWT

### 1. Tóm tắt điều hành
Tracker Maintenance là một hệ thống quản lý các tác vụ bảo trì được xây dựng trên kiến trúc Multi-tier (Đa lớp) hiện đại trên Amazon Web Services (AWS). Từ góc độ kỹ thuật, giao diện người dùng (Frontend) được phát triển bằng React/Vue, trong khi hệ thống xử lý cốt lõi (Backend) được xây dựng bằng Node.js/FastAPI, kết hợp với cơ sở dữ liệu quan hệ bảo mật. Hệ thống tự chủ hoàn toàn trong việc quản lý danh tính thông qua cơ chế JWT Auth. 

### 2. Mục tiêu
Với Tracker Maintenance, mục tiêu cốt lõi tập trung vào việc tối ưu hóa quy trình quản lý và nâng cao tối đa bảo mật hệ thống:
- Xây dựng hạ tầng Cloud ổn định, tách biệt rõ ràng luồng truy cập Front-end và Back-end.
- Thay thế các giải pháp xác thực phụ thuộc (như Cognito) bằng hệ thống JWT nội bộ linh hoạt và tự chủ hơn.
- Triển khai cơ chế bảo vệ chủ động chống lại các cuộc tấn công dò mật khẩu (Brute-force protection).
- Tối ưu hóa tải máy chủ bằng cách sử dụng luồng tải file trực tiếp lên S3 (Pre-signed URL) và xử lý sự kiện tự động.

### 3. Tuyên bố vấn đề
- **Tình trạng hiện tại:** Các hệ thống theo dõi truyền thống thường dễ bị tổn thương trước các cuộc tấn công brute-force, cấu hình lộ lọt API Key và thường xuyên gặp hiện tượng nghẽn cổ chai khi xử lý tải file dung lượng lớn thông qua máy chủ chính.
- **Giải pháp:** Tracker Maintenance tận dụng mạng riêng ảo VPC trên AWS để bảo vệ cơ sở dữ liệu. Hệ thống chuyển đổi sang xác thực JWT, bảo mật nghiêm ngặt cấu hình bằng file `.env` và áp dụng kiến trúc Hướng sự kiện (Event-Driven) với S3 và Lambda để xử lý file.
- **Lợi ích:** Mang lại một hệ thống có tính sẵn sàng cao, bảo mật chặt chẽ từ lớp mạng đến lớp ứng dụng, đồng thời cung cấp trải nghiệm mượt mà cho người dùng cuối.

### 4. Kiến trúc hệ thống
Toàn bộ cơ sở hạ tầng được triển khai trên AWS trong mạng nội bộ Amazon VPC, được phân chia rành mạch thành Public Subnet (chứa máy chủ EC2) và Private Subnet (chứa cơ sở dữ liệu RDS).

**Công nghệ sử dụng:**
- **Frontend:** React/Vue (Lưu trữ tĩnh và phân phối qua CDN).
- **Backend:** Node.js / FastAPI.
- **Cơ sở dữ liệu:** Amazon RDS (PostgreSQL/MySQL).

**Các dịch vụ AWS cốt lõi:**
- **Amazon EC2:** Máy chủ chính chạy ứng dụng Backend, xử lý logic xác thực JWT và kiểm soát Brute-force.
- **Amazon S3:** Lưu trữ tĩnh cho Frontend và là kho chứa các tệp tin/hình ảnh tải lên từ người dùng.
- **Amazon CloudFront & Route 53:** Phân phối nội dung toàn cầu (CDN) và phân giải tên miền tốc độ cao.
- **AWS Lambda & Amazon SNS:** Xử lý luồng sự kiện tự động (Event-Driven). Khi có file mới trên S3, Lambda sẽ được kích hoạt để xử lý và SNS sẽ gửi thông báo đến kỹ thuật viên.
- **Amazon CloudWatch:** Dịch vụ giám sát và lưu trữ log tập trung để phát hiện lỗi hệ thống.

![Kiến trúc hệ thống](/images/2-Proposal/architecture.png?classes=shadow)

**Các luồng dữ liệu chính:**
- **Luồng xác thực bảo mật:** Người dùng gửi yêu cầu đăng nhập. Máy chủ EC2 kiểm tra logic Brute-force (chặn nếu sai quá nhiều lần). Nếu hợp lệ, Backend cấp phát một JWT Token an toàn để người dùng truy cập các API nghiệp vụ.
- **Luồng tải file tối ưu:** Frontend gọi API đến EC2 để xin cấp quyền. EC2 trả về một S3 Pre-signed URL tạm thời. Frontend sử dụng URL này để tải file trực tiếp lên S3, bỏ qua việc truyền tải nặng nề qua EC2.
- **Luồng thông báo sự kiện:** Ngay khi file được tải lên S3 thành công (Event Trigger), AWS Lambda sẽ tự động khởi chạy logic xử lý và kích hoạt Amazon SNS để đẩy thông báo theo thời gian thực.

### 5. Triển khai kỹ thuật
Nhóm phát triển chia sẻ các nhiệm vụ kỹ thuật để đảm bảo tiến độ dự án:
- **Xây dựng Front-end & Back-end:** WhooDuck1810 và phuonganh284 phối hợp phát triển giao diện React/Vue, thiết lập môi trường bảo mật `.env` và viết logic API trên Node.js/FastAPI.
- **Bảo mật Xác thực:** Gỡ bỏ tích hợp AWS Cognito, lập trình và chuyển đổi toàn bộ sang cơ chế xác thực JWT kết hợp tính năng Anti-login protection.
- **Triển khai Hạ tầng AWS:** Thiết lập kiến trúc mạng VPC, cấu hình S3 Pre-signed URL và luồng sự kiện Lambda - SNS.

### 6. Lộ trình triển khai
Lộ trình triển khai dự án Tracker Maintenance diễn ra trong 8 tuần:
- **Tuần 1-2:** Tìm hiểu tổng quan kiến trúc Web trên AWS. Khởi tạo repository và cấu trúc mã nguồn.
- **Tuần 3-4:** Xây dựng khung ứng dụng cơ bản. Thiết lập biến môi trường bảo mật (`.env`), dọn dẹp các API key bị lộ khỏi lịch sử Git.
- **Tuần 5-6:** Triển khai ứng dụng lên hạ tầng AWS (EC2, S3, CloudFront), tích hợp cơ sở dữ liệu RDS và cấu hình Amplify/Cognito bước đầu.
- **Tuần 7-8:** Tối ưu hóa xác thực (thay thế Cognito bằng JWT). Triển khai tính năng chống Brute-force, tối ưu hóa UI/UX và hoàn thiện tài liệu hệ thống.

### 7. Ước tính chi phí
Kiến trúc kết hợp giữa EC2 và các dịch vụ quản lý của AWS giúp tối ưu hóa chi phí:
- **Amazon EC2 & RDS:** Sử dụng các phiên bản máy chủ nhỏ (t2.micro/t3.micro) nằm trong giới hạn Free Tier cho môi trường phát triển.
- **Amazon S3 & CloudFront:** Chi phí phân phối nội dung tĩnh và lưu trữ rất thấp, hầu hết được bao phủ bởi gói miễn phí hàng tháng.
- **AWS Lambda & SNS:** Tính phí dựa trên số lần gọi hàm và số lượng tin nhắn, cực kỳ tiết kiệm cho luồng xử lý sự kiện.

### 8. Đánh giá rủi ro
- **Rủi ro lộ thông tin nhạy cảm:** Được giải quyết triệt để thông qua việc sử dụng tệp `.env` đồng bộ hóa chặt chẽ và cấu hình `.gitignore` chuẩn.
- **Tấn công dò mật khẩu (Brute-force):** Được ngăn chặn bằng logic Anti-login protection lập trình trực tiếp tại Backend.
- **Xâm nhập cơ sở dữ liệu:** Rủi ro rất thấp nhờ thiết kế mạng VPC cô lập hoàn toàn Amazon RDS trong Private Subnet, không cho phép truy cập trực tiếp từ Internet.

### 9. Kết quả kỳ vọng
Triển khai thành công hệ thống Tracker Maintenance bảo mật, hoạt động ổn định và tối ưu hiệu năng. Dự án là minh chứng cho khả năng kết hợp nhuần nhuyễn giữa kiến trúc Web đa lớp truyền thống (EC2, RDS) và các tính năng Cloud hiện đại (S3 Pre-signed URL, Event-Driven Lambda), mang lại một công cụ quản lý bảo trì mạnh mẽ và tự chủ hoàn toàn về mặt xác thực.
