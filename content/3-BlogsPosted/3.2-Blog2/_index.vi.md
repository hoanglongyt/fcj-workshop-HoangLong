---
title: "Blog 2"
date: 2026-06-30
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# TỐI ƯU HÓA AUTOSCALING TRONG AMAZON EKS VỚI KARPENTER

Karpenter là giải pháp Kubernetes Node Autoscaler mã nguồn mở thế hệ mới do AWS phát triển, mang lại khả năng cấp phát tài nguyên (*node provisioning*) nhanh chóng, chính xác và tối ưu chi phí hơn rất nhiều so với Kubernetes Cluster Autoscaler truyền thống.

### Các điểm chính cần nắm:

* **Just-in-Time Provisioning**: Karpenter lắng nghe trực tiếp các Pod ở trạng thái *Unschedulable* và tự động tính toán nhu cầu tài nguyên (CPU, RAM, GPU, Architecture) để khởi tạo đúng loại EC2 instance phù hợp nhất chỉ trong vài giây.
* **Tối ưu chi phí vượt trội (Consolidation & Spot Instances)**: Tự động gom nhóm các Pod (*defragmentation*) và thay thế các node dư thừa tài nguyên bằng các instance nhỏ hơn hoặc tận dụng EC2 Spot Instances với chi phí thấp hơn tới 90%.
* **Bỏ qua Node Groups**: Karpenter tương tác trực tiếp với AWS EC2 Fleet API, không phụ thuộc vào Auto Scaling Groups (ASG) hay Managed Node Groups của EKS.
* **Tự động thay thế & Quản lý vòng đời**: Tự động thay thế node cũ khi hết hạn (*drift/expiry*), giúp đảm bảo an ninh và nâng cấp phiên bản node mượt mà không làm gián đoạn ứng dụng.

---

### Link bài đăng Facebook
- **Đường dẫn bài viết**: [https://www.facebook.com/share/p/19MaWGhgGr/](https://www.facebook.com/share/p/19MaWGhgGr/)

---

### Hướng dẫn triển khai Karpenter trên Amazon EKS Cluster

#### 1. Cấu hình IAM Role & ServiceAccount cho Karpenter
Karpenter cần IAM Role với quyền tạo/xóa EC2 Instances và gán IAM Instance Profile cho worker nodes:

```bash
# Tạo ServiceAccount và liên kết IAM Role cho Karpenter Controller
eksctl create iamserviceaccount \
  --cluster my-eks-cluster \
  --namespace karpenter \
  --name karpenter \
  --role-name my-eks-cluster-karpenter \
  --attach-policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy \
  --approve
```

#### 2. Cấu hình NodePool & EC2NodeClass
Định nghĩa tài nguyên node mà Karpenter được phép cấp phát:

```yaml
apiVersion: karpenter.sh/v1
kind: NodePool
metadata:
  name: default
spec:
  template:
    spec:
      requirements:
        - key: kubernetes.io/arch
          operator: In
          values: ["amd64", "arm64"]
        - key: karpenter.sh/capacity-type
          operator: In
          values: ["spot", "on-demand"]
        - key: karpenter.k8s.aws/instance-category
          operator: In
          values: ["c", "m", "r"]
      nodeClassRef:
        group: karpenter.k8s.aws
        kind: EC2NodeClass
        name: default
  disruption:
    consolidationPolicy: WhenEmptyOrUnderutilized
    consolidateAfter: 1m
---
apiVersion: karpenter.k8s.aws/v1
kind: EC2NodeClass
metadata:
  name: default
spec:
  amiSelectorTerms:
    - alias: al2023@latest
  role: KarpenterNodeRole-my-eks-cluster
  subnetSelectorTerms:
    - tags:
        karpenter.sh/discovery: my-eks-cluster
  securityGroupSelectorTerms:
    - tags:
        karpenter.sh/discovery: my-eks-cluster
```

#### 3. Kiểm tra tính năng Tự động Cấp phát (Provisioning)
Khi triển khai một Deployment có số lượng Replicas lớn hoặc yêu cầu tài nguyên cao, Karpenter sẽ lập tức nhận diện và tạo các EC2 instance phù hợp trong thời gian ngắn, giúp giảm thiểu thời gian chờ đợi của Pod.