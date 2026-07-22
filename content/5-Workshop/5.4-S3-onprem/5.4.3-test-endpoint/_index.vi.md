---
title : "Tích hợp Chứng chỉ SSL HTTPS với AWS Certificate Manager (ACM)"
date : 2026-07-22 
weight : 3
chapter : false
pre : " <b> 5.4.3 </b> "
---

#### 1. Tổng quan Bước 5.4.3

Trong bước này, bạn sẽ tích hợp chứng chỉ bảo mật **SSL/TLS miễn phí từ AWS Certificate Manager (ACM)** vào phân phối **CloudFront CDN** để đảm bảo toàn bộ lưu lượng truy cập giao diện **TSL-SignMap React Admin Web** được mã hóa bảo mật ở cổng 443 (HTTPS).

- **Lưu ý:** Chứng chỉ ACM dành cho CloudFront CDN phải bắt buộc được khởi tạo tại AWS Region **us-east-1 (N. Virginia)**.

---

#### 2. Quy Trình Thực Hiện Chi Tiết

##### Bước 1: Yêu cầu chứng chỉ SSL/TLS trên AWS Certificate Manager (ACM)
1. Mở **AWS Certificate Manager Console**, đổi Region trên góc phải thành **us-east-1 (N. Virginia)**.
2. Bấm **Request a certificate** -> chọn **Request a public certificate** -> bấm **Next**.
3. Fully qualified domain name: Nhập tên miền ứng dụng (ví dụ: `admin.tsl-signmap.com` hoặc `*.tsl-signmap.com`).
4. Validation method: Chọn **DNS validation (recommended)**.
5. Key algorithm: Chọn **RSA 2048**.
6. Bấm **Request**.

##### Bước 2: Xác minh tên miền qua Amazon Route 53
1. Chọn chứng chỉ vừa tạo trong trang danh sách ACM.
2. Tại mục **Domains**, bấm nút **Create records in Route 53**.
3. Hệ thống tự động thêm bản ghi CNAME xác minh tên miền vào Hosted Zone trên Amazon Route 53.
4. Chờ 1 - 3 phút để trạng thái chứng chỉ chuyển sang **Issued (Đã cấp)**.

##### Bước 3: Đính kèm Chứng chỉ ACM vào CloudFront Distribution
1. Chuyển sang **AWS CloudFront Console**, chọn Distribution của bạn.
2. Tại thẻ **General**, cuộn xuống mục **Settings** -> chọn **Edit**.
3. **Alternate domain name (CNAME):** Nhập `admin.tsl-signmap.com`.
4. **Custom SSL certificate:** Chọn chứng chỉ ACM vừa được cấp (`admin.tsl-signmap.com`).
5. Minimum TLS version: Chọn **TLSv1.2_2021 (recommended)**.
6. Bấm **Save changes**.

---

#### 3. Kiểm Tra Kết Nối Bảo Mật

1. Mở terminal và thực thi lệnh cURL tới tên miền custom:
   ```bash
   curl -I https://admin.tsl-signmap.com
   ```
2. Kết quả trả về chứa các header bảo mật:
   - `HTTP/2 200`
   - `Server: CloudFront`
   - `X-Cache: Hit from cloudfront`
   - Biểu tượng khóa bảo mật HTTPS xuất hiện trên thanh địa chỉ trình duyệt.
