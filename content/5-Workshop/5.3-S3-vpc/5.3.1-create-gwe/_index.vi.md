---
title : "Tạo một Gateway Endpoint"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

1. Mở [Amazon VPC console](https://us-east-1.console.aws.amazon.com/vpc/home?region=us-east-1#Home:)
2. Trong thanh điều hướng, chọn **Endpoints**, click **Create Endpoint**:

{{% notice note %}}
Bạn sẽ thấy các điểm cuối VPC hiện có hỗ trợ AWS Systems Manager (SSM).
{{% /notice %}}

3. Trong Create endpoint console:
+ Đặt tên cho endpoint: `s3-gwe`
+ Trong service category, chọn **AWS services**
+ Trong **Services**, gõ "s3" trong hộp tìm kiếm và chọn dịch vụ với loại **gateway**
+ Đối với VPC, chọn **VPC Cloud** từ drop-down menu.
+ Đối với Route tables, chọn bảng định tuyến đã liên kết với 2 subnets.
+ Đối với Policy, để tùy chọn mặc định là Full access để cho phép toàn quyền truy cập vào dịch vụ.
+ Không thêm tag vào VPC endpoint.
+ Click **Create endpoint**, click x sau khi nhận được thông báo tạo thành công.
