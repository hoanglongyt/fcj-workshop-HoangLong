---
title : "Thiết lập cụm Microservices và Cơ sở dữ liệu RDS"
date : 2026-07-22 
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### 1. Tổng quan Tầng Ứng Dụng Microservices và CSDL RDS

Trong phần này, chúng ta sẽ thực hành triển khai cụm 8 Microservices trên các máy chủ **AWS EC2 Instances** nằm trong **Private App Subnet** và cấu hình CSDL **AWS RDS for SQL Server 2022** cùng cụm **Amazon ElastiCache (Redis)** tại **Private DB Subnet**.

Hạ tầng ứng dụng được thiết kế đảm bảo các dịch vụ hoạt động an toàn hoàn toàn trong mạng nội bộ VPC, giao tiếp với nhau qua **AWS Cloud Map Service Discovery (`*.local`)** và chỉ tiếp nhận kết nối API ngoại vi thông qua **Ocelot API Gateway (Port 5008)** nối từ **Application Load Balancer (ALB)**.

---

#### 2. Chi Tiết Các Thành Phần Cụm 8 Microservices & CSDL

| Tên Dịch Vụ | Cổng (Port) | Vị Trí Subnet | Chức Năng Chi Tiết |
| :--- | :--- | :--- | :--- |
| **ApiGateway Container** | `5008` | Private App Subnet | Ocelot API Gateway tiếp nhận và điều hướng HTTPS request `/api/*` từ ALB đến các dịch vụ nội bộ |
| **UserService** | `5001` | Private App Subnet | Quản lý người dùng, xác thực token JWT và quản lý phân quyền |
| **TrafficSignService** | `5002` | Private App Subnet | Quản lý & tra cứu dữ liệu không gian biển báo GIS (chuẩn địa lý SRID 4326) |
| **ContributionService** | `5003` | Private App Subnet | Tiếp nhận người dùng đóng góp biển báo mới & tải ảnh trực tiếp lên S3 Media Bucket |
| **FeedbackService** | `5004` | Private App Subnet | Xử lý phản hồi, báo cáo lỗi thông tin biển báo từ cộng đồng |
| **PaymentService** | `5005` | Private App Subnet | Xử lý các giao dịch thanh toán và tích hợp cổng thanh toán |
| **RewardService** | `5006` | Private App Subnet | Quản lý điểm thưởng và đổi quà cho người dùng đóng góp |
| **NotificationService** | `5007` | Private App Subnet | Gửi thông báo real-time và cập nhật trạng thái hệ thống |
| **EC2 Scraper Instance** | Cron Task | Private App Subnet | Chạy script `scrape_signs.py` cào tự động dữ liệu 15 tỉnh thành từ OpenStreetMap API |
| **AWS RDS SQL Server 2022** | `1433` | Private DB Subnet | CSDL SQL Server 2022 lưu trữ 1,286+ biển báo GIS geography, dữ liệu user và giao dịch (Primary - Standby Multi-AZ) |
| **Amazon ElastiCache** | `6379` | Private DB Subnet | Cụm bộ nhớ đệm Redis giúp tối ưu tốc độ đọc/ghi dữ liệu biển báo tra cứu |

---

#### 3. Nội Dung Thực Hành Chi Tiết

- [5.3.1 Triển khai Cơ sở dữ liệu AWS RDS SQL Server & ElastiCache Redis](5.3.1-create-gwe/)
- [5.3.2 Triển khai cụm EC2 Microservices, Ocelot API Gateway & AWS Cloud Map](5.3.2-test-gwe/)