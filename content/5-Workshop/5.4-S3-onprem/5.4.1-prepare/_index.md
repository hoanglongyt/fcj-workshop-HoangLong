---
title : "Prepare Frontend Static Assets & Create S3 Static Web Bucket"
date : 2026-07-22 
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

#### 1. Step 5.4.1 Overview

In this step, you will build and package the **TSL-SignMap** system **React Admin Web (`ADMIN.WEB`)** frontend application and provision an **AWS S3 Bucket** configured for **Static Website Hosting**.

- **S3 Bucket Name:** `tsl-signmap-production-static-web-ckroy7`
- **Region:** Singapore (`ap-southeast-1`)
- **Website Endpoint URL:** [http://tsl-signmap-production-static-web-ckroy7.s3-website-ap-southeast-1.amazonaws.com/](http://tsl-signmap-production-static-web-ckroy7.s3-website-ap-southeast-1.amazonaws.com/)

---

#### 2. Step-by-Step Implementation

##### Step 1: Build React Admin Web Frontend
1. Open terminal inside the React frontend repository directory and run build packaging:
   ```bash
   npm run build
   ```
2. The compiled static output directory `dist/` is generated containing `index.html`, JavaScript bundles, CSS stylesheets, and UI media assets.

##### Step 2: Create AWS S3 Bucket
1. Open **AWS S3 Console** and click **Create bucket**.
2. Bucket name: Enter `tsl-signmap-production-static-web-ckroy7`.
3. AWS Region: Select `ap-southeast-1` (Singapore).
4. Object Ownership: Select **ACLs disabled (recommended)**.
5. Block Public Access settings:
   - Retain default block public access settings to prepare for CloudFront Origin Access Control (OAC) setup in the next exercise.
6. Click **Create bucket**.

##### Step 3: Configure S3 Static Website Hosting
1. Click bucket name `tsl-signmap-production-static-web-ckroy7` -> select **Properties** tab.
2. Scroll to **Static website hosting** section and click **Edit**.
3. Select **Enable**.
4. Hosting type: Select **Host a static website**.
5. Index document: Enter `index.html`.
6. Error document: Enter `index.html` (supporting Single Page Application React Router client-side routing).
7. Click **Save changes**.

##### Step 4: Upload Static Assets to S3 Bucket
1. Select **Objects** tab inside S3 Bucket -> click **Upload**.
2. Upload all files and subfolders inside `dist/` into the S3 Bucket:
   ```bash
   aws s3 sync ./dist s3://tsl-signmap-production-static-web-ckroy7/
   ```
3. Click **Upload** and confirm completion.

---

#### 3. Verification

Once upload completes, verify the static website endpoint:
```text
http://tsl-signmap-production-static-web-ckroy7.s3-website-ap-southeast-1.amazonaws.com/
```
The React Admin Web application launches successfully and is prepared for CloudFront CDN distribution in step 5.4.2.
