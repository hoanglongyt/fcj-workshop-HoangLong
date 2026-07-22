---
title: "Blog 2"
date: 2026-06-30
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# OPTIMIZING AUTOSCALING IN AMAZON EKS WITH KARPENTER

Karpenter is a next-generation open-source Kubernetes node autoscaler built by AWS. It provides fast, precise, and cost-effective node provisioning compared to the traditional Kubernetes Cluster Autoscaler.

### Key points to know:

* **Just-in-Time Provisioning**: Karpenter directly listens for *Unschedulable* pods and automatically calculates the exact resource requirements (CPU, RAM, GPU, Architecture) to launch the best-fit EC2 instances in seconds.
* **Exceptional Cost Optimization (Consolidation & Spot Instances)**: Automatically consolidates workloads (*defragmentation*) and replaces underutilized nodes with smaller instances or leverages EC2 Spot Instances for up to 90% cost savings.
* **Bypassing Auto Scaling Groups**: Karpenter directly communicates with the AWS EC2 Fleet API without relying on Auto Scaling Groups (ASG) or EKS Managed Node Groups.
* **Automated Node Lifecycle & Drift Management**: Automatically replaces nodes upon expiration or configuration drift, ensuring smooth security patching and node upgrades without disrupting application availability.

---

### Facebook Post Link
- **Post Link**: [https://www.facebook.com/share/p/19MaWGhgGr/](https://www.facebook.com/share/p/19MaWGhgGr/)

---

### Step-by-Step Guide: Deploying Karpenter on Amazon EKS Cluster

#### 1. Configure IAM Roles & ServiceAccount for Karpenter
Karpenter requires an IAM Role with permissions to launch/terminate EC2 Instances and attach IAM Instance Profiles to worker nodes:

```bash
# Create ServiceAccount and associate IAM Role for Karpenter Controller
eksctl create iamserviceaccount \
  --cluster my-eks-cluster \
  --namespace karpenter \
  --name karpenter \
  --role-name my-eks-cluster-karpenter \
  --attach-policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy \
  --approve
```

#### 2. Configure NodePool & EC2NodeClass
Define the node resources that Karpenter is allowed to provision:

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

#### 3. Verify Automatic Provisioning
When deploying a workload with high replica counts or large resource requests, Karpenter rapidly provisions suitable EC2 instances within seconds, dramatically reducing Pod pending time.