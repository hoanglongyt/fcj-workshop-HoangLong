---
title : "Deploy EC2 Microservices Cluster, Ocelot API Gateway & AWS Cloud Map"
date : 2026-07-22 
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---

#### 1. EC2 Microservices & Endpoints Overview Based on Architecture Diagram

According to the architecture diagram **`TSLSignMap.drawio.png`**, the 8 Microservices container cluster and compute workloads are designed for load balancing, automated scaling, and private connectivity:

- **ALB (6) & Target Group -> Auto Scaling Group:** HTTPS API request traffic (4) flows from CloudFront/Route 53 through Internet Gateway (5) into **ALB (6)**. The ALB forwards requests to the **Target Group** routing into **`EC2 Ocelot API + 7 Containers`** managed inside an **Auto Scaling Group** spanning **AZ - A (Private subnet A)** and **AZ - B (Private subnet B)**.
- **SageMaker Endpoint (9):** Integrates the **SageMaker (YOLO AI)** model via a private **`Endpoint (SageMaker) (9)`** into `EC2 Ocelot API + 7 Containers` for AI image-based traffic sign detection.
- **S3 Endpoint (10):** Connects the microservices cluster to **`S3 (Media Bucket)`** via a private **`EndPoint - S3 (10)`** to store and retrieve uploaded sign images.
- **EC2 Scraper Instance (11):** Placed in **AZ - A (Private subnet A)**, running the `scrape_signs.py` script via Cron schedule:
  - Scrapes sign data from **OpenStreetMap API** via **NAT Gateway** (in Public Subnet A).
  - Updates scraped sign records directly into **RDS Primary - SQL (7)**.
  - Updates cache data into **Amazon ElastiCache (Redis)**.

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
Configure API routing from ALB (6) (Port 5008) to internal microservices:
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

##### Step 3: Deploy Docker Containers on EC2 Auto Scaling Group
1. Provision **EC2 Instances** within Private Subnet A (AZ-A) and Private Subnet B (AZ-B).
2. Authenticate to **AWS ECR** and pull container images:
   ```bash
   aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.ap-southeast-1.amazonaws.com
   docker pull <aws_account_id>.dkr.ecr.ap-southeast-1.amazonaws.com/tsl-trafficsign-service:latest
   ```
3. Run microservices containers using Docker Compose or Docker CLI:
   ```bash
   docker run -d --name trafficsignservice -p 5002:5002 -e "ConnectionStrings__SQLServer=Server=tsl-signmap-db.c123456789.ap-southeast-1.rds.amazonaws.com;Database=TSLSignMapDB;User Id=tslsignadmin;Password=<secret_password>;" <aws_account_id>.dkr.ecr.ap-southeast-1.amazonaws.com/tsl-trafficsign-service:latest
   ```

##### Step 4: Run EC2 Scraper Instance (11)
Launch the EC2 Scraper Instance (11) in Private Subnet A retrieving sign data from OpenStreetMap API:
```bash
python3 scrape_signs.py --province "HoChiMinh" --batch-size 500
```

---

#### 3. End-to-End Data Flow Verification & Health Check

1. Invoke Ocelot API Gateway Health Check endpoint via ALB (6):
   ```bash
   curl -I https://api.tsl-signmap.com/api/trafficsigns/health
   ```
   *Result:* Returns HTTP Status `200 OK`.

2. Test spatial GIS SRID 4326 traffic sign location queries via SageMaker Endpoint (9) and RDS Primary (7):
   ```bash
   curl -X GET "https://api.tsl-signmap.com/api/trafficsigns/nearby?lat=10.7769&lng=106.7009&radius=500"
   ```
   *Result:* Returns JSON list of nearby traffic signs retrieved from RDS Primary SQL (7) and cached by Redis.
