---
title : "Clean up resources"
date : 2026-07-22
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

#### 1. Workshop Conclusion

Congratulations on successfully completing the hands-on workshop for designing, deploying, and securing the **TSL-SignMap** AWS cloud infrastructure!

Throughout this workshop, you implemented key cloud architecture patterns adhering to the **AWS Well-Architected Framework**:
- Provisioned a **3-Tier Multi-AZ AWS VPC (`10.0.0.0/16`)** spanning 2 Availability Zones (**AZ - A** and **AZ - B**).
- Deployed **AWS EC2 Instances** hosting Ocelot API Gateway, 7 Microservices Containers, and the EC2 Scraper Instance.
- Configured internal service discovery via **AWS Cloud Map (`*.local`)**.
- Built a high-availability **AWS RDS for SQL Server 2022** (Primary - Standby Multi-AZ) database and an **Amazon ElastiCache (Redis)** cache cluster.
- Secured private service connectivity with **S3 VPC Endpoint** and **SageMaker VPC Endpoint (YOLO AI)** hardened by **VPC Endpoint Policies**.
- Accelerated global content delivery for React Admin Web using **AWS CloudFront CDN** and **S3 Static Web**.

---

#### 2. Resource Cleanup Steps

To avoid incurring unexpected charges on your AWS account, clean up all provisioned resources in the following step-by-step order:

##### Step 1: Delete VPC Endpoints
1. Open the **AWS VPC Console** and navigate to **Endpoints**.
2. Select the created endpoints (`s3-gwe` and `sagemaker-vpce`).
3. Click **Actions** -> select **Delete endpoints** and confirm.

##### Step 2: Delete Application Load Balancer (ALB)
1. Navigate to **EC2 Console** and select **Load Balancers**.
2. Select `TSL-SignMap-ALB`.
3. Click **Actions** -> select **Delete load balancer** and confirm.

##### Step 3: Terminate EC2 Instances
1. In the **EC2 Console**, click **Instances**.
2. Select `EC2 Microservices Instance` and `EC2 Scraper Instance`.
3. Click **Instance state** -> select **Terminate instance** and confirm.

##### Step 4: Delete RDS SQL Server & ElastiCache Redis
1. Open the **Amazon RDS Console** and select **Databases**.
2. Select `tsl-signmap-db`.
3. Click **Actions** -> select **Delete** (uncheck *Create final snapshot* if not required) and type `delete me` to confirm.
4. Open **Amazon ElastiCache Console**, select the Redis cluster, and click **Delete**.

##### Step 5: Delete S3 Buckets
1. Open the **Amazon S3 Console**.
2. Select `tsl-signmap-media-bucket` and the `S3 Static Web` bucket.
3. Click **Empty** to clear all stored objects, then click **Delete** and type the bucket name to permanently remove them.

##### Step 6: Delete AWS VPC & Network Infrastructure
1. Open the **AWS VPC Console** and select **Your VPCs**.
2. Select `TSL-SignMap-VPC`.
3. Click **Actions** -> select **Delete VPC** and confirm. AWS will automatically clean up associated Subnets, Route Tables, Internet Gateways, and Security Groups.