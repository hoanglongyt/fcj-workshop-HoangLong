---
title : "Triển khai cụm EC2 Microservices, Ocelot API Gateway & AWS Cloud Map"
date : 2026-07-22 
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---

#### 1. Tổng quan cụm EC2 Microservices và Service Discovery

Trong phần này, chúng ta sẽ triển khai cụm 8 Microservices trên các máy chủ **AWS EC2 Instances** trong **Private App Subnet** (`10.0.2.0/24`) kết hợp dịch vụ định tuyến tên **AWS Cloud Map Service Discovery (`*.local`)** và cấu hình **Ocelot API Gateway**.

- **AWS EC2 Microservices Instances:** Triển khai các Docker Containers bao gồm `Ocelot API Gateway (Port 5008)` và 7 dịch vụ nghiệp vụ (`UserService:5001`, `TrafficSignService:5002`, `ContributionService:5003`, `FeedbackService:5004`, `PaymentService:5005`, `RewardService:5006`, `NotificationService:5007`).
- **AWS Cloud Map (Service Discovery):** Khởi tạo namespace DNS riêng tư **`tslsignmap.local`** giúp Ocelot API Gateway điều hướng linh hoạt request đến các IP máy chủ nội bộ.
- **EC2 Scraper Instance:** Máy chủ chuyên biệt chạy script Python `scrape_signs.py` được tự động kích hoạt bởi lịch **AWS EventBridge** (2h sáng hàng ngày) để thu thập dữ liệu biển báo từ OpenStreetMap API và cập nhật vào RDS SQL Server.

---

#### 2. Quy Trình Triển Khai Chi Tiết

##### Bước 1: Khởi tạo Private DNS Namespace trên AWS Cloud Map
1. Mở **AWS Cloud Map Console**, bấm **Create namespace**.
2. Namespace name: Nhập `tslsignmap.local`.
3. Instance discovery: Chọn **API calls and DNS queries in VPCs**.
4. VPC: Chọn `TSL-SignMap-VPC`.
5. Đăng ký 8 Services:
   - `gateway.tslsignmap.local` (Port 5008)
   - `user.tslsignmap.local` (Port 5001)
   - `trafficsign.tslsignmap.local` (Port 5002)
   - `contribution.tslsignmap.local` (Port 5003)
   - `feedback.tslsignmap.local` (Port 5004)
   - `payment.tslsignmap.local` (Port 5005)
   - `reward.tslsignmap.local` (Port 5006)
   - `notification.tslsignmap.local` (Port 5007)

##### Bước 2: Cấu hình Ocelot API Gateway (`ocelot.json`)
Cấu hình định tuyến từ ALB (Port 5008) đến các microservices nội bộ:
```json
{
  "Routes": [
    {
      "DownstreamPathTemplate": "/api/v1/trafficsigns/{everything}",
      "DownstreamScheme": "http",
      "DownstreamHostAndPorts": [
        {
          "Host": "trafficsign.tslsignmap.local",
          "Port": 5002
        }
      ],
      "UpstreamPathTemplate": "/api/trafficsigns/{everything}",
      "UpstreamHttpMethod": [ "Get", "Post", "Put", "Delete" ]
    },
    {
      "DownstreamPathTemplate": "/api/v1/contributions/{everything}",
      "DownstreamScheme": "http",
      "DownstreamHostAndPorts": [
        {
          "Host": "contribution.tslsignmap.local",
          "Port": 5003
        }
      ],
      "UpstreamPathTemplate": "/api/contributions/{everything}",
      "UpstreamHttpMethod": [ "Get", "Post" ]
    }
  ]
}
```

##### Bước 3: Triển khai Docker Containers trên máy chủ EC2
1. Khởi tạo máy chủ **EC2 Instances** trong Private App Subnet.
2. Đăng nhập Docker Registry **AWS ECR** và pull các Container Images:
   ```bash
   aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.ap-southeast-1.amazonaws.com
   docker pull <aws_account_id>.dkr.ecr.ap-southeast-1.amazonaws.com/tsl-trafficsign-service:latest
   ```
3. Khởi chạy cụm 8 microservices containers với Docker Compose hoặc Docker CLI:
   ```bash
   docker run -d --name trafficsignservice -p 5002:5002 -e "ConnectionStrings__SQLServer=Server=tsl-signmap-db.c123456789.ap-southeast-1.rds.amazonaws.com;Database=TSLSignMapDB;User Id=tslsignadmin;Password=<secret_password>;" <aws_account_id>.dkr.ecr.ap-southeast-1.amazonaws.com/tsl-trafficsign-service:latest
   ```

##### Bước 4: Khởi chạy EC2 Scraper Task (`scrape_signs.py`)
Khởi chạy EC2 Scraper Instance thu thập dữ liệu từ OpenStreetMap API:
```bash
python3 scrape_signs.py --province "HoChiMinh" --batch-size 500
```

---

#### 3. Kiểm Tra Luồng Dữ Liệu và Health Check API

1. Truy cập Health Check endpoint của Ocelot API Gateway từ ALB:
   ```bash
   curl -I https://api.tsl-signmap.com/api/trafficsigns/health
   ```
   *Kết quả:* Trả về HTTP Status `200 OK`.

2. Thử nghiệm tra cứu vị trí biển báo GIS địa lý SRID 4326:
   ```bash
   curl -X GET "https://api.tsl-signmap.com/api/trafficsigns/nearby?lat=10.7769&lng=106.7009&radius=500"
   ```
   *Kết quả:* Trả về danh sách JSON chứa các biển báo giao thông được cache bởi Redis và truy vấn từ RDS SQL Server 2022.