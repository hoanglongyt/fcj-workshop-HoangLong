---
title : "Cấu hình VPC Endpoints (S3 & SageMaker) và Policies"
date : 2026-07-22
weight : 5
chapter : false
pre : " <b> 5.5 </b> "
---

#### 1. Tổng quan VPC Endpoints & Endpoint Policies

Trong hệ thống **TSL-SignMap**, để đảm bảo an toàn thông tin theo chuẩn **AWS Well-Architected Framework**, tất cả lưu lượng dữ liệu giữa các máy chủ EC2 Microservices với dịch vụ lưu trữ **Amazon S3** và mô hình nhận diện AI **Amazon SageMaker (YOLO)** đều được định tuyến thông qua các **VPC Endpoints (AWS PrivateLink)** mà không cần đi qua Internet công cộng.

- **S3 VPC Endpoint:** Cho phép `ContributionService` lưu trữ và truy xuất các tệp ảnh biển báo giao thông tải lên **S3 Media Bucket** một cách riêng tư và bảo mật.
- **SageMaker VPC Endpoint:** Cho phép `ApiGateway` và các Microservices gửi yêu cầu nhận diện ảnh đến mô hình **YOLO AI Endpoint** hoàn toàn trong mạng nội bộ VPC.
- **VPC Endpoint Policies (IAM Resource Policies):** Đính kèm trực tiếp vào VPC Endpoint để kiểm soát truy cập theo nguyên tắc đặc quyền tối thiểu (Least Privilege), ngăn chặn hành vi rò rỉ dữ liệu (Data Exfiltration) ra các bucket không được phép bên ngoài.

---

#### 2. Cấu hình Chính sách VPC Endpoint Policy (Restricting S3 Access)

Chính sách bên dưới đảm bảo các máy chủ trong VPC chỉ được phép đọc/ghi dữ liệu vào đúng bộ chứa **`tsl-signmap-media-bucket`**, các yêu cầu truy cập đến những S3 bucket khác từ dải mạng VPC sẽ tự động bị từ chối:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowTSLMediaBucketAccessOnly",
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::tsl-signmap-media-bucket",
        "arn:aws:s3:::tsl-signmap-media-bucket/*"
      ]
    }
  ]
}
```

---

#### 3. Các Bước Cấu Hình & Kiểm Tra Quyền Truy Cập

##### Bước 1: Đính kèm Policy vào S3 VPC Endpoint
1. Truy cập **AWS VPC Console** -> chọn mục **Endpoints**.
2. Chọn **S3 VPC Endpoint** đã khởi tạo (`s3-gwe`).
3. Chọn thẻ **Policy**, nhấn **Edit Policy**.
4. Dán đoạn JSON Policy giới hạn quyền ở trên và bấm **Save**.

##### Bước 2: Kiểm tra kết nối từ máy chủ EC2 Microservices
1. Sử dụng AWS Session Manager kết nối vào máy chủ `EC2 Microservices Instance` trong Private App Subnet.
2. Tải thử một tệp ảnh biển báo lên đúng S3 Media Bucket của hệ thống:
   ```bash
   aws s3 cp traffic_sign_001.jpg s3://tsl-signmap-media-bucket/
   ```
   *Kết quả:* Tệp được tải lên thành công qua dải mạng riêng tư.

3. Kiểm tra tính năng bảo mật bằng cách tải tệp lên một S3 bucket bên ngoài bất kỳ:
   ```bash
   aws s3 cp traffic_sign_001.jpg s3://external-unauthorized-bucket/
   ```
   *Kết quả:* Hệ thống trả về lỗi `AccessDenied` do VPC Endpoint Policy chặn truy cập.

---

#### 4. Tóm tắt

Bằng việc cấu hình **S3 VPC Endpoint** và **SageMaker VPC Endpoint** kết hợp với **VPC Endpoint Policies**, hệ thống TSL-SignMap đã bảo vệ an toàn toàn bộ luồng truyền tải dữ liệu ảnh biển báo và mô hình AI. Điều này ngăn chặn triệt để nguy cơ lộ dữ liệu ra Internet công cộng và đảm bảo tính tuân thủ bảo mật cho hệ thống.
