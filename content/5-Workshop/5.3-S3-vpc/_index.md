---
title : "Setup Microservices Cluster and RDS Database"
date : 2026-07-22 
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### 1. Microservices Application & RDS Database Overview

In this section, you will deploy the 8 Microservices cluster on **AWS EC2 Instances** within the **Private App Subnet** and configure **AWS RDS for SQL Server 2022** database alongside an **Amazon ElastiCache (Redis)** cluster inside the **Private DB Subnet**.

The application architecture ensures all backend services operate securely within the private VPC network, discovering each other via **AWS Cloud Map Service Discovery (`*.local`)** and accepting external API requests exclusively through **Ocelot API Gateway (Port 5008)** routed from the **Application Load Balancer (ALB)**.

---

#### 2. Microservices & Database Components Specification

| Service Name | Port | Subnet Location | Component Details & Responsibilities |
| :--- | :--- | :--- | :--- |
| **ApiGateway Container** | `5008` | Private App Subnet | Ocelot API Gateway receiving and routing HTTPS `/api/*` requests from ALB to internal microservices |
| **UserService** | `5001` | Private App Subnet | User authentication, JWT token management, and role-based authorization |
| **TrafficSignService** | `5002` | Private App Subnet | Spatial GIS traffic sign data management & queries (SRID 4326 standard) |
| **ContributionService** | `5003` | Private App Subnet | Traffic sign user contribution processing & uploading image assets to S3 Media Bucket |
| **FeedbackService** | `5004` | Private App Subnet | User feedback, sign data validation, and report management |
| **PaymentService** | `5005` | Private App Subnet | Payment transactions processing and payment gateway integrations |
| **RewardService** | `5006` | Private App Subnet | User reward points accumulation and redemption logic |
| **NotificationService** | `5007` | Private App Subnet | Real-time push notifications and system event broadcasting |
| **EC2 Scraper Instance** | Cron Task | Private App Subnet | Automated Python script (`scrape_signs.py`) scraping sign data from OpenStreetMap API |
| **AWS RDS SQL Server 2022** | `1433` | Private DB Subnet | Relational Database storing 1,286+ GIS geography traffic signs, users, and transactions (Primary - Standby Multi-AZ) |
| **Amazon ElastiCache** | `6379` | Private DB Subnet | In-memory Redis cache cluster optimizing spatial lookup query speeds |

---

#### 3. Hands-on Lab Steps

- [5.3.1 Deploy AWS RDS SQL Server & ElastiCache Redis Cluster](5.3.1-create-gwe/)
- [5.3.2 Deploy EC2 Microservices Cluster, Ocelot API Gateway & AWS Cloud Map](5.3.2-test-gwe/)