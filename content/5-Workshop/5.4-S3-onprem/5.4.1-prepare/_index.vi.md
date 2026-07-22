---
title : "Chuẩn bị tệp tĩnh Frontend & Tạo S3 Static Web Bucket"
date : 2026-07-22 
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

#### 1. Tổng quan Bước 5.4.1

Trong bước này, bạn sẽ thực hiện đóng gói ứng dụng **React Admin Web (`ADMIN.WEB`)** của hệ thống **TSL-SignMap** và khởi tạo bộ chứa **AWS S3 Bucket** được cấu hình chế độ **Static Website Hosting**.

- **S3 Bucket Name:** `tsl-signmap-production-static-web-ckroy7`
- **Region:** Singapore (`ap-southeast-1`)
- **Website Endpoint URL:** [http://tsl-signmap-production-static-web-ckroy7.s3-website-ap-southeast-1.amazonaws.com/](http://tsl-signmap-production-static-web-ckroy7.s3-website-ap-southeast-1.amazonaws.com/)

---

#### 2. Quy Trình Thực Hiện Chi Tiết

##### Bước 1: Biên dịch mã nguồn React Admin Web
1. Tại thư mục ứng dụng Frontend React, mở terminal và thực thi lệnh đóng gói:
   ```bash
   npm run build
   ```
2. Thư mục mã nguồn tĩnh `dist/` được tạo thành công chứa `index.html`, các tệp JavaScript bundle, CSS và các tài nguyên hình ảnh.

##### Bước 2: Tạo AWS S3 Bucket
1. Mở **AWS S3 Console**, bấm **Create bucket**.
2. Bucket name: Nhập `tsl-signmap-production-static-web-ckroy7`.
3. AWS Region: Chọn `ap-southeast-1` (Singapore).
4. Object Ownership: Chọn **ACLs disabled (recommended)**.
5. Block Public Access settings for this bucket:
   - Tạm thời giữ mặc định để chuẩn bị cấu hình riêng với CloudFront Origin Access Control (OAC) ở bước tiếp theo.
6. Bấm **Create bucket**.

##### Bước 3: Cấu hình S3 Static Website Hosting
1. Nhấp vào tên bucket `tsl-signmap-production-static-web-ckroy7` -> chuyển sang thẻ **Properties**.
2. Cuộn xuống mục **Static website hosting**, chọn **Edit**.
3. Chọn **Enable**.
4. Hosting type: Chọn **Host a static website**.
5. Index document: Nhập `index.html`.
6. Error document: Nhập `index.html` (để hỗ trợ Client-side Routing cho ứng dụng Single Page Application React Router).
7. Nhấn **Save changes**.

##### Bước 4: Tải tệp tĩnh `dist/` lên S3 Bucket
1. Truy cập thẻ **Objects** trong S3 Bucket -> chọn **Upload**.
2. Tải toàn bộ các tệp và thư mục con bên trong thư mục `dist/` lên S3 Bucket:
   ```bash
   aws s3 sync ./dist s3://tsl-signmap-production-static-web-ckroy7/
   ```
3. Bấm **Upload** và xác nhận hoàn tất.

---

#### 3. Kiểm Tra Kết Nối

Sau khi tải tệp thành công, bạn có thể kiểm tra đường dẫn website tĩnh:
```text
http://tsl-signmap-production-static-web-ckroy7.s3-website-ap-southeast-1.amazonaws.com/
```
Giao diện React Admin Web khởi chạy thành công và sẵn sàng để tích hợp CDN ở bước 5.4.2.
