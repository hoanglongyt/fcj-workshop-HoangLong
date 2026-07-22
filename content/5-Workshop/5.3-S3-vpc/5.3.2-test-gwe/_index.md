---
title : "Test the Gateway Endpoint"
date : 2024-01-01 
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---

#### Create S3 bucket

1. Navigate to **S3 management console**
2. In the Bucket console, choose **Create bucket**
3. In **the Create bucket console**:
+ **Name the bucket**: choose a globally unique bucket name
+ Leave other fields as default
+ Scroll down and choose **Create bucket**
+ Successfully create S3 bucket.

#### Connect to EC2 with session manager

+ For this workshop, you will use **AWS Session Manager** to access **EC2 instances**. Session Manager is a fully managed AWS Systems Manager capability allowing browser-based shell access without opening inbound ports or managing SSH keys.

1. In the **AWS Management Console**, type `Systems Manager` in the search box and press **Enter**.
2. From the **Systems Manager** menu, select **Session Manager** under **Node Management**.
3. Click **Start Session**, and select **the EC2 instance** named **Test-Gateway-Endpoint**. 

{{% notice info %}}
This EC2 instance is running in "VPC Cloud" and will be used to test connectivity to Amazon S3 through the Gateway endpoint you created (s3-gwe).
{{% /notice %}}

**Session Manager** will open a new browser tab with a shell prompt: `sh-4.2 $`.

#### Create a file and upload to s3 bucket

1. Change to the ssm-user's home directory: `cd ~`
2. Create a test file: `fallocate -l 1G testfile.xyz`
3. Upload file to S3 bucket: `aws s3 cp testfile.xyz s3://your-bucket-name`

You have successfully uploaded the file to your S3 bucket.

#### Check object in S3 bucket

1. Navigate to S3 console.  
2. Click the name of your S3 bucket.
3. In the Bucket console, verify the uploaded object.

#### Section summary

Congratulations on completing access to S3 from VPC. You created a Gateway endpoint for Amazon S3 and verified file uploads using AWS CLI securely without traversing the Public Internet.
