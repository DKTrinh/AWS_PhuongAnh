---
title : "Các bước triển khai - Tracker Maintenance"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---
### Các bước triển khai chi tiết

#### Bước 1: Thiết lập VPC riêng biệt, cô lập (Isolated Private VPC)
Quá trình xây dựng môi trường mạng cô lập để đặt toàn bộ luồng xử lý dữ liệu bảo trì thiết bị vào một vùng an toàn:

Khởi tạo một VPC tùy chỉnh có tên là **Tracker-VPC** với khối IP CIDR là `10.1.0.0/16` làm dải mạng riêng chính.

![Danh sách VPC](/images/5-Workshop/5.3-Implementation_Steps/tracker-vpc-list.png?classes=shadow)

> [!NOTE]
> Danh sách VPC hiển thị VPC tùy chỉnh với CIDR 10.1.0.0/16 dành riêng cho hệ thống Tracker Maintenance

Chia các dải mạng con (subnets) thành các phân tầng cô lập: Public Subnets (chứa các thành phần tiếp xúc với Internet công cộng) và Private Subnets (chứa các giao diện mạng nội bộ của các dịch vụ tính toán để cô lập hoàn toàn IP của chúng khỏi Internet Gateway). Hệ thống được phân bổ đều trên hai Availability Zones (us-east-1a và us-east-1b) để đảm bảo tính dự phòng.

![Danh sách Subnet](/images/5-Workshop/5.3-Implementation_Steps/tracker-subnet-list.png?classes=shadow)

> [!NOTE]
> Danh sách các Subnet được phân bổ trên các phân tầng Public và Private của kiến trúc bảo trì

Cấu hình các Route Table tương ứng: Public Subnet định tuyến trực tiếp đến Internet Gateway; Private Subnet thiết lập dải định tuyến nội bộ với NAT Gateway được đặt thành None để tránh tiêu tốn chi phí duy trì tài khoản hàng tháng.

#### Bước 2: Triển khai Serverless Backend xử lý nhật ký ESP32 Tracker

Khởi tạo một hàm tính toán Serverless có tên là **Process_ESP32_Tracker_Telemetry** chạy trên môi trường Node.js. Hàm này xử lý các logic nghiệp vụ quan trọng: nhận các gói dữ liệu chẩn đoán và tình trạng pin từ các thiết bị ESP32 tracker, thực hiện phân tích lỗi phần cứng và tạo một S3 Presigned URL ngắn hạn (thời gian sống giới hạn trong 5 phút) để thiết bị có quyền tải lên các tệp nhật ký lỗi (log files) cục bộ.

![Cấu hình mã nguồn Lambda](/images/5-Workshop/5.3-Implementation_Steps/tracker-lambda-source.png?classes=shadow)

> [!NOTE]
> Giao diện cấu hình mã nguồn của hàm Lambda Process_ESP32_Tracker_Telemetry tích hợp với API Gateway trên AWS Console

Cấu hình kết nối mạng cho hàm Lambda: Di chuyển đến tab Configuration -> chọn VPC -> Kết nối trực tiếp hàm Lambda với Tracker-VPC.

Lựa chọn chính xác 2 Private Subnets để bắt buộc Lambda phải chạy hoàn toàn trong phân vùng mạng nội bộ an toàn. Gán Security Group mặc định của VPC cho hàm Lambda để quản lý các quy tắc Inbound/Outbound.

![Cấu hình Lambda VPC](/images/5-Workshop/5.3-Implementation_Steps/tracker-lambda-vpc.png?classes=shadow)

> [!NOTE]
> Cấu hình phân bổ hàm Lambda chạy trong môi trường mạng nội bộ Tracker-VPC và các Private Subnet trên AWS Console

#### Bước 3: Cấu hình quyền truy cập hệ thống trên AWS IAM

Khi cấu hình Lambda được đặt trong Private Subnet, hệ thống ban đầu sẽ báo lỗi phân quyền vì Lambda chưa có quyền tạo các giao diện mạng ảo nội bộ.

Để giải quyết vấn đề này, hãy truy cập dịch vụ IAM -> Roles -> Chọn chính xác Role thực thi hiện tại của hàm (Process_ESP32_Tracker_Telemetry-role-...).

Nhấp vào nút Add permissions -> Attach policies -> Tìm kiếm và chọn policy do AWS quản lý: AWSLambdaVPCAccessExecutionRole.

Xác nhận gán quyền để cấp cho Lambda đầy đủ các đặc quyền tạo và quản lý Elastic Network Interfaces (ENIs) nhằm truy cập dải mạng VPC.

![Gán IAM Role](/images/5-Workshop/5.3-Implementation_Steps/tracker-iam-role.png?classes=shadow)

> [!NOTE]
> Giao diện gán policy AWSLambdaVPCAccessExecutionRole cho IAM Role của hàm Lambda hệ thống bảo trì trên AWS Console

#### Bước 4: Thiết lập Bảo mật Amazon S3 lưu trữ Firmware và Log
Thiết lập kết nối mạng riêng để tối ưu hóa chi phí và bảo vệ kho lưu trữ dữ liệu thiết bị:

Khởi tạo một Amazon S3 Bucket có tên là **tracker-maintenance-storage** ở chế độ hoàn toàn riêng tư (Block Public Access) và bật mặc định Server-Side Encryption (SSE-S3) để lưu trữ tất cả các bản cập nhật firmware (OTA) và báo cáo lỗi phần cứng.

![Quản lý tệp S3](/images/5-Workshop/5.3-Implementation_Steps/tracker-s3-files.png?classes=shadow)

> [!NOTE]
> Giao diện quản lý các tệp firmware và nhật ký lỗi được lưu trữ bên trong Bucket tracker-maintenance-storage trên AWS Console

Trên trang quản lý VPC Console, tạo một Gateway VPC Endpoint cho S3 (tên dịch vụ: com.amazonaws.us-east-1.s3) và gán trực tiếp vào các Route Table của các Private Subnet trong Tracker-VPC.

Bản ghi định tuyến mới tự động được hệ thống thêm vào sẽ hướng tất cả các yêu cầu từ hàm Lambda nội bộ đến Amazon S3 thông qua mạng xương sống nội bộ của AWS thay vì định tuyến ra ngoài Internet. Cơ chế này giúp cắt giảm 100% chi phí băng thông mạng cho S3 và tăng tốc độ truyền tải các bản vá phần mềm.

Cấu hình S3 Bucket Policy nghiêm ngặt để bảo vệ tài nguyên: Từ chối tuyệt đối tất cả các yêu cầu truy cập đọc/ghi từ Internet bên ngoài và chỉ chấp nhận các yêu cầu tương tác hợp lệ đi qua Gateway VPC Endpoint mới được tạo của hệ thống.

![S3 Block Public Access](/images/5-Workshop/5.3-Implementation_Steps/tracker-s3-security.png?classes=shadow)

> [!NOTE]
> Trạng thái kích hoạt của tính năng Block Public Access bảo vệ kho lưu trữ tài liệu bảo trì S3 trên AWS Console

#### Bước 5: Cấu hình hệ thống cảnh báo bảo trì với Amazon CloudWatch & SNS
Thiết lập "mắt thần" giám sát tự động để theo dõi trạng thái ngoại tuyến hoặc lỗi cảm biến của các tracker:

Hàm Lambda nghiệp vụ tự động đẩy tất cả nhật ký chẩn đoán chi tiết vào dịch vụ Amazon CloudWatch Log Groups trong thời gian thực.

Thiết lập một Metric Filter tùy chỉnh có tên là **TrackerHardwareErrorFilter** trên Log Group của dự án để quét qua luồng nhật ký và đếm tần suất của các mã lỗi viết hoa như "HARDWARE_FAULT", "SENSOR_TIMEOUT" hoặc "BATTERY_CRITICAL".

Khởi tạo một CloudWatch Alarm kết nối trực tiếp với chỉ số (metric) của bộ lọc. Cấu hình ngưỡng cảnh báo nhạy bén: Chuyển sang trạng thái báo động khẩn cấp (ALARM) ngay khi số lượng lỗi >= 5 trong khoảng thời gian giám sát 5 phút.

Liên kết Alarm với dịch vụ Amazon SNS (Simple Notification Service). Khi trạng thái báo động kích hoạt, SNS sẽ tự động biên soạn thông báo yêu cầu bảo trì hệ thống và gửi email khẩn cấp theo thời gian thực tới hộp thư quản trị của nhà phát triển (dokat0903000@gmail.com).

![Quản lý CloudWatch Log Groups](/images/5-Workshop/5.3-Implementation_Steps/tracker-cloudwatch-logs.png?classes=shadow)

> [!NOTE]
> Giao diện quản lý CloudWatch Log Groups để giám sát nhật ký chẩn đoán phần cứng Tracker trên AWS Console