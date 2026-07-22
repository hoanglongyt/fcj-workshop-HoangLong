---
title : "Triển khai cụm EC2 Microservices, Ocelot API Gateway & AWS Cloud Map"
date : 2026-07-22 
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---

#### 1. Tổng quan cụm EC2 Microservices & Endpoints Theo Sơ Đồ Kiến Trúc

Dựa trên sơ đồ kiến trúc **`TSLSignMap.drawio.png`**, cụm 8 Microservices và các máy chủ điện toán được thiết kế đảm bảo khả năng cân bằng tải, mở rộng tự động và kết nối riêng tư:

- **ALB (6) & Target Group -> Auto Scaling Group:** Luồng kết nối HTTPS API `/api/*` (4) đi từ CloudFront/Route 53 qua Internet Gateway (5) tới **ALB (6)**. ALB phân phối lưu lượng tới **Target Group** điều hướng vào cụm máy chủ **`EC2 Ocelot API + 7 Containers`** nằm trong **Auto Scaling Group** spanning qua 2 vùng **AZ - A (Private subnet A)** và **AZ - B (Private subnet B)**.
- **Endpoint SageMaker (9):** Kết nối trực tiếp mô hình AI **SageMaker (YOLO AI)** qua điểm cuối riêng tư **`Endpoint (SageMaker) (9)`** vào `EC2 Ocelot API + 7 Containers` để thực hiện nhận diện biển báo qua hình ảnh.
- **Endpoint S3 (10):** Kết nối cụm microservices với **`S3 (Media Bucket)`** thông qua điểm cuối riêng tư **`EndPoint - S3 (10)`** để lưu trữ và truy xuất các tệp ảnh biển báo người dùng tải lên.
- **EC2 Scraper Instance (11):** Đặt tại **AZ - A (Private subnet A)**, vận hành script `scrape_signs.py` theo lịch Cronjob:
  - Cào dữ liệu biển báo từ **OpenStreetMap API** qua **NAT Gateway** (tại Public Subnet A).
  - Cập nhật trực tiếp dữ liệu cào được vào **RDS Primary - SQL (7)**.
  - Cập nhật bộ nhớ đệm vào cụm **Amazon ElastiCache (Redis)**.

---

#### 2. Quy Trình Triển Khai Chi Tiết Theo Sơ Đồ

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
Cấu hình định tuyến từ ALB (6) (Port 5008) đến các microservices nội bộ:
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

##### Bước 3: Triển khai Docker Containers trên EC2 Auto Scaling Group
1. Khởi tạo máy chủ **EC2 Instances** trong Private Subnet A (AZ-A) và Private Subnet B (AZ-B).
2. Đăng nhập Docker Registry **AWS ECR** và pull các Container Images:
   ```bash
   aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.ap-southeast-1.amazonaws.com
   docker pull <aws_account_id>.dkr.ecr.ap-southeast-1.amazonaws.com/tsl-trafficsign-service:latest
   ```
3. Khởi chạy cụm 8 microservices containers với Docker Compose hoặc Docker CLI:
   ```bash
   docker run -d --name trafficsignservice -p 5002:5002 -e "ConnectionStrings__SQLServer=Server=tsl-signmap-db.c123456789.ap-southeast-1.rds.amazonaws.com;Database=TSLSignMapDB;User Id=tslsignadmin;Password=<secret_password>;" <aws_account_id>.dkr.ecr.ap-southeast-1.amazonaws.com/tsl-trafficsign-service:latest
   ```

##### Bước 4: Khởi chạy EC2 Scraper Instance (11)
Khởi chạy EC2 Scraper Instance (11) tại Private Subnet A cào dữ liệu từ OpenStreetMap API:
```bash
python3 scrape_signs.py --province "HoChiMinh" --batch-size 500
```

---

#### 3. Kiểm Tra Luồng Dữ Liệu và Health Check API

1. Truy cập Health Check endpoint của Ocelot API Gateway từ ALB (6):
   ```bash
   curl -I https://api.tsl-signmap.com/api/trafficsigns/health
   ```
   *Kết quả:* Trả về HTTP Status `200 OK`.

2. Thử nghiệm tra cứu vị trí biển báo GIS địa lý SRID 4326 qua Endpoint SageMaker (9) và RDS Primary (7):
   ```bash
   curl -X GET "https://api.tsl-signmap.com/api/trafficsigns/nearby?lat=10.7769&lng=106.7009&radius=500"
   ```
   *Kết quả:* Trả về danh sách JSON chứa các biển báo giao thông được cache bởi Redis và truy vấn từ RDS Primary SQL (7).