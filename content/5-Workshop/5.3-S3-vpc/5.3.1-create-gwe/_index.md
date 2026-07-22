---
title : "Create a gateway endpoint"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

1. Open the [Amazon VPC console](https://us-east-1.console.aws.amazon.com/vpc/home?region=us-east-1#Home:)
2. In the navigation pane, choose **Endpoints**, then click **Create Endpoint**:

{{% notice note %}}
You will see existing VPC endpoints supporting AWS Systems Manager (SSM).
{{% /notice %}}

3. In the Create endpoint console:
+ Specify name of the endpoint: `s3-gwe`
+ In service category, choose **AWS services**
+ In **Services**, type `s3` in the search box and choose the service with type **gateway**
+ For VPC, select **VPC Cloud** from the drop-down.
+ For **Configure route tables**, select the route table associated with subnets.
+ **For Policy**, leave the default option, **Full Access**, to allow full access to the service.
+ Do not add a tag to the VPC endpoint at this time.
+ Click **Create endpoint**, then click x after receiving a successful creation message.