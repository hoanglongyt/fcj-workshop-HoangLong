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

#### 3. Multi-AZ VPC Architecture

- **Infrastructure Scope:** AWS Region Singapore (`ap-southeast-1`) with **AWS VPC (`10.0.0.0/16`)** deployed across 2 Availability Zones (**AZ - A** and **AZ - B**).
- **Subnet Layer Segmentation based on Architecture Diagram:**
  1. **Public Subnet Tier (Public Subnet A & Public Subnet B):**
     - Receives inbound traffic from **Internet Gateway (5)** into **AWS Application Load Balancer (ALB) (6)** distributing requests to Target Group.
     - Hosts **NAT Gateways** in each AZ enabling private workloads to initiate outbound internet connections (e.g. scraping OpenStreetMap API).
  2. **Private App Subnet Tier (Private Subnet A & Private Subnet B):**
     - Deployed within an **Auto Scaling Group** spanning AZ-A and AZ-B.
     - **Private Subnet A (AZ-A):** Hosts `EC2 Ocelot API Gateway + 7 Microservices Containers` and `EC2 Scraper Instance (11)`.
     - **Private Subnet B (AZ-B):** Hosts redundant `EC2 Ocelot API Gateway + 7 Microservices Containers`.
     - **Private VPC Endpoint Integrations:**
       - Connected to **SageMaker (YOLO AI)** via **SageMaker VPC Endpoint (9)**.
       - Connected to **S3 Media Bucket** via **S3 VPC Endpoint (10)**.
       - Connected to **Amazon ElastiCache (Redis)** cache cluster.
  3. **Private DB Subnet Tier (Private DB Sub A & Private DB Sub B):**
     - **Private DB Sub A (AZ-A):** Hosts **RDS Primary - SQL** (SQL Server 2022) handling data queries from EC2 Microservices and EC2 Scraper.
     - **Private DB Sub B (AZ-B):** Hosts **RDS Standby** for continuous Multi-AZ synchronous replication and high-availability failover.
  4. **Secondary Disaster Recovery Region (12):**
     - Synchronizes cross-region backup data including `S3 Bucket`, `AWS Backup`, and `RDS Backup`.

---

#### 4. System Architecture Diagram

![overview](/images/5-Workshop/5.1-Workshop-overview/diagram1.png)