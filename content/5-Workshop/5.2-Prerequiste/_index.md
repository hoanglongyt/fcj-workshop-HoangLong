---
title : "Prerequisites and 3-Tier VPC Subnet Setup"
date : 2026-07-22 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### 1. Required IAM Permissions

To deploy the **TSL-SignMap** infrastructure including the 3-Tier VPC, EC2 Instances, RDS SQL Server, Application Load Balancer, S3 Buckets, and Secrets Manager, your AWS user account requires an IAM Policy with the following permissions:

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

#### 2. AWS VPC Network Architecture & Subnet Planning

The **TSL-SignMap** network infrastructure is deployed in the AWS Singapore Region (`ap-southeast-1`) with an **AWS VPC CIDR (`10.0.0.0/16`)** divided into 3 subnet tiers across 2 Availability Zones (**AZ - A** and **AZ - B**):

##### Subnet CIDR Block Planning Table

| Subnet Tier | Subnet Name | Availability Zone | CIDR Block | Purpose & Workload |
| :--- | :--- | :--- | :--- | :--- |
| **Public Subnet** | `Public Subnet A` | `ap-southeast-1a` (AZ-A) | `10.0.1.0/24` | Hosts ALB for public HTTPS API traffic and NAT Gateway A |
| **Public Subnet** | `Public Subnet B` | `ap-southeast-1b` (AZ-B) | `10.0.1.128/24` | Hosts secondary ALB and NAT Gateway B for Multi-AZ redundancy |
| **Private App Subnet** | `Private Subnet A` | `ap-southeast-1a` (AZ-A) | `10.0.2.0/24` | Hosts EC2 Ocelot API Gateway, 7 Microservices Containers & EC2 Scraper Instance |
| **Private App Subnet** | `Private Subnet B` | `ap-southeast-1b` (AZ-B) | `10.0.2.128/24` | Hosts redundant EC2 Ocelot API Gateway + Microservices instances |
| **Private DB Subnet** | `Private DB Sub A` | `ap-southeast-1a` (AZ-A) | `10.0.3.0/24` | Hosts Primary AWS RDS for SQL Server 2022 (Port 1433) |
| **Private DB Subnet** | `Private DB Sub B` | `ap-southeast-1b` (AZ-B) | `10.0.3.128/24` | Hosts Standby AWS RDS SQL Server (Multi-AZ synchronous replication) & ElastiCache (Redis) |

---

#### 3. Network & Security Provisioning Workflow

##### Step 1: Initialize AWS VPC & Gateways
1. Navigate to **AWS VPC Console** and click **Create VPC**.
2. Select **VPC and more**, set **Name tag**: `TSL-SignMap-VPC`.
3. Set **IPv4 CIDR block**: `10.0.0.0/16`.
4. Set **Number of Availability Zones (AZs)**: `2` (`ap-southeast-1a` and `ap-southeast-1b`).
5. Set **Number of Public subnets**: `2`.
6. Set **Number of Private subnets**: `4` (2 App Subnets + 2 DB Subnets).
7. Enable **NAT Gateways** (In 2 AZs) and **VPC Endpoints** for private connectivity.

##### Step 2: Configure Security Groups
- **`ALB-Security-Group`**: Allows inbound **HTTPS (Port 443)** and **HTTP (Port 80)** from `0.0.0.0/0`.
- **`EC2-App-Security-Group`**: Allows inbound **Port 5008 (Ocelot API Gateway)** from `ALB-Security-Group` and internal inter-service communications.
- **`RDS-DB-Security-Group`**: Allows inbound **Port 1433 (SQL Server)** and **Port 6379 (Redis)** exclusively from `EC2-App-Security-Group`.