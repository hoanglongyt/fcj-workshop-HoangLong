---
title : "Deploy AWS RDS SQL Server & ElastiCache Redis Cluster"
date : 2026-07-22 
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

#### 1. RDS SQL Server 2022 & ElastiCache Redis Overview Based on Architecture Diagram

According to the architecture diagram **`TSLSignMap.drawio.png`**, the **TSL-SignMap** system manages spatial GIS traffic sign data (SRID 4326 geographical standard with 1,286+ spatial entities), user accounts, contribution logs, and feedback entries. The database and caching tier is structured as follows:

- **RDS Primary - SQL (7):** Provisioned in **AZ - A (Private DB Sub A)** (`10.0.3.0/24`), receiving read/write traffic directly from `EC2 Ocelot API + 7 Containers` and `EC2 Scraper Instance (11)`.
- **RDS Standby (8):** Provisioned in **AZ - B (Private DB Sub B)** (`10.0.3.128/24`), linked directly to RDS Primary via real-time **Multi-AZ Synchronous Replication (link between 7 and 8)** for high availability failover.
- **Amazon ElastiCache (Redis):** Placed in **Private DB Sub B**, accepting cache connections from `EC2 Scraper Instance (11)` and microservices to store frequent spatial sign lookup queries, reducing latency to under `50ms`.
- **Disaster Recovery Replication (Secondary DR Region) (12):** Automated database snapshot backups from RDS Primary (7) and Standby (8) are replicated to the DR region (12).

---

#### 2. Detailed Deployment Workflow

##### Step 1: Create DB Subnet Group for Multi-AZ
1. Open the **Amazon RDS Console**, and select **Subnet groups** from the left navigation pane.
2. Click **Create DB Subnet Group** and set name to `tsl-signmap-db-subnet-group`.
3. Select VPC `TSL-SignMap-VPC`.
4. Add Subnets: Select 2 Availability Zones (`ap-southeast-1a` for AZ-A and `ap-southeast-1b` for AZ-B) and select 2 database subnets: `Private DB Sub A` (`10.0.3.0/24`) and `Private DB Sub B` (`10.0.3.128/24`).
5. Click **Create**.

##### Step 2: Provision RDS Primary (7) & RDS Standby (8)
1. In the RDS Console, click **Databases** -> click **Create database**.
2. Select **Standard create**.
3. Engine options: Select **Microsoft SQL Server**, edition **SQL Server 2022 Standard Edition**.
4. Deployment options: Select **Multi-AZ DB Instance** (Creates RDS Primary in AZ-A (7) and Standby replica in AZ-B (8)).
5. Settings:
   - DB instance identifier: `tsl-signmap-db`.
   - Master username: `tslsignadmin`.
   - Master password: Enter master password (stored in AWS Secrets Manager).
6. Instance configuration: Select **db.m6i.xlarge** (4 vCPU, 16 GiB RAM).
7. Storage: Autoscaling storage up to `500 GiB`.
8. Connectivity:
   - Virtual private cloud (VPC): Select `TSL-SignMap-VPC`.
   - DB subnet group: Select `tsl-signmap-db-subnet-group`.
   - Public access: Select **No** (Isolated in Private DB Subnet).
   - Existing VPC security groups: Select `RDS-DB-Security-Group` (allowing Port 1433 exclusively from `EC2-App-Security-Group`).
9. Click **Create database**.

##### Step 3: Provision Amazon ElastiCache (Redis) Cluster
1. Open **Amazon ElastiCache Console**, select **Redis clusters** -> click **Create Redis cluster**.
2. Location: Select **AWS Cloud**, cluster mode **Enabled**.
3. Cluster info: Name `tsl-signmap-redis-cache`.
4. Node type: Select `cache.t4g.small`.
5. Subnet group: Select DB tier subnet group.
6. Security group: Allow inbound Port `6379` from `EC2-App-Security-Group` and `EC2 Scraper Instance (11)`.
7. Click **Create**.

---

#### 3. Connection Verification & GIS SRID 4326 Schema Setup

1. From the `EC2 Microservices Instance` in Private App Subnet, verify database port 1433 connectivity to RDS Primary (7):
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