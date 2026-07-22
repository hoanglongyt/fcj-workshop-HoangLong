---
title : "Triển khai Cơ sở dữ liệu AWS RDS SQL Server & ElastiCache Redis"
date : 2026-07-22 
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

#### 1. Tổng quan CSDL RDS SQL Server 2022 và ElastiCache Redis

Hệ thống **TSL-SignMap** quản lý dữ liệu không gian biển báo giao thông GIS (chuẩn địa lý SRID 4326 với hơn 1,286+ đối tượng địa lý), thông tin người dùng, các giao dịch đóng góp và phản hồi. Toàn bộ dữ liệu cốt lõi này được lưu trữ trong CSDL **AWS RDS for SQL Server 2022** kết hợp cụm bộ nhớ đệm **Amazon ElastiCache (Redis)**.

- **AWS RDS for SQL Server 2022:** Triển khai tại **Private DB Subnet** (`10.0.3.0/24`) mô hình **Multi-AZ Deployment** (RDS Primary đặt tại `Private DB Sub A` / RDS Standby đặt tại `Private DB Sub B`). Mô hình này đảm bảo tính sẵn sàng cao (High Availability), tự động đồng bộ dữ liệu (Synchronous Replication) và tự động chuyển vùng khi có sự cố.
- **Amazon ElastiCache for Redis:** Cụm bộ nhớ đệm Redis (Port 6379) lưu trữ tạm thời các kết quả tra cứu địa lý biển báo phổ biến, giúp giảm tải cho CSDL SQL Server và tối ưu thời gian phản hồi API xuống dưới `50ms`.

---

#### 2. Quy Trình Triển Khai Chi Tiết

##### Bước 1: Khởi tạo DB Subnet Group
1. Mở **Amazon RDS Console**, chọn mục **Subnet groups** ở menu bên trái.
2. Bấm **Create DB Subnet Group**, nhập tên `tsl-signmap-db-subnet-group`.
3. Chọn VPC `TSL-SignMap-VPC`.
4. Thêm các phân vùng Subnet: Chọn 2 Availability Zones (`ap-southeast-1a` và `ap-southeast-1b`) và chọn 2 subnets thuộc tầng DB: `Private DB Sub A` (`10.0.3.0/24`) và `Private DB Sub B` (`10.0.3.128/24`).
5. Bấm **Create**.

##### Bước 2: Khởi tạo Cơ sở dữ liệu AWS RDS for SQL Server 2022
1. Tại RDS Console, chọn **Databases** -> bấm **Create database**.
2. Chọn phương thức **Standard create**.
3. Engine options: Chọn **Microsoft SQL Server**, phiên bản **SQL Server 2022 Standard Edition**.
4. Deployment options: Chọn **Multi-AZ DB Instance** (Tạo bản sao dự phòng Standby ở AZ-B).
5. Settings:
   - DB instance identifier: `tsl-signmap-db`.
   - Master username: `tslsignadmin`.
   - Master password: Nhập mật khẩu quản trị bảo mật (được mã hóa lưu tại AWS Secrets Manager).
6. Instance configuration: Chọn **db.m6i.xlarge** (4 vCPU, 16 GiB RAM).
7. Storage: Autoscaling Storage lên tới `500 GiB`.
8. Connectivity:
   - Virtual private cloud (VPC): Chọn `TSL-SignMap-VPC`.
   - DB subnet group: Chọn `tsl-signmap-db-subnet-group`.
   - Public access: Chọn **No** (Hoàn toàn ẩn trong mạng riêng tư).
   - Existing VPC security groups: Chọn `RDS-DB-Security-Group` (chỉ mở Port 1433 từ `EC2-App-Security-Group`).
9. Bấm **Create database**.

##### Bước 3: Triển khai Cụm Amazon ElastiCache (Redis)
1. Mở **Amazon ElastiCache Console**, chọn **Redis clusters** -> bấm **Create Redis cluster**.
2. Location: Chọn **AWS Cloud**, cluster mode **Enabled**.
3. Cluster info: Nhập tên `tsl-signmap-redis-cache`.
4. Node type: Chọn `cache.t4g.small`.
5. Subnet group: Chọn subnet group tương ứng với `Private DB Subnet`.
6. Security group: Cho phép truy cập cổng `6379` từ `EC2-App-Security-Group`.
7. Bấm **Create**.

---

#### 3. Kiểm Tra Trạng Thái & Khởi Tạo Schema GIS SRID 4326

1. Từ máy chủ `EC2 Microservices Instance` trong Private App Subnet, chạy lệnh kiểm tra kết nối qua cổng 1433:
   ```bash
   nc -zv tsl-signmap-db.c123456789.ap-southeast-1.rds.amazonaws.com 1433
   ```
2. Thực thi script SQL khởi tạo bảng CSDL và chỉ mục địa lý Spatial Index:
   ```sql
   CREATE TABLE TrafficSigns (
       Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
       SignCode VARCHAR(50) NOT NULL,
       SignName NVARCHAR(255) NOT NULL,
       Location GEOGRAPHY NOT NULL, -- Spatial SRID 4326
       CreatedAt DATETIME2 DEFAULT GETDATE()
   );
   
   CREATE SPATIAL INDEX SX_TrafficSigns_Location ON TrafficSigns(Location);
   ```
