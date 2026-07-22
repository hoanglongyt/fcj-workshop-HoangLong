---
title: "Blog 1"
date: 2026-06-30
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# SESSION POLICIES TRONG AMAZON EKS POD IDENTITY

Amazon EKS Pod Identity vừa bổ sung tính năng **session policies**, cho phép bạn thu hẹp quyền IAM một cách linh hoạt và chính xác cho từng pod mà không cần tạo thêm nhiều IAM roles riêng biệt. Đây là bước tiến quan trọng giúp áp dụng nguyên tắc **least privilege** hiệu quả hơn trong môi trường Kubernetes quy mô lớn.

### Các điểm chính cần nắm:

* **Session policy là gì?**: Là một IAM policy inline được chỉ định khi tạo hoặc cập nhật một Pod Identity association.
* **Nguyên lý phân quyền hiệu quả**: Quyền thực tế (*Effective permissions*) = Giao (*Intersection*) giữa permissions của IAM role gốc và session policy. Do đó, session policy chỉ có thể thu hẹp chứ không thể mở rộng thêm quyền so với role gốc.
* **Tối ưu hóa số lượng IAM Role**: Giúp tránh tình trạng over-permissioning khi tái sử dụng chung một IAM role cho nhiều workloads khác nhau.
* **Hỗ trợ linh hoạt**: Hỗ trợ cả mô hình cùng tài khoản (*same-account*) và liên tài khoản (*cross-account*) thông qua IAM role chaining.
* **Tránh giới hạn Quota**: Giảm đáng kể số lượng IAM roles cần tạo và quản lý, tránh chạm mốc giới hạn quota IAM trong các Kubernetes cluster lớn.
* **Cấu hình đa dạng**: Thao tác dễ dàng thông qua AWS Management Console, AWS CLI hoặc AWS SDK khi liên kết Kubernetes ServiceAccount với IAM role.

Tính năng này đặc biệt hữu ích khi bạn có nhiều ứng dụng chạy trên cùng một IAM role nhưng cần giới hạn quyền khác nhau (ví dụ: một pod chỉ có quyền đọc từ một S3 bucket cụ thể, trong khi pod khác chỉ được phép gọi một số AWS API định sẵn).

---

### Link bài đăng Facebook
- **Đường dẫn bài viết**: [https://www.facebook.com/share/p/14mL1ERofZM/](https://www.facebook.com/share/p/14mL1ERofZM/)

---

### Hướng dẫn chi tiết cấu hình Session Policy cho EKS Pod Identity

#### 1. Tạo IAM Role chung cho Pod Identity
Tạo một IAM Role với Trust Policy cho phép dịch vụ EKS Pod Identity giả lập (`eks-pod-identity.amazonaws.com`):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "pods.eks.amazonaws.com"
      },
      "Action": [
        "sts:AssumeRole",
        "sts:TagSession"
      ]
    }
  ]
}
```

#### 2. Định nghĩa Session Policy khi tạo Pod Identity Association
Sử dụng AWS CLI để tạo association giữa Kubernetes ServiceAccount và IAM Role, đồng thời truyền inline Session Policy để thu hẹp quyền:

```bash
aws eks create-pod-identity-association \
  --cluster-name my-eks-cluster \
  --namespace default \
  --service-account my-app-sa \
  --role-arn arn:aws:iam::123456789012:role/MySharedPodRole \
  --service-account-association-options '{"sessionPolicy": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Action\":[\"s3:GetObject\"],\"Resource\":\"arn:aws:s3:::my-app-bucket/*\"}]}"}'
```

#### 3. Kiểm tra và xác nhận quyền hạn thực tế trong Pod
Khi ứng dụng trong Pod gửi request đến dịch vụ AWS, EKS Pod Identity Agent sẽ tự động cung cấp temporary credentials đã được áp dụng Session Policy. Pod sẽ chỉ truy cập được các tài nguyên được cho phép trong phạm vi Session Policy đã định nghĩa.