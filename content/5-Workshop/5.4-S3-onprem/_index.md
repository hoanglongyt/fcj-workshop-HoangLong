---
title : "Configure AWS CloudFront and S3 Static Web"
date : 2026-07-22 
weight : 4 
chapter : false
pre : " <b> 5.4. </b> "
---

#### 1. Admin Web Frontend Distribution Layer Overview (CloudFront & S3 Static Web)

In this section, you will deploy the global distribution infrastructure for the React Admin Web (`ADMIN.WEB`) frontend of the **TSL-SignMap** system at the **Global Edge Layer**, adhering to AWS security and performance best practices:

- **AWS Simple Storage Service (S3 Static Web):** Stores compiled frontend static assets (`dist/`) including HTML, CSS, JavaScript, and user interface media.
- **AWS CloudFront (CDN):** Global Content Delivery Network caching and delivering static assets from the S3 Static Web Bucket to end users with low latency under `< 100ms`.
- **AWS Certificate Manager (ACM):** Issues and manages SSL/TLS security certificates enabling HTTPS connection encryption (Port 443).
- **AWS WAF (Web Application Firewall):** Edge-located web application firewall protecting the site against web exploits (DDoS, SQL Injection, Cross-Site Scripting).

- **Static Web Endpoint URL (AWS S3 Hosting):** [http://tsl-signmap-production-static-web-ckroy7.s3-website-ap-southeast-1.amazonaws.com/](http://tsl-signmap-production-static-web-ckroy7.s3-website-ap-southeast-1.amazonaws.com/)

---

#### 2. Detailed Edge Layer Components

| AWS Service | Role & Function | Configuration & Protocols |
| :--- | :--- | :--- |
| **AWS S3 Static Web** | Stores React Admin Web compiled static assets (`dist/`) | S3 Website Hosting Bucket |
| **AWS CloudFront** | Global CDN delivering static content with page load `< 100ms` | Price Class 100 / HTTPS (Port 443) |
| **AWS Certificate Manager (ACM)** | Provides SSL/TLS certificates for HTTPS encryption | TLS Certificate (Region us-east-1) |
| **AWS WAF** | Filters web traffic blocking malicious exploits at Global Edge | Managed Rule Sets & Rate Limiting |

---

#### 3. Hands-on Lab Exercises

- [5.4.1 Prepare Frontend Static Assets & Create S3 Static Web Bucket](5.4.1-prepare/)
- [5.4.2 Configure AWS CloudFront CDN & Origin Access Control (OAC)](5.4.2-create-interface-enpoint/)
- [5.4.3 Integrate SSL HTTPS Certificate with AWS Certificate Manager (ACM)](5.4.3-test-endpoint/)
- [5.4.4 Configure AWS WAF Firewall & Edge Monitoring](5.4.4-dns-simulation/)
