---
title : "Cấu hình AWS CloudFront CDN & Origin Access Control (OAC)"
date : 2026-07-22 
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

#### 1. Tổng quan Bước 5.4.2

Trong bước này, bạn sẽ cấu hình phân phối **AWS CloudFront Distribution** tại tầng **Global Edge Services** kết nối tới bộ chứa **S3 Static Web Bucket (`tsl-signmap-production-static-web-ckroy7`)**.

- **Mục tiêu:** Tăng tốc nạp ứng dụng React Admin Web xuống dưới `< 100ms` toàn cầu và thắt chặt bảo mật bằng **Origin Access Control (OAC)** để bắt buộc mọi truy cập phải đi qua CloudFront CDN.

---

#### 2. Quy Trình Thực Hiện Chi Tiết

##### Bước 1: Khởi tạo CloudFront Distribution
1. Mở **AWS CloudFront Console**, bấm **Create distribution**.
2. **Origin domain:** Chọn S3 Bucket `tsl-signmap-production-static-web-ckroy7.s3.ap-southeast-1.amazonaws.com`.
3. **Origin access:**
   - Chọn **Origin access control settings (recommended)**.
   - Bấm **Create control setting**, giữ tên mặc định và bấm **Create**.
4. **Default cache behavior:**
   - Viewer protocol policy: Chọn **Redirect HTTP to HTTPS**.
   - Allowed HTTP methods: Chọn `GET, HEAD`.
   - Cache key and origin requests: Chọn **CachingOptimized** (Policy khuyến nghị tối ưu cache tệp tĩnh).
5. **Price class:** Chọn **Use all edge locations (best performance)** hoặc Price Class 100.
6. Default root object: Nhập `index.html`.
7. Bấm **Create distribution**.

##### Bước 2: Cập nhật S3 Bucket Policy với OAC
1. Sau khi tạo Distribution, CloudFront cung cấp một đoạn JSON **S3 Bucket Policy**. Sao chép đoạn policy này.
2. Truy cập **AWS S3 Console** -> chọn bucket `tsl-signmap-production-static-web-ckroy7` -> thẻ **Permissions**.
3. Tại mục **Bucket policy**, bấm **Edit** và dán đoạn JSON:
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
4. Bấm **Save changes**.

##### Bước 3: Cấu hình Custom Error Responses cho React SPA Routing
Để tránh lỗi `404 Not Found` khi người dùng refresh các tuyến đường trong React Router (ví dụ `/admin/signs`, `/admin/users`):
1. Trong CloudFront Distribution -> chọn thẻ **Error pages** -> chọn **Create custom error response**.
2. HTTP error code: Chọn **404: Not Found**.
3. Customize error response: Chọn **Yes**.
4. Response page path: Nhập `/index.html`.
5. HTTP response code: Chọn **200: OK**.
6. Bấm **Create custom error response**.

---

#### 3. Kiểm Tra Kết Nối

1. Sao chép tên miền CloudFront Domain Name (ví dụ `d123456789.cloudfront.net`).
2. Mở trình duyệt và truy cập: `https://d123456789.cloudfront.net`
3. Kiểm tra trang web nạp thành công qua kết nối bảo mật HTTPS với độ trễ phản hồi `< 100ms`.