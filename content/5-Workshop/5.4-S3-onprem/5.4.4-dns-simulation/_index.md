---
title : "Configure AWS WAF Firewall & Edge Monitoring"
date : 2026-07-22 
weight : 4
chapter : false
pre : " <b> 5.4.4 </b> "
---

#### 1. Step 5.4.4 Overview

In this step, you will deploy an **AWS WAF (Web Application Firewall)** web ACL at the **Global Edge Services** layer attached directly to your **CloudFront CDN** distribution.

- **Goal:** Protect the React Admin Web interface and backend APIs against cyber exploits such as SQL Injection, Cross-Site Scripting (XSS), HTTP Flood / DDoS attacks, and malicious bots.

---

#### 2. Step-by-Step Implementation

##### Step 1: Provision AWS WAF Web ACL
1. Open **AWS WAF & Shield Console**, and click **Web ACLs** in the left navigation pane.
2. Resource type: Select **Global resources (CloudFront distribution)**.
3. Click **Create web ACL**.
4. Name: Enter `tsl-signmap-edge-waf-acl`.
5. CloudFront distributions to associate: Select the CloudFront Distribution created in step 5.4.2.
6. Click **Next**.

##### Step 2: Add Managed Protection Rule Sets
1. Under **Add rules and rule groups**, click **Add rules** -> select **Add managed rule groups**.
2. **AWS managed rule groups:**
   - Enable **Core rule set (CRS):** Protects against OWASP Top 10 web security vulnerabilities.
   - Enable **Known bad inputs:** Blocks request patterns containing malformed code payloads.
   - Enable **Amazon IP reputation list:** Blocks IP addresses flagged on AWS threat intelligence lists.
3. Click **Add rules**.

##### Step 3: Add Rate-based Rule for DDoS / Brute-Force Prevention
1. Click **Add rules** -> select **Add my own rules and rule groups**.
2. Rule type: Select **Rate-based rule**.
3. Rule name: Enter `PreventDDoSAndBruteForce`.
4. Rate limit: Enter `2000` (max 2000 requests per IP address within 5-minute window).
5. Action: Select **Block**.
6. Click **Add rule**.

##### Step 4: Finalize & Attach Web ACL
1. Set rule evaluation priority and click **Next**.
2. Metric name: Retain default metrics forwarding to **Amazon CloudWatch**.
3. Click **Create web ACL**.

---

#### 3. Edge Traffic Verification & Monitoring

1. Send a simulated XSS exploit request against the CloudFront domain:
   ```bash
   curl -I "https://admin.tsl-signmap.com/?q=<script>alert('xss')</script>"
   ```
2. **Result:** AWS WAF inspects and blocks the request, returning `HTTP 403 Forbidden`.

3. Open **Amazon CloudWatch Console** -> select **Metrics** -> view `AllowedRequests` vs `BlockedRequests` metrics graphs to monitor real-time edge security traffic.