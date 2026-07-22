---
title : "Integrate SSL HTTPS Certificate with AWS Certificate Manager (ACM)"
date : 2026-07-22 
weight : 3
chapter : false
pre : " <b> 5.4.3 </b> "
---

#### 1. Step 5.4.3 Overview

In this step, you will integrate a free **SSL/TLS certificate from AWS Certificate Manager (ACM)** into your **CloudFront CDN** distribution to guarantee end-to-end HTTPS (Port 443) encryption for the **TSL-SignMap React Admin Web** application.

- **Note:** ACM certificates used by CloudFront CDN distributions must be requested exclusively in the **us-east-1 (N. Virginia)** AWS Region.

---

#### 2. Step-by-Step Implementation

##### Step 1: Request SSL/TLS Certificate in AWS Certificate Manager (ACM)
1. Open **AWS Certificate Manager Console**, and switch Region in the top right to **us-east-1 (N. Virginia)**.
2. Click **Request a certificate** -> select **Request a public certificate** -> click **Next**.
3. Fully qualified domain name: Enter application domain name (e.g. `admin.tsl-signmap.com` or `*.tsl-signmap.com`).
4. Validation method: Select **DNS validation (recommended)**.
5. Key algorithm: Select **RSA 2048**.
6. Click **Request**.

##### Step 2: Validate Domain Ownership via Amazon Route 53
1. Click the created certificate entry inside the ACM console list.
2. Under **Domains** section, click **Create records in Route 53**.
3. The system automatically creates the validation CNAME record inside your Amazon Route 53 Hosted Zone.
4. Wait 1 - 3 minutes until certificate status updates to **Issued**.

##### Step 3: Attach ACM Certificate to CloudFront Distribution
1. Switch to **AWS CloudFront Console**, and select your distribution.
2. Under **General** tab, scroll to **Settings** -> click **Edit**.
3. **Alternate domain name (CNAME):** Enter `admin.tsl-signmap.com`.
4. **Custom SSL certificate:** Select the issued ACM certificate (`admin.tsl-signmap.com`).
5. Minimum TLS version: Select **TLSv1.2_2021 (recommended)**.
6. Click **Save changes**.

---

#### 3. Security Verification

1. Open terminal and issue a cURL command against the custom domain:
   ```bash
   curl -I https://admin.tsl-signmap.com
   ```
2. Response returns secure HTTP headers:
   - `HTTP/2 200`
   - `Server: CloudFront`
   - `X-Cache: Hit from cloudfront`
   - Secure padlock icon displays in browser address bar verifying HTTPS connection.
