---
title : "Cấu hình Tường lửa AWS WAF & Giám sát Tầng Edge"
date : 2026-07-22 
weight : 4
chapter : false
pre : " <b> 5.4.4 </b> "
---

#### 1. Tổng quan Bước 5.4.4

Trong bước này, bạn sẽ triển khai tường lửa ứng dụng web **AWS WAF (Web Application Firewall)** tại vùng **Global Edge Services** đính kèm trực tiếp vào phân phối **CloudFront CDN**.

- **Mục tiêu:** Bảo vệ giao diện React Admin Web và các API khỏi các cuộc tấn công mạng nguy hiểm như SQL Injection, Cross-Site Scripting (XSS), HTTP Flood / DDoS và lọc bot độc hại.

---

#### 2. Quy Trình Thực Hiện Chi Tiết

##### Bước 1: Khởi tạo AWS WAF Web ACL
1. Mở **AWS WAF & Shield Console**, chọn mục **Web ACLs** ở menu bên trái.
2. Resource type: Chọn **Global resources (CloudFront distribution)**.
3. Bấm **Create web ACL**.
4. Name: Nhập `tsl-signmap-edge-waf-acl`.
5. CloudFront distributions to associate: Chọn CloudFront Distribution đã tạo ở bước 5.4.2.
6. Bấm **Next**.

##### Bước 2: Thêm các Managed Rule Sets chống tấn công
1. Tại mục **Add rules and rule groups**, bấm **Add rules** -> chọn **Add managed rule groups**.
2. **AWS managed rule groups:**
   - Bật **Core rule set (CRS):** Bảo vệ các lỗ hổng web phổ biến OWASP Top 10.
   - Bật **Known bad inputs:** Chặn các request chứa mã độc hại.
   - Bật **Amazon IP reputation list:** Chặn các địa chỉ IP xấu trong danh sách đen của AWS.
3. Bấm **Add rules**.

##### Bước 3: Thêm Rate-based Rule chống tấn công DDoS / Brute-force
1. Bấm **Add rules** -> chọn **Add my own rules and rule groups**.
2. Rule type: Chọn **Rate-based rule**.
3. Rule name: Nhập `PreventDDoSAndBruteForce`.
4. Rate limit: Nhập `2000` (giới hạn tối đa 2000 request từ 1 địa chỉ IP trong 5 phút).
5. Action: Chọn **Block**.
6. Bấm **Add rule**.

##### Bước 4: Hoàn tất cấu hình và đính kèm Web ACL
1. Đặt thứ tự ưu tiên các rules và bấm **Next**.
2. Metric name: Giữ mặc định để đẩy log giám sát sang **Amazon CloudWatch**.
3. Bấm **Create web ACL**.

---

#### 3. Kiểm Tra & Giám Sát Lưu Lượng Edge

1. Thử nghiệm gửi request giả lập tấn công XSS tới tên miền CloudFront:
   ```bash
   curl -I "https://admin.tsl-signmap.com/?q=<script>alert('xss')</script>"
   ```
2. **Kết quả:** AWS WAF chặn request và trả về lỗi `HTTP 403 Forbidden`.

3. Mở **Amazon CloudWatch Console** -> chọn **Metrics** -> xem biểu đồ `AllowedRequests` vs `BlockedRequests` để theo dõi lưu lượng truy cập tầng Global Edge theo thời gian thực.