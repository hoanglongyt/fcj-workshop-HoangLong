---
title : "Deploy AWS RDS SQL Server & ElastiCache Redis Cluster"
date : 2026-07-22 
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

#### 1. RDS SQL Server 2022 & ElastiCache Redis Overview

The **TSL-SignMap** system manages spatial GIS traffic sign data (SRID 4326 geographical standard with 1,286+ spatial entities), user accounts, contribution logs, and feedback entries. All core system data is stored in **AWS RDS for SQL Server 2022** paired with an **Amazon ElastiCache (Redis)** cache cluster.

- **AWS RDS for SQL Server 2022:** Deployed within the **Private DB Subnet** (`10.0.3.0/24`) using a **Multi-AZ Deployment** model (RDS Primary in `Private DB Sub A` / RDS Standby in `Private DB Sub B`). This configuration guarantees high availability, synchronous replication, and automated failover.
- **Amazon ElastiCache for Redis:** In-memory Redis cluster (Port 6379) caching frequent GIS traffic sign lookup queries, reducing database load and delivering API response times under `50ms`.

---

#### 2. Detailed Deployment Workflow

##### Step 1: Create DB Subnet Group
1. Open the **Amazon RDS Console**, and select **Subnet groups** from the left navigation pane.
2. Click **Create DB Subnet Group** and set name to `tsl-signmap-db-subnet-group`.
3. Select VPC `TSL-SignMap-VPC`.
4. Add Subnets: Select 2 Availability Zones (`ap-southeast-1a` & `ap-southeast-1b`) and select the 2 database tier subnets: `Private DB Sub A` (`10.0.3.0/24`) and `Private DB Sub B` (`10.0.3.128/24`).
5. Click **Create**.

##### Step 2: Provision AWS RDS for SQL Server 2022
1. In the RDS Console, click **Databases** -> click **Create database**.
2. Select **Standard create**.
3. Engine options: Select **Microsoft SQL Server**, edition **SQL Server 2022 Standard Edition**.
4. Deployment options: Select **Multi-AZ DB Instance** (Creates a synchronous standby in AZ-B).
5. Settings:
   - DB instance identifier: `tsl-signmap-db`.
   - Master username: `tslsignadmin`.
   - Master password: Enter secure master password (encrypted in AWS Secrets Manager).
6. Instance configuration: Select **db.m6i.xlarge** (4 vCPU, 16 GiB RAM).
7. Storage: Autoscaling storage up to `500 GiB`.
8. Connectivity:
   - Virtual private cloud (VPC): Select `TSL-SignMap-VPC`.
   - DB subnet group: Select `tsl-signmap-db-subnet-group`.
   - Public access: Select **No** (Strictly isolated in private network).
   - Existing VPC security groups: Select `RDS-DB-Security-Group` (allowing Port 1433 exclusively from `EC2-App-Security-Group`).
9. Click **Create database**.

##### Step 3: Provision Amazon ElastiCache (Redis) Cluster
1. Open **Amazon ElastiCache Console**, select **Redis clusters** -> click **Create Redis cluster**.
2. Location: Select **AWS Cloud**, cluster mode **Enabled**.
3. Cluster info: Name `tsl-signmap-redis-cache`.
4. Node type: Select `cache.t4g.small`.
5. Subnet group: Select DB tier subnet group.
6. Security group: Allow inbound Port `6379` from `EC2-App-Security-Group`.
7. Click **Create**.

---

#### 3. Connection Verification & GIS SRID 4326 Schema Setup

1. From the `EC2 Microservices Instance` in Private App Subnet, verify database port 1433 connectivity:
   ```bash
   nc -zv tsl-signmap-db.c123456789.ap-southeast-1.rds.amazonaws.com 1433
   ```
2. Execute SQL DDL script creating tables and spatial indices:
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