---
title: "Blog 1"
date: 2026-06-30
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# SESSION POLICIES IN AMAZON EKS POD IDENTITY

Amazon EKS Pod Identity has recently added the **session policies** feature, allowing you to narrow IAM permissions flexibly and precisely for each pod without needing to create many separate IAM roles. This is an important step forward that helps apply the principle of **least privilege** more effectively in large-scale Kubernetes environments.

### Key points to know:

* **What is a Session Policy?**: An inline IAM policy specified when creating or updating a Pod Identity association.
* **Effective Permissions Principle**: Effective permissions = intersection between the IAM role permissions and the session policy → the session policy can only narrow permissions, not expand them.
* **Optimizing IAM Roles**: Helps avoid over-permissioning when reusing a single IAM role for multiple workloads with different needs.
* **Flexible Support**: Supports both same-account and cross-account scenarios (via IAM role chaining).
* **Avoiding Quota Limits**: Significantly reduces the number of IAM roles that need to be created and managed, helping avoid hitting IAM quota limits in large clusters.
* **Multiple Configuration Methods**: Easily configured through the AWS Management Console, AWS CLI, or AWS SDK when creating an association between a Kubernetes ServiceAccount and an IAM role.

This feature is especially useful when you have many applications running on the same IAM role but need different permission restrictions (for example: one pod only reads a specific S3 bucket, another pod only calls certain APIs).

---

### Facebook Post Link
- **Post Link**: [https://www.facebook.com/share/p/14mL1ERofZM/](https://www.facebook.com/share/p/14mL1ERofZM/)

---

### Step-by-Step Guide: Configuring Session Policies in EKS Pod Identity

#### 1. Create a Shared IAM Role for Pod Identity
Create an IAM Role with a Trust Policy allowing the EKS Pod Identity service (`eks-pod-identity.amazonaws.com`) to assume the role:

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

#### 2. Define the Session Policy when Creating a Pod Identity Association
Use the AWS CLI to create an association between a Kubernetes ServiceAccount and the IAM Role, passing an inline Session Policy to restrict permissions:

```bash
aws eks create-pod-identity-association \
  --cluster-name my-eks-cluster \
  --namespace default \
  --service-account my-app-sa \
  --role-arn arn:aws:iam::123456789012:role/MySharedPodRole \
  --service-account-association-options '{"sessionPolicy": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Action\":[\"s3:GetObject\"],\"Resource\":\"arn:aws:s3:::my-app-bucket/*\"}]}"}'
```

#### 3. Verify Effective Permissions Inside the Pod
When the application inside the Pod makes requests to AWS services, the EKS Pod Identity Agent automatically supplies temporary credentials scoped by the Session Policy. The Pod will only be able to access resources explicitly permitted within the defined Session Policy.