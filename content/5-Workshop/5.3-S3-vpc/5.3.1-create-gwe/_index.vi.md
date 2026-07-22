---
title : "Triển khai Cơ sở dữ liệu AWS RDS SQL Server & ElastiCache Redis"
date : 2026-07-22 
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

#### 1. Tổng quan CSDL RDS SQL Server 2022 và ElastiCache Redis Theo Sơ Đồ Kiến Trúc

Theo sơ đồ kiến trúc **`TSLSignMap.drawio.png`**, hệ thống **TSL-SignMap** quản lý dữ liệu không gian biển báo giao thông GIS (chuẩn địa lý SRID 4326 với hơn 1,286+ đối tượng địa lý), thông tin người dùng, giao dịch và phản hồi. Toàn bộ tầng CSDL và bộ nhớ đệm được bố trí chi tiết như sau:

- **RDS Primary - SQL (7):** Khởi tạo tại **AZ - A (Private DB Sub A)** (`10.0.3.0/24`), tiếp nhận trực tiếp các truy vấn đọc/ghi từ cụm `EC2 Ocelot API + 7 Containers` và máy chủ `EC2 Scraper Instance (11)`.
- **RDS Standby (8):** Khởi tạo tại **AZ - B (Private DB Sub B)** (`10.0.3.128/24`), liên kết trực tiếp với RDS Primary qua luồng đồng bộ dữ liệu thời gian thực **Multi-AZ Synchronous Replication (kết nối giữa 7 và 8)** giúp sẵn sàng tự động chuyển vùng khi có sự cố.
- **Amazon ElastiCache (Redis):** Đặt tại **Private DB Sub B**, kết nối trực tiếp từ `EC2 Scraper Instance (11)` và các máy chủ microservices để lưu đệm các truy vấn địa lý biển báo phổ biến, tối ưu thời gian phản hồi API dưới `50ms`.
- **Sao lưu thảm họa (Secondary Disaster Recovery Region) (12):** Dữ liệu bản sao snapshot từ RDS Primary (7) và RDS Standby (8) được tự động chuyển tiếp và lưu trữ an toàn sang vùng DR phụ (12).

---

#### 2. Quy Trình Triển Khai Chi Tiết Theo Sơ Đồ

##### Bước 1: Khởi tạo DB Subnet Group Cho Multi-AZ
1. Mở **Amazon RDS Console**, chọn mục **Subnet groups** ở menu bên trái.
2. Bấm **Create DB Subnet Group**, nhập tên `tsl-signmap-db-subnet-group`.
3. Chọn VPC `TSL-SignMap-VPC`.
4. Thêm các phân vùng Subnet: Chọn 2 Availability Zones (`ap-southeast-1a` cho AZ-A và `ap-southeast-1b` cho AZ-B) và chọn 2 subnets thuộc tầng DB: `Private DB Sub A` (`10.0.3.0/24`) và `Private DB Sub B` (`10.0.3.128/24`).
5. Bấm **Create**.

##### Bước 2: Khởi tạo CSDL RDS Primary (7) & RDS Standby (8)
1. Tại RDS Console, chọn **Databases** -> bấm **Create database**.
2. Chọn phương thức **Standard create**.
3. Engine options: Chọn **Microsoft SQL Server**, phiên bản **SQL Server 2022 Standard Edition**.
4. Deployment options: Chọn **Multi-AZ DB Instance** (Khởi tạo RDS Primary tại AZ-A (7) và bản sao RDS Standby tại AZ-B (8)).
5. Settings:
   - DB instance identifier: `tsl-signmap-db`.
   - Master username: `tslsignadmin`.
   - Master password: Nhập mật khẩu quản trị bảo mật (được mã hóa lưu tại AWS Secrets Manager).
6. Instance configuration: Chọn **db.m6i.xlarge** (4 vCPU, 16 GiB RAM).
7. Storage: Autoscaling Storage lên tới `500 GiB`.
8. Connectivity:
   - Virtual private cloud (VPC): Chọn `TSL-SignMap-VPC`.
   - DB subnet group: Chọn `tsl-signmap-db-subnet-group`.
   - Public access: Chọn **No** (Hoàn toàn ẩn trong Private DB Subnet).
   - Existing VPC security groups: Chọn `RDS-DB-Security-Group` (chỉ mở Port 1433 từ `EC2-App-Security-Group`).
9. Bấm **Create database**.

##### Bước 3: Triển khai Cụm Amazon ElastiCache (Redis)
1. Mở **Amazon ElastiCache Console**, chọn **Redis clusters** -> bấm **Create Redis cluster**.
2. Location: Chọn **AWS Cloud**, cluster mode **Enabled**.
3. Cluster info: Nhập tên `tsl-signmap-redis-cache`.
4. Node type: Chọn `cache.t4g.small`.
5. Subnet group: Chọn subnet group tương ứng với `Private DB Subnet`.
6. Security group: Cho phép truy cập cổng `6379` từ `EC2-App-Security-Group` và `EC2 Scraper Instance (11)`.
7. Bấm **Create**.

---

#### 3. Kiểm Tra Trạng Thái & Khởi Tạo Schema GIS SRID 4326

1. Từ máy chủ `EC2 Microservices Instance` trong Private App Subnet, chạy lệnh kiểm tra kết nối tới RDS Primary (7) qua cổng 1433:
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
