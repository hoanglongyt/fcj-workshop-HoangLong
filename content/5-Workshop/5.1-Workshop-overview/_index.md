---
title : "Introduction"
date : 2026-07-22 
weight : 1 
chapter : false
pre : " <b> 5.1. </b> "
---

#### 1. TSL-SignMap System Overview

**TSL-SignMap** is a spatial GIS traffic sign management, contribution, and lookup application (SRID 4326 spatial standard) operating on AWS cloud infrastructure featuring a **3-Tier Multi-AZ VPC**, **AWS EC2 Instances Architecture**, **Amazon SageMaker (YOLO AI)** integration, and **AWS RDS for SQL Server 2022**.

The system provides centralized traffic sign management, enabling users to search sign locations, contribute new traffic signs, utilize AI image-based sign recognition, and automatically sync sign data from OpenStreetMap.

---

#### 2. Key AWS Services in the Infrastructure

| No. | AWS Service | Role & Feature Details | Parameters & Ports |
| :--- | :--- | :--- | :--- |
| 1 | **AWS CloudFront** | Global Content Delivery Network (CDN) for React Admin Web (`ADMIN.WEB`), loading in `< 100ms`. | Port 443 (HTTPS) |
| 2 | **AWS Simple Storage Service (S3)** | Stores Frontend static assets (`dist/`) and user-uploaded sign image files (`S3 Media Bucket`). | S3 Standard Bucket |
| 3 | **AWS Application Load Balancer (ALB)** | Load balances and routes HTTPS API traffic (`/api/*`) to Ocelot API Gateway. | Public Subnet `10.0.1.0/24` |
| 4 | **AWS Elastic Container Registry (ECR)** | Secure Docker Container Registry hosting images for 8 Microservices & Python Scraper. | Private Docker Registry |
| 5 | **AWS Amazon EC2 (Elastic Compute Cloud)** | EC2 Virtual instances running Docker containers for 8 Microservices (Ocelot API Gateway + 7 Services) & EC2 Scraper Instance. | Private App Subnet `10.0.2.0/24` |
| 6 | **AWS Cloud Map** | Internal VPC Service Discovery DNS (`*.local`) resolving microservice endpoints for API Gateway. | Internal DNS `*.local` |
| 7 | **AWS Relational Database Service (RDS)** | SQL Server 2022 storing 1,286+ GIS geography SRID 4326 traffic signs, users, and transactions. | Port 1433 (Private DB Subnet `10.0.3.0/24`) |
| 8 | **AWS EventBridge** | Cronjob scheduler (`0 2 * * ? *` - 2 AM daily) triggering Python Scraper Task across 15 provinces. | Cron Schedule |
| 9 | **AWS Secrets Manager & ACM** | Encrypted secret storage (Connection Strings/JWT Keys) and free SSL/TLS certificate management. | Encrypted Secrets / TLS |

---

#### 3. 3-Tier Multi-AZ VPC Architecture

- **AWS Region:** Singapore (`ap-southeast-1`)
- **AWS VPC CIDR:** `10.0.0.0/16`
- **Subnet Layers across Multi-AZ (AZ-A & AZ-B):**
  1. **Public Subnet (`10.0.1.0/24`):** Hosts AWS Application Load Balancer (ALB) and NAT Gateways.
  2. **Private App Subnet (`10.0.2.0/24`):** Hosts AWS EC2 Instances running 8 Microservices:
     - `ApiGateway Container` (Port 5008 - Ocelot API Gateway)
     - `UserService` (Port 5001)
     - `TrafficSignService` (Port 5002)
     - `ContributionService` (Port 5003)
     - `FeedbackService` (Port 5004)
     - `PaymentService` (Port 5005)
     - `RewardService` (Port 5006)
     - `NotificationService` (Port 5007)
     - `EC2 Scraper Instance` (`scrape_signs.py`)
     - `AWS Cloud Map` (Service Discovery) & `SageMaker AI Endpoint` (YOLO AI model)
  3. **Private DB Subnet (`10.0.3.0/24`):** Hosts AWS RDS for SQL Server 2022 (Port 1433) Primary - Standby setup and Amazon ElastiCache (Redis).

---

#### 4. System Architecture Diagram

![overview](/images/5-Workshop/5.1-Workshop-overview/diagram1.png)