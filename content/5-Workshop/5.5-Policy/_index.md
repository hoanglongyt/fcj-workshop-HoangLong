---
title : "Configure VPC Endpoints (S3 & SageMaker) and Policies"
date : 2026-07-22
weight : 5
chapter : false
pre : " <b> 5.5 </b> "
---

#### 1. VPC Endpoints & Endpoint Policies Overview

In the **TSL-SignMap** system architecture, following the **AWS Well-Architected Framework**, all network traffic between EC2 Microservices, **Amazon S3**, and the **Amazon SageMaker (YOLO AI)** inference model is routed securely through **VPC Endpoints (AWS PrivateLink)** without traversing the public internet.

- **S3 VPC Endpoint:** Grants `ContributionService` secure, private access to store and retrieve traffic sign images in **S3 Media Bucket**.
- **SageMaker VPC Endpoint:** Enables `ApiGateway` and microservices to send sign detection inference calls to the **SageMaker YOLO AI Endpoint** privately inside the VPC.
- **VPC Endpoint Policies (IAM Resource Policies):** Attached directly to VPC Endpoints to enforce least-privilege access control and prevent unauthorized data exfiltration to external S3 buckets.

---

#### 2. Configuring S3 VPC Endpoint Policy (Restricting S3 Access)

The policy below ensures EC2 instances within the VPC are strictly permitted to read/write to **`tsl-signmap-media-bucket`**, while requests to any unauthorized external S3 buckets are denied:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowTSLMediaBucketAccessOnly",
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::tsl-signmap-media-bucket",
        "arn:aws:s3:::tsl-signmap-media-bucket/*"
      ]
    }
  ]
}
```

---

#### 3. Configuration & Verification Steps

##### Step 1: Attach Policy to S3 VPC Endpoint
1. Navigate to **AWS VPC Console** -> select **Endpoints**.
2. Select the created **S3 VPC Endpoint** (`s3-gwe`).
3. Click **Policy** tab and choose **Edit Policy**.
4. Paste the restriction JSON policy above and click **Save**.

##### Step 2: Test Connectivity from EC2 Microservices Instance
1. Connect to the `EC2 Microservices Instance` in Private App Subnet via AWS Session Manager.
2. Test uploading a sign image to the authorized system S3 Media Bucket:
   ```bash
   aws s3 cp traffic_sign_001.jpg s3://tsl-signmap-media-bucket/
   ```
   *Result:* File uploaded successfully over the private network.

3. Verify security enforcement by attempting an upload to an unauthorized external S3 bucket:
   ```bash
   aws s3 cp traffic_sign_001.jpg s3://external-unauthorized-bucket/
   ```
   *Result:* Request fails with `AccessDenied` blocked by the VPC Endpoint Policy.

---

#### 4. Summary

By configuring **S3 VPC Endpoint** and **SageMaker VPC Endpoint** paired with **VPC Endpoint Policies**, the TSL-SignMap infrastructure guarantees end-to-end data security for image assets and AI model calls, preventing data leaks and meeting enterprise compliance standards.
