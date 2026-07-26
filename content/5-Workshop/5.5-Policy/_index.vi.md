---
title : "Dọn dẹp tài nguyên"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---
### Thu hồi và Dọn dẹp Tài nguyên
Sau khi hoàn tất các kịch bản kiểm thử thực tế cho luồng chẩn đoán và bảo trì thiết bị định vị ESP32, toàn bộ tài nguyên hạ tầng đám mây thử nghiệm đã được tiến hành thu hồi nhằm tối ưu hóa ngân sách và triệt tiêu nguy cơ phát sinh cước phí ngoài ý muốn trên tài khoản AWS:

* **Làm sạch không gian lưu trữ S3**: Xóa bỏ vĩnh viễn toàn bộ các tệp nhật ký lỗi phần cứng (crash logs) và các bản vá phần mềm (firmware OTA) thử nghiệm bên trong bucket `tracker-maintenance-storage` để giải phóng dung lượng.
* **Vô hiệu hóa VPC Endpoints**: Tiến hành gỡ bỏ Gateway VPC Endpoint chuyên dụng cho S3, đồng thời xóa các bản ghi định tuyến nội bộ khỏi Route Table của các Private Subnet thuộc `Tracker-VPC` để đưa cấu hình mạng về trạng thái gốc.
* **Hủy hệ thống Giám sát & Báo động**: Xóa bộ lọc chỉ số `TrackerHardwareErrorFilter` và gỡ bỏ các Alarm cảnh báo lỗi cảm biến trên dịch vụ Amazon CloudWatch. Đồng thời, hủy chủ đề (Topic) trên Amazon SNS để ngắt luồng gửi email tự động điều phối bảo trì.