---
title: "Workshop"
date: 2026-07-22
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

#### Thiết kế và Triển khai Hạ tầng AWS cho Hệ thống TSL-SignMap

#### Tổng quan

**TSL-SignMap** là hệ thống quản lý, đóng góp và tra cứu thông tin biển báo giao thông không gian GIS (SRID 4326), được vận hành trên hạ tầng đám mây **AWS** tuân theo tiêu chuẩn **AWS Well-Architected Framework**. Hệ thống được thiết kế với tính sẵn sàng cao (High Availability), khả năng mở rộng linh hoạt (Scalability) và bảo mật nhiều lớp (Defense-in-Depth).

Trong workshop này, chúng ta sẽ thực hành xây dựng, cấu hình và tối ưu hóa hạ tầng AWS toàn diện cho TSL-SignMap với các khối kiến trúc chính:

+ **Tầng Edge & Bảo mật (Global Edge Layer):** Sử dụng **Amazon Route 53**, **AWS CloudFront (CDN)** kết hợp **AWS WAF** và **AWS Certificate Manager (ACM)** giúp tăng tốc độ phân phối trang React Admin Web, bảo vệ chống tấn công web và quản lý chứng chỉ SSL/TLS.
+ **Tầng Mạng VPC 3-Tier Multi-AZ:** Thiết lập AWS VPC phân chia theo 2 Availability Zones (AZ-A & AZ-B) gồm 3 phân vùng subnet:
  - **Public Subnet:** Chứa **Application Load Balancer (ALB)** tiếp nhận lưu lượng HTTPS và các **NAT Gateway** cho truy cập Internet chiều ra.
  - **Private App Subnet:** Chứa cụm **Auto Scaling Group** vận hành **Ocelot API Gateway** cùng **7 Microservices Containers** và **SageMaker AI Endpoint** kết nối đến mô hình nhận diện biển báo YOLO AI.
  - **Private DB Subnet:** Chứa cơ sở dữ liệu **AWS RDS for SQL Server 2022** mô hình Primary - Standby (Multi-AZ replication) và cụm bộ nhớ đệm **Amazon ElastiCache (Redis)**.
+ **Dịch vụ Tích hợp & Kết nối Riêng tư (VPC Endpoints & Storage):**
  - **S3 VPC Endpoint (Gateway/Interface):** Cho phép các Microservices truy cập an toàn đến **S3 Media Bucket** (chứa ảnh biển báo) mà không đi qua Internet công cộng.
  - **SageMaker VPC Endpoint:** Cho phép API Gateway gọi các endpoint AI SageMaker hoàn toàn trong mạng nội bộ.
  - **EC2 Scraper Instance:** Tự động cào dữ liệu từ OpenStreetMap API và cập nhật vào RDS SQL Server.
+ **Sao lưu & Phục hồi Thảm họa (Disaster Recovery & Backup):** Tự động đồng bộ dữ liệu sang **Secondary Disaster Recovery Region** thông qua AWS Backup, RDS Backup và S3 Cross-Region Replication.
+ **Quản trị & Giám sát (Governance & Monitoring):** Tích hợp **AWS Secrets Manager**, **AWS IAM**, **Amazon ECR** và **Amazon CloudWatch** để giám sát và quản lý hạ tầng tập trung.

#### Nội dung

1. [Tổng quan hệ thống TSL-SignMap và Hạ tầng AWS](5.1-Workshop-overview/)
2. [Chuẩn bị và Phân chia mạng VPC 3-Tier](5.2-Prerequiste/)
3. [Thiết lập cụm Microservices và Cơ sở dữ liệu RDS](5.3-S3-vpc/)
4. [Cấu hình AWS CloudFront và S3 Static Web](5.4-S3-onprem/)
5. [Cấu hình VPC Endpoints (S3 & SageMaker) và Policies](5.5-Policy/)
6. [Dọn dẹp tài nguyên](5.6-Cleanup/)