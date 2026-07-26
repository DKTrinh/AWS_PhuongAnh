---
title: "Các bước chuẩn bị"
date: 2026-07-26
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

### Các bước chuẩn bị

#### Tài khoản và Truy cập (AWS IAM & CLI)

Hệ thống yêu cầu một tài khoản AWS đang hoạt động với quyền quản trị viên IAM. Để tuân thủ Nguyên tắc Quyền hạn tối thiểu (Principle of Least Privilege), nhóm phát triển không sử dụng tài khoản Root chính mà thay vào đó tạo một tài khoản nhà phát triển phụ tên là **Tracker-Developer** để cấp quyền và lấy các khóa bảo mật nhằm kết nối từ máy trạm làm việc thông qua các bước sau:

- **Bước 1 (Cấu hình trên AWS Console):** Đăng nhập vào AWS Management Console bằng tài khoản quản trị viên và điều hướng đến **IAM (Identity and Access Management)** → **Users (Người dùng)** → chọn người dùng **Tracker-Developer**. Di chuyển đến tab **Security credentials (Thông tin bảo mật)**, tìm đến phần **Access keys (Khóa truy cập)** và chọn **Create access key (Tạo khóa truy cập)**. Chọn **Command Line Interface (CLI)** làm trường hợp sử dụng (use case), đồng ý với các điều khoản và xác nhận để hệ thống tạo cặp khóa: **Access Key ID** và **Secret Access Key**. Tải xuống tệp `.csv` chứa thông tin khóa bảo mật này.

- **Bước 2 (Cấu hình trên máy trạm cục bộ):** Mở Terminal hoặc PowerShell trên máy tính cá nhân và chạy lệnh sau:

```bash
aws configure
````

Nhập **Access Key ID** và **Secret Access Key** đã được tạo ở Bước 1. Thiết lập **Default region name** thành `ap-southeast-1` (Singapore - vùng triển khai lý tưởng để tối ưu hóa độ trễ mạng cho người dùng tại Việt Nam) và **Default output format** thành `json`. Cấu hình này sẽ tự động được lưu trong thư mục người dùng (`~/.aws/` trên Linux/macOS hoặc `%USERPROFILE%\.aws\` trên Windows).

> [!NOTE]
> Tab **IAM Security credentials** với phần **Access keys** được sử dụng để tạo các khóa kết nối CLI.

#### Môi trường Máy trạm Cục bộ và Mã nguồn

Đảm bảo máy trạm đã cài đặt thành công môi trường **Node.js** (phiên bản 18 trở lên) và **Python** (phiên bản 3.10 trở lên) để phục vụ cho việc phát triển Backend (FastAPI/Node.js) cũng như đóng gói giao diện Frontend (React/Vue).

Cài đặt **Git** để quản lý mã nguồn. Cần đặc biệt chú ý kiểm tra cấu hình tệp `.gitignore` chuẩn xác trước khi commit để tránh việc vô tình đẩy các thông tin nhạy cảm (như API keys) lên kho lưu trữ trực tuyến.

Chuẩn bị sẵn tệp `.env` tại môi trường local chứa các biến môi trường thiết yếu như chuỗi kết nối cơ sở dữ liệu (`DB_URL`) và khóa bí mật mã hóa token (`JWT_SECRET`).

#### Lựa chọn Vùng Cơ sở hạ tầng

Lựa chọn vùng cơ sở hạ tầng mạng AWS tại Singapore (`ap-southeast-1`) làm vùng triển khai mặc định. Điều này đảm bảo tốc độ phản hồi (latency) thấp nhất, mang lại trải nghiệm truy cập mượt mà cho hệ thống Tracker Maintenance, đồng thời hỗ trợ đầy đủ các dịch vụ AWS cốt lõi cấu thành nên kiến trúc.


