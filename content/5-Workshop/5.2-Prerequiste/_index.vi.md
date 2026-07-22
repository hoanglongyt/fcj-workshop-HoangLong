---
title : "Chuẩn bị và Phân chia mạng VPC 3-Tier"
date : 2026-07-22 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### 1. Yêu cầu Quyền IAM (IAM Permissions)

Để triển khai toàn bộ hạ tầng hệ thống **TSL-SignMap** bao gồm VPC 3-Tier, cụm EC2 Instances, RDS SQL Server, Application Load Balancer, S3 Buckets và Secrets Manager, tài khoản AWS của bạn cần có IAM Policy với các quyền dịch vụ dưới đây:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "TSLSignMapInfrastructurePermissions",
            "Effect": "Allow",
            "Action": [
                "ec2:*",
                "rds:*",
                "s3:*",
                "elasticloadbalancing:*",
                "autoscaling:*",
                "cloudwatch:*",
                "logs:*",
                "secretsmanager:*",
                "acm:*",
                "route53:*",
                "servicediscovery:*",
                "elasticache:*",
                "sagemaker:*",
                "iam:PassRole",
                "iam:CreateRole",
                "iam:AttachRolePolicy"
            ],
            "Resource": "*"
        }
    ]
}
```

---

#### 2. Cấu hình Hạ tầng Mạng AWS VPC (VPC & Subnet Planning)

Hạ tầng mạng của hệ thống **TSL-SignMap** được triển khai tại AWS Region Singapore (`ap-southeast-1`) với dải mạng tổng **AWS VPC CIDR (`10.0.0.0/16`)** chia thành 3 tầng Subnet riêng biệt trải rộng trên 2 Availability Zones (**AZ - A** và **AZ - B**):

##### Bảng phân chia dải mạng Subnets (CIDR Block Table)

| Tầng Phân Mạng | Subnet Name | Availability Zone | Dải Mạng CIDR | Mục Đích Sử Dụng |
| :--- | :--- | :--- | :--- | :--- |
| **Public Subnet** | `Public Subnet A` | `ap-southeast-1a` (AZ-A) | `10.0.1.0/24` | Chứa ALB tiếp nhận lưu lượng HTTPS API công cộng và NAT Gateway A |
| **Public Subnet** | `Public Subnet B` | `ap-southeast-1b` (AZ-B) | `10.0.1.128/24` | Chứa ALB phụ và NAT Gateway B dự phòng Multi-AZ |
| **Private App Subnet** | `Private Subnet A` | `ap-southeast-1a` (AZ-A) | `10.0.2.0/24` | Chứa EC2 Ocelot API Gateway, 7 Microservices Containers & EC2 Scraper Instance |
| **Private App Subnet** | `Private Subnet B` | `ap-southeast-1b` (AZ-B) | `10.0.2.128/24` | Chứa cụm EC2 Ocelot API Gateway + Microservices dự phòng |
| **Private DB Subnet** | `Private DB Sub A` | `ap-southeast-1a` (AZ-A) | `10.0.3.0/24` | Chứa CSDL AWS RDS Primary for SQL Server 2022 (Port 1433) |
| **Private DB Subnet** | `Private DB Sub B` | `ap-southeast-1b` (AZ-B) | `10.0.3.128/24` | Chứa AWS RDS Standby SQL Server (Multi-AZ synchronous replication) & ElastiCache (Redis) |

---

#### 3. Quy trình Triển khai Hạ tầng Mạng và Bảo mật

##### Bước 1: Khởi tạo AWS VPC & Gateway
1. Truy cập **AWS VPC Console** chọn **Create VPC**.
2. Chọn **VPC and more**, nhập **Name tag**: `TSL-SignMap-VPC`.
3. Nhập dải IP **IPv4 CIDR block**: `10.0.0.0/16`.
4. Chọn **Number of Availability Zones (AZs)**: `2` (`ap-southeast-1a` và `ap-southeast-1b`).
5. Chọn **Number of Public subnets**: `2`.
6. Chọn **Number of Private subnets**: `4` (2 App Subnets + 2 DB Subnets).
7. Đánh dấu chọn tạo **NAT Gateways** (In 2 AZs) và **VPC Endpoints** cho kết nối riêng tư.

##### Bước 2: Thiết lập Security Groups (Tường lửa cho dịch vụ)
- **`ALB-Security-Group`**: Cho phép traffic inbound **HTTPS (Port 443)** và **HTTP (Port 80)** từ `0.0.0.0/0`.
- **`EC2-App-Security-Group`**: Chỉ cho phép traffic inbound **Port 5008 (Ocelot API Gateway)** từ `ALB-Security-Group` và giao tiếp nội bộ giữa 7 Microservices.
- **`RDS-DB-Security-Group`**: Chỉ cho phép traffic inbound **Port 1433 (SQL Server)** và **Port 6379 (Redis)** từ `EC2-App-Security-Group`.