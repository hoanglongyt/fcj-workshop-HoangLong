---
title : "Configure AWS CloudFront CDN & Origin Access Control (OAC)"
date : 2026-07-22 
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

#### 1. Step 5.4.2 Overview

In this step, you will configure an **AWS CloudFront Distribution** at the **Global Edge Services** layer connecting to the **S3 Static Web Bucket (`tsl-signmap-production-static-web-ckroy7`)**.

- **Goal:** Accelerate React Admin Web asset delivery globally under `< 100ms` and enforce security using **Origin Access Control (OAC)** restricting access strictly through CloudFront CDN.

---

#### 2. Step-by-Step Implementation

##### Step 1: Create CloudFront Distribution
1. Open **AWS CloudFront Console** and click **Create distribution**.
2. **Origin domain:** Select S3 Bucket `tsl-signmap-production-static-web-ckroy7.s3.ap-southeast-1.amazonaws.com`.
3. **Origin access:**
   - Select **Origin access control settings (recommended)**.
   - Click **Create control setting**, retain defaults and click **Create**.
4. **Default cache behavior:**
   - Viewer protocol policy: Select **Redirect HTTP to HTTPS**.
   - Allowed HTTP methods: Select `GET, HEAD`.
   - Cache key and origin requests: Select **CachingOptimized** (recommended static asset caching policy).
5. **Price class:** Select **Use all edge locations (best performance)** or Price Class 100.
6. Default root object: Enter `index.html`.
7. Click **Create distribution**.

##### Step 2: Update S3 Bucket Policy with OAC
1. Once Distribution creation completes, copy the generated JSON **S3 Bucket Policy**.
2. Open **AWS S3 Console** -> select bucket `tsl-signmap-production-static-web-ckroy7` -> select **Permissions** tab.
3. Under **Bucket policy**, click **Edit** and paste JSON:
   ```json
   {
       "Version": "2012-10-17",
       "Statement": {
           "Sid": "AllowCloudFrontServicePrincipalReadOnly",
           "Effect": "Allow",
           "Principal": {
               "Service": "cloudfront.amazonaws.com"
           },
           "Action": "s3:GetObject",
           "Resource": "arn:aws:s3:::tsl-signmap-production-static-web-ckroy7/*",
           "Condition": {
               "StringEquals": {
                   "AWS:SourceArn": "arn:aws:cloudfront::<account_id>:distribution/<distribution_id>"
               }
           }
       }
   }
   ```
4. Click **Save changes**.

##### Step 3: Configure Custom Error Responses for React SPA Routing
To prevent `404 Not Found` errors when users reload deep client routes in React Router (e.g. `/admin/signs`, `/admin/users`):
1. In CloudFront Distribution -> select **Error pages** tab -> click **Create custom error response**.
2. HTTP error code: Select **404: Not Found**.
3. Customize error response: Select **Yes**.
4. Response page path: Enter `/index.html`.
5. HTTP response code: Select **200: OK**.
6. Click **Create custom error response**.

---

#### 3. Verification

1. Copy the CloudFront Domain Name (e.g. `d123456789.cloudfront.net`).
2. Open browser and navigate to: `https://d123456789.cloudfront.net`
3. Verify application loads successfully over secure HTTPS with response latency under `< 100ms`.