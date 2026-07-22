---
title : "Deploy EC2 Microservices Cluster, Ocelot API Gateway & AWS Cloud Map"
date : 2026-07-22 
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---

#### 1. EC2 Microservices & Service Discovery Overview

In this section, you will deploy the 8 Microservices container cluster on **AWS EC2 Instances** within the **Private App Subnet** (`10.0.2.0/24`) integrated with **AWS Cloud Map Service Discovery (`*.local`)** and **Ocelot API Gateway**.

- **AWS EC2 Microservices Instances:** Hosts Docker containers for `Ocelot API Gateway (Port 5008)` and 7 business services (`UserService:5001`, `TrafficSignService:5002`, `ContributionService:5003`, `FeedbackService:5004`, `PaymentService:5005`, `RewardService:5006`, `NotificationService:5007`).
- **AWS Cloud Map (Service Discovery):** Initializes the private DNS namespace **`tslsignmap.local`** allowing Ocelot API Gateway to dynamically resolve private microservice IP addresses.
- **EC2 Scraper Instance:** Dedicated instance executing the Python script `scrape_signs.py` triggered by **AWS EventBridge** (2 AM daily cron) to scrape traffic sign data from OpenStreetMap API and update RDS SQL Server 2022.

---

#### 2. Detailed Deployment Workflow

##### Step 1: Create Private DNS Namespace on AWS Cloud Map
1. Open **AWS Cloud Map Console** and click **Create namespace**.
2. Namespace name: Set `tslsignmap.local`.
3. Instance discovery: Select **API calls and DNS queries in VPCs**.
4. VPC: Select `TSL-SignMap-VPC`.
5. Register 8 Services:
   - `gateway.tslsignmap.local` (Port 5008)
   - `user.tslsignmap.local` (Port 5001)
   - `trafficsign.tslsignmap.local` (Port 5002)
   - `contribution.tslsignmap.local` (Port 5003)
   - `feedback.tslsignmap.local` (Port 5004)
   - `payment.tslsignmap.local` (Port 5005)
   - `reward.tslsignmap.local` (Port 5006)
   - `notification.tslsignmap.local` (Port 5007)

##### Step 2: Configure Ocelot API Gateway (`ocelot.json`)
Configure API routing from ALB (Port 5008) to internal microservices:
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

##### Step 3: Deploy Docker Containers on EC2 Instances
1. Provision **EC2 Instances** within Private App Subnet.
2. Authenticate to **AWS ECR** and pull container images:
   ```bash
   aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.ap-southeast-1.amazonaws.com
   docker pull <aws_account_id>.dkr.ecr.ap-southeast-1.amazonaws.com/tsl-trafficsign-service:latest
   ```
3. Run microservices containers using Docker Compose or Docker CLI:
   ```bash
   docker run -d --name trafficsignservice -p 5002:5002 -e "ConnectionStrings__SQLServer=Server=tsl-signmap-db.c123456789.ap-southeast-1.rds.amazonaws.com;Database=TSLSignMapDB;User Id=tslsignadmin;Password=<secret_password>;" <aws_account_id>.dkr.ecr.ap-southeast-1.amazonaws.com/tsl-trafficsign-service:latest
   ```

##### Step 4: Run EC2 Scraper Task (`scrape_signs.py`)
Launch the EC2 Scraper Instance retrieving traffic sign data from OpenStreetMap API:
```bash
python3 scrape_signs.py --province "HoChiMinh" --batch-size 500
```

---

#### 3. End-to-End Data Flow Verification & Health Check

1. Invoke Ocelot API Gateway Health Check endpoint via ALB:
   ```bash
   curl -I https://api.tsl-signmap.com/api/trafficsigns/health
   ```
   *Result:* Returns HTTP Status `200 OK`.

2. Test spatial GIS SRID 4326 traffic sign location queries:
   ```bash
   curl -X GET "https://api.tsl-signmap.com/api/trafficsigns/nearby?lat=10.7769&lng=106.7009&radius=500"
   ```
   *Result:* Returns JSON list of nearby traffic signs retrieved from RDS SQL Server 2022 and cached by Redis.
