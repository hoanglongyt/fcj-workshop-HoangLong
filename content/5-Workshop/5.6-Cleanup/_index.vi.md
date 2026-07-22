---
title : "Dọn dẹp tài nguyên"
date : 2026-07-22
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

#### 1. Tổng Kết Workshop

Xin chúc mừng bạn đã hoàn thành bài thực hành xây dựng, cấu hình và bảo mật hạ tầng đám mây AWS cho hệ thống **TSL-SignMap**!

Trong workshop này, bạn đã thực hành xây dựng hệ thống tuân theo tiêu chuẩn **AWS Well-Architected Framework**:
- Triển khai phân mạng **AWS VPC 3-Tier Multi-AZ (`10.0.0.0/16`)** trên 2 Availability Zones (**AZ - A** và **AZ - B**).
- Vận hành cụm máy chủ **AWS EC2 Instances** chạy Ocelot API Gateway cùng 7 Microservices Containers và EC2 Scraper Instance.
- Định tuyến dịch vụ nội bộ với **AWS Cloud Map Service Discovery (`*.local`)**.
- Triển khai CSDL **AWS RDS for SQL Server 2022** (Primary - Standby Multi-AZ) và bộ nhớ đệm **Amazon ElastiCache (Redis)**.
- Bảo mật kết nối riêng tư với **S3 VPC Endpoint** và **SageMaker VPC Endpoint (YOLO AI)** đính kèm **VPC Endpoint Policies** chống rò rỉ dữ liệu.
- Phân phối trang web React Admin Web toàn cầu với **AWS CloudFront CDN** và **S3 Static Web**.

---

#### 2. Các Bước Dọn Dẹp Tài Nguyên (Resource Cleanup)

Để tránh phát sinh chi phí tài nguyên không cần thiết trên tài khoản AWS của bạn sau khi hoàn tất bài lab, hãy thực hiện dọn dẹp tài nguyên theo thứ tự các bước bên dưới:

##### Bước 1: Xóa VPC Endpoints
1. Mở **AWS VPC Console**, chọn **Endpoints** ở menu bên trái.
2. Chọn các endpoint đã tạo (`s3-gwe` và `sagemaker-vpce`).
3. Chọn **Actions** -> bấm **Delete endpoints** và xác nhận xóa.

##### Bước 2: Xóa Application Load Balancer (ALB)
1. Truy cập **EC2 Console**, chọn **Load Balancers**.
2. Chọn Load Balancer `TSL-SignMap-ALB`.
3. Chọn **Actions** -> bấm **Delete load balancer** và xác nhận.

##### Bước 3: Xóa các máy chủ EC2 Instances
1. Trong **EC2 Console**, chọn mục **Instances**.
2. Chọn các máy chủ `EC2 Microservices Instance` và `EC2 Scraper Instance`.
3. Chọn **Instance state** -> chọn **Terminate instance** và xác nhận xóa.

##### Bước 4: Xóa CSDL RDS SQL Server & ElastiCache Redis
1. Mở **Amazon RDS Console**, chọn **Databases**.
2. Chọn CSDL `tsl-signmap-db` (Primary/Standby).
3. Chọn **Actions** -> bấm **Delete** (bỏ chọn *Create final snapshot* nếu không cần lưu lại dữ liệu) và nhập xác nhận `delete me`.
4. Chuyển sang **Amazon ElastiCache Console**, chọn cụm Redis và bấm **Delete**.

##### Bước 5: Xóa các S3 Buckets
1. Mở **Amazon S3 Console**.
2. Chọn bộ chứa `tsl-signmap-media-bucket` và bộ chứa `S3 Static Web`.
3. Nhấn **Empty** để xóa toàn bộ tệp bên trong, sau đó chọn **Delete** và nhập tên bucket để xóa hoàn toàn.

##### Bước 6: Xóa AWS VPC & Hạ Tầng Mạng
1. Mở **AWS VPC Console**, chọn **Your VPCs**.
2. Chọn `TSL-SignMap-VPC`.
3. Chọn **Actions** -> bấm **Delete VPC** và xác nhận xóa. AWS sẽ tự động xóa VPC cùng các Subnets, Route Tables, Internet Gateway và Security Groups đi kèm.