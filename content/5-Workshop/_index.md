---
title: "Workshop"
date: 2026-07-22
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

#### Design and Deploy AWS Infrastructure for TSL-SignMap System

#### Overview

**TSL-SignMap** is a spatial GIS traffic sign management, contribution, and lookup system (SRID 4326) operating on **AWS** cloud infrastructure built following the **AWS Well-Architected Framework**. The system is designed for High Availability, Scalability, and Multi-layer Security.

In this workshop, you will learn how to design, configure, and optimize a comprehensive AWS infrastructure for TSL-SignMap featuring key architecture layers:

+ **Global Edge & Security Layer:** Utilizing **Amazon Route 53**, **AWS CloudFront (CDN)** paired with **AWS WAF** and **AWS Certificate Manager (ACM)** to accelerate React Admin Web distribution, defend against web threats, and manage SSL/TLS certificates.
+ **3-Tier Multi-AZ VPC Architecture:** Deploying an AWS VPC across 2 Availability Zones (AZ-A & AZ-B) with 3 subnet tiers:
  - **Public Subnet:** Hosts the **Application Load Balancer (ALB)** handling HTTPS traffic and **NAT Gateways** for outbound internet connectivity.
  - **Private App Subnet:** Hosts an **Auto Scaling Group** running **Ocelot API Gateway**, **7 Microservices Containers**, and a **SageMaker AI Endpoint** integrated with YOLO AI for sign detection.
  - **Private DB Subnet:** Hosts **AWS RDS for SQL Server 2022** in Primary - Standby (Multi-AZ replication) setup and **Amazon ElastiCache (Redis)**.
+ **Private Connectivity & Integrations (VPC Endpoints & Storage):**
  - **S3 VPC Endpoint (Gateway/Interface):** Grants microservices secure, private access to **S3 Media Bucket** (storing traffic sign images) without traversing the public internet.
  - **SageMaker VPC Endpoint:** Enables API Gateway to invoke SageMaker AI models privately within the VPC.
  - **EC2 Scraper Instance:** Automatically scrapes traffic sign data from OpenStreetMap API and updates RDS SQL Server.
+ **Backup & Disaster Recovery:** Syncs critical data to a **Secondary Disaster Recovery Region** via AWS Backup, RDS Backup, and S3 Cross-Region Replication.
+ **Governance & Monitoring:** Integrates **AWS Secrets Manager**, **AWS IAM**, **Amazon ECR**, and **Amazon CloudWatch** for centralized infrastructure monitoring and management.

#### Content

1. [TSL-SignMap System Overview and AWS Infrastructure](5.1-Workshop-overview/)
2. [Prerequisites and 3-Tier VPC Subnet Setup](5.2-Prerequiste/)
3. [Setup Microservices Cluster and RDS Database](5.3-S3-vpc/)
4. [Configure AWS CloudFront and S3 Static Web](5.4-S3-onprem/)
5. [Configure VPC Endpoints (S3 & SageMaker) and Policies](5.5-Policy/)
6. [Clean up resources](5.6-Cleanup/)