---
title : "Cấu hình AWS CloudFront và S3 Static Web"
date : 2026-07-22 
weight : 4 
chapter : false
pre : " <b> 5.4. </b> "
---

#### 1. Tổng quan Tầng Phân Phối Giao Diện Admin Web (CloudFront & S3 Static Web)

Trong phần này, chúng ta sẽ thực hành triển khai hạ tầng phân phối giao diện ứng dụng quản trị React Admin Web (`ADMIN.WEB`) của hệ thống **TSL-SignMap** tại tầng **Global Edge Services** tuân theo các tiêu chuẩn bảo mật và hiệu năng hàng đầu của AWS:

- **AWS Simple Storage Service (S3 Static Web):** Lưu trữ toàn bộ các tệp mã nguồn tĩnh sau khi đã biên dịch (`dist/`) bao gồm HTML, CSS, JavaScript và assets giao diện người dùng.
- **AWS CloudFront (CDN):** Mạng phân phối nội dung toàn cầu giúp lưu đệm (cache) và phân phối các tệp tĩnh từ S3 Static Web Bucket tới người dùng cuối với độ trễ thấp `< 100ms`.
- **AWS Certificate Manager (ACM):** Cấp và quản lý tự động chứng chỉ bảo mật SSL/TLS phục vụ mã hóa kết nối HTTPS (Port 443).
- **AWS WAF (Web Application Firewall):** Tường lửa ứng dụng web đặt tại vùng Global Edge giúp ngăn chặn các cuộc tấn công web phổ biến (DDoS, SQL Injection, Cross-Site Scripting).

- **Đường dẫn trang web tĩnh (S3 Static Web Hosting):** [http://tsl-signmap-production-static-web-ckroy7.s3-website-ap-southeast-1.amazonaws.com/](http://tsl-signmap-production-static-web-ckroy7.s3-website-ap-southeast-1.amazonaws.com/)

---

#### 2. Chi Tiết Thành Phần Hạ Tầng Tầng Edge

| Dịch Vụ AWS | Vai Trò & Chức Năng Chi Tiết | Cấu Hinh & Giao Thức |
| :--- | :--- | :--- |
| **AWS S3 Static Web** | Bộ chứa lưu trữ các tệp ứng dụng React Admin Web (`dist/`) | S3 Website Hosting Bucket |
| **AWS CloudFront** | Mạng CDN phân phối nội dung tĩnh toàn cầu với tốc độ nạp trang `< 100ms` | Price Class 100 / HTTPS (Port 443) |
| **AWS Certificate Manager (ACM)** | Cấp chứng chỉ SSL/TLS mã hóa kết nối HTTPS | TLS Certificate (Region us-east-1) |
| **AWS WAF** | Tường lửa lọc traffic chống tấn công web độc hại ở tầng Global Edge | Managed Rule Sets & Rate Limiting |

---

#### 3. Nội Dung Thực Hành Chi Tiết

- [5.4.1 Chuẩn bị tệp tĩnh Frontend & Tạo S3 Static Web Bucket](5.4.1-prepare/)
- [5.4.2 Cấu hình AWS CloudFront CDN & Origin Access Control (OAC)](5.4.2-create-interface-enpoint/)
- [5.4.3 Tích hợp Chứng chỉ SSL HTTPS với AWS Certificate Manager (ACM)](5.4.3-test-endpoint/)
- [5.4.4 Cấu hình Tường lửa AWS WAF & Giám sát Tầng Edge](5.4.4-dns-simulation/)
