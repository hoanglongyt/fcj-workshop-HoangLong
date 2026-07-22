---
title: "Các bài blogs đã đăng"
date: 2026-06-30
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Tại đây là phần liệt kê và giới thiệu các bài viết kỹ thuật (blogs) đã đăng trên cộng đồng [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj):

### [Blog 1 - SESSION POLICIES TRONG AMAZON EKS POD IDENTITY](3.1-Blog1/)
Blog này giới thiệu tính năng Session Policies mới trong Amazon EKS Pod Identity, cho phép thu hẹp quyền IAM một cách linh hoạt và chính xác cho từng Pod mà không cần tạo thêm nhiều IAM Roles riêng biệt, giúp áp dụng triệt để nguyên tắc Least Privilege trong các Kubernetes cluster lớn.

### [Blog 2 - TỐI ƯU HÓA AUTOSCALING TRONG AMAZON EKS VỚI KARPENTER](3.2-Blog2/)
Blog này giới thiệu giải pháp Karpenter - Kubernetes Node Autoscaler thế hệ mới do AWS phát triển. Karpenter giúp tự động cấp phát tài nguyên node (*Just-in-Time Provisioning*), gom nhóm Pod (*Consolidation*) và tận dụng EC2 Spot Instances để tối ưu hóa chi phí vận hành EKS cluster đến 90%.

### [Blog 3 - AMAZON BEDROCK GUARDRAILS - AN TOÀN VÀ BẢO MẬT CHO ỨNG DỤNG GENERATIVE AI](3.3-Blog3/)
Blog này giới thiệu giải pháp Amazon Bedrock Guardrails cung cấp các rào cản bảo vệ (*safeguards*) đa lớp cho ứng dụng Generative AI. Hỗ trợ lọc nội dung cấm (*Denied Topics*), làm mờ thông tin cá nhân (*PII Redaction*), và chống lại các kỹ thuật tấn công *Prompt Injection* / *Jailbreak*.