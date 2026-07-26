---
title: "Blog 2"
date: 2026-07-26
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# TỰ ĐỘNG CHUYỂN ĐỔI ẢNH 2D THÀNH ASSET GAME 3D BẰNG OPEN-SOURCE MODEL TRÊN AWS EC2

Chào các anh chị và các bạn,

Nếu từng làm game, chắc hẳn bạn cũng biết thử thách lớn nhất khi tạo asset 3D không chỉ nằm ở khâu vẽ shape, mà là giải đồng thời cả 3 bài toán: hình học (geometry), chất liệu (texture) và tính nhất quán vật lý (physical consistency). So với tạo ảnh 2D bằng AI, việc sinh asset 3D tốn thời gian hơn rất nhiều nếu làm thủ công.

Hôm nay mình muốn chia sẻ bài viết thử nghiệm cách chuyển đổi ảnh 2D thành model 3D bằng các mã nguồn mở (TripoSG và MV-Adapter) chạy trực tiếp trên máy chủ Amazon EC2 GPU. Đây là một pattern rất thích hợp cho các game studio muốn dựng môi trường sandbox thử nghiệm chất lượng asset (PoC) trước khi đưa vào sản xuất.

## Bối Cảnh & Mục Tiêu Thử Nghiệm

Thay vì dùng các dịch vụ SaaS trả phí khép kín, việc tự triển khai open-source model trên EC2 mang lại khả năng tùy biến sâu ở tầng hạ tầng.

- **Đầu vào:** Ảnh concept art 2D (định dạng JPG/PNG)
- **Đầu ra:** Asset 3D dạng file `.glb` đã được dựng lưới (mesh) và phủ texture đa góc nhìn
- **Hạ tầng:** Chạy trên EC2 instances dòng GPU (G4dn và G6e) kết hợp lưu trữ Amazon S3

## Những Điểm Nổi Bật Trong Quy Trình

- **Dựng Lưới (Geometry Generation) với TripoSG:** Sử dụng model TripoSG chạy trên instance `g4dn.2xlarge` (NVIDIA T4) để chuyển ảnh 2D thành mesh 3D dạng `.glb`. Bạn có thể tùy chỉnh tham số `--faces` để kiểm soát polygon count phù hợp với game low-poly hay high-poly.
- **Tối Ưu Texture Đa Góc Nhìn với MV-Adapter:** Để bề mặt model nét và đồng nhất ở mọi góc nhìn, pipeline dùng thêm MV-Adapter trên instance `g6e.2xlarge` (vRAM lớn hơn). Tool này lấy chính ảnh 2D ban đầu làm tham chiếu để áp texture lên model 3D.
- **Xử Lý Lỗi Mesh Non-Manifold:** Một điểm thực tế rất hay: mesh sinh ra từ AI thường bị lỗi non-manifold (lưới không kín hoặc bị chồng lấp). Quy trình này bổ sung một bước chuyển đổi (Fix Manifold) qua PyMeshLab trước khi đưa vào khâu phủ texture, tránh việc crash tool ở bước sau.

## Luồng Xử Lý Dữ Liệu (Pipeline)

```text
Ảnh Concept 2D ──> Lưu trữ trên Amazon S3
                      │
                      ▼
[EC2 g4dn.2xlarge] TripoSG Model
  └─> Sinh file 3D Mesh (.glb) thô
                      │
                      ▼
[EC2 g6e.2xlarge] MV-Adapter Pipeline
  ├─> Bước 1: Fix Manifold Mesh (PyMeshLab)
  ├─> Bước 2: Phủ Multi-view Texture từ ảnh 2D
  └─> Xuất file 3D hoàn chỉnh (.glb)
                      │
                      ▼
Tải asset hoàn thiện về Amazon S3

```
Các bước triển khai tóm tắt:

Chuẩn bị dữ liệu: Tải ảnh concept 2D lên Amazon S3 bucket.
Sinh hình khối (3D Mesh): Kết nối EC2 g4dn.2xlarge, kích hoạt môi trường PyTorch 2.5.1 (CUDA 12.4), clone repo TripoSG và chạy lệnh sinh file .glb từ ảnh S3.
Phủ Texture & Khắc phục lỗi lưới: Kết nối EC2 g6e.2xlarge, dùng script Python chuyển đổi mesh sang manifold object, sau đó chạy MV-Adapter để mapping texture từ ảnh gốc 2D lên model 3D.
Lưu trữ: Đẩy file .glb hoàn chỉnh ngược lại S3 để các 3D artist hoặc game dev tải về import vào Unity/Unreal Engine.
Đánh Giá Thực Tế

Ưu điểm: Tận dụng hạ tầng GPU linh hoạt của AWS, chủ động điều chỉnh được tham số polygon count và hoàn toàn kiểm soát mã nguồn model. Dễ dàng tự động hóa thành một script pipeline cho team Art.

Lưu ý: Việc cài đặt phụ thuộc khá nhiều vào phiên bản CUDA và thư viện Python (như PyMeshLab, Jaxtyping). Bạn nên đóng gói các bước này thành Docker container hoặc lưu lại dưới dạng custom AMI để tiết kiệm thời gian cho các lần thử nghiệm sau.

Kết Luận

Tuy asset tạo ra chưa thể dùng ngay cho các dự án AAA đòi hỏi tối ưu khắt khe, nhưng workflow này giúp rút ngắn đáng kể thời gian chuyển đổi từ ý tưởng 2D sang bản prototype 3D. Đây là bước khởi đầu rất tốt cho các indie game dev hoặc studio muốn ứng dụng Generative AI vào quy trình sản xuất game của mình.

Tài liệu tham khảo
AWS Games Blog – Open source 3D game asset generation using AWS
TripoSG Repository: https://github.com/VAST-AI-Research/TripoSG
MV-Adapter Repository: https://github.com/huanngzh/MV-Adapter
<img src="/images/Blogs/blog2.png" alt="Blog 2" width="1000" />