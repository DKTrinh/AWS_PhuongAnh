---
title : "Kết quả thử nghiệm & Thực nghiệm"
date : 2024-01-01 
weight : 4 
chapter : false
pre : " <b> 5.4. </b> "
---
### Kết quả Thử nghiệm & Thực nghiệm

> [!TIP]
> 🎬 **Video Demo:** Xem trực tiếp video demo kiểm thử hệ thống Tracker Maintenance trên YouTube: 


Sau khi hoàn tất cấu hình cơ sở hạ tầng mạng cô lập và hệ thống Serverless, quá trình thử nghiệm thực tế từ đầu đến cuối (end-to-end) đã được tiến hành để đánh giá tính ổn định của luồng thu thập nhật ký bảo trì thiết bị ESP32:

#### Thử nghiệm Luồng Đăng ký & Xác thực Kỹ thuật viên (AWS Cognito)

Thực hiện cấp tài khoản cho Kỹ thuật viên bảo trì trên ứng dụng quản lý nội bộ.

![Luồng Đăng ký Cognito](/images/5-Workshop/5.4-Test_Results_Experimentation/tracker-cognito-auth.png?classes=shadow)

> [!WARNING]
> Hình ảnh chức năng tạo tài khoản và phân quyền truy cập cho Kỹ thuật viên bảo trì thiết bị

Hệ thống AWS Cognito ghi nhận chính xác kỹ thuật viên mới và tự động gửi thông tin đăng nhập tạm thời về email. Kỹ thuật viên đăng nhập lần đầu, đổi mật khẩu và trạng thái tài khoản trên hệ thống chuyển sang "Confirmed", cấp quyền truy cập vào bảng điều khiển bảo trì.

![Quản trị Người dùng Cognito](/images/5-Workshop/5.4-Test_Results_Experimentation/tracker-cognito-users.png?classes=shadow)

> [!WARNING]
> Giao diện quản lý danh sách Kỹ thuật viên bảo trì và trạng thái xác thực trên AWS Cognito User Pool

#### Thử nghiệm Luồng Tải Nhật ký Lỗi Thiết bị (Crash Logs) qua S3 Presigned URL

Mô phỏng một thiết bị ESP32 Tracker gặp sự cố phần cứng. Thiết bị tự động gửi yêu cầu (payload) chứa mã lỗi và MAC Address tới API Gateway để kích hoạt hàm AWS Lambda `Process_ESP32_Tracker_Telemetry`.

Hàm Lambda nằm trong Private Subnet thực thi mượt mà, xác thực chữ ký của thiết bị và gọi qua S3 Gateway Endpoint nội bộ. Nó yêu cầu Amazon S3 cấp một đường liên kết mã hóa tạm thời (Presigned URL) có hiệu lực 5 phút để thiết bị có thể upload file log cục bộ.

Sau khi nhận được liên kết, module Wi-Fi của ESP32 thực hiện một HTTP PUT request để đẩy trực tiếp tệp `.txt` chứa nhật ký lỗi vào Amazon S3 Bucket `tracker-maintenance-storage`.

Truy cập S3 Console, tệp log của thiết bị đã được phân loại tự động vào đúng thư mục ngày tháng với dung lượng chính xác, minh chứng cho việc luồng kết nối bảo mật nội bộ VPC đã hoạt động thành công 100%.

![Tải log lên S3 Thành công](/images/5-Workshop/5.4-Test_Results_Experimentation/tracker-s3-upload.png?classes=shadow)

> [!WARNING]
> Hình ảnh tệp nhật ký chẩn đoán phần cứng ESP32 được tải lên an toàn hệ thống S3

#### Thử nghiệm Hệ thống Cảnh báo Lỗi Phần cứng với CloudWatch & SNS

Cố tình ngắt kết nối cảm biến trên phần cứng ESP32 để thiết bị gửi chuỗi cảnh báo "HARDWARE_FAULT" và "SENSOR_TIMEOUT" liên tục về hệ thống Backend.

Ngay khi dữ liệu được tiếp nhận, CloudWatch Log Groups lập tức ghi nhận nhật ký (logs). Bộ lọc chỉ số (Metric Filter) `TrackerHardwareErrorFilter` bắt được các từ khóa lỗi nghiêm trọng và đẩy chỉ số vượt ngưỡng cấu hình (>= 5 lỗi trong 5 phút).

Hệ thống lập tức chuyển sang trạng thái ALARM. Trong chưa đầy 30 giây, một email thông báo yêu cầu bảo trì khẩn cấp từ Amazon SNS đã được gửi trực tiếp tới hộp thư quản trị, kèm theo ID của thiết bị đang gặp sự cố. Điều này chứng minh luồng giám sát và tự động phát cảnh báo bảo trì hoạt động hoàn toàn chính xác.

![CloudWatch Alarm Triggered](/images/5-Workshop/5.4-Test_Results_Experimentation/tracker-cloudwatch-alarm.png?classes=shadow)

> [!WARNING]
> Cảnh báo khẩn cấp được kích hoạt trên CloudWatch gửi email điều phối bảo trì cho thiết bị lỗi