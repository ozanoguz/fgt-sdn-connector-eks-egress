# FortiGate SDN Connector & AWS EKS N/S Hands-on-Lab
This document describes how to protect managed Kubernetes cluster on AWS platform using FortiGate-VM deployment for North/South traffic. This hands-on-lab consists on following steps:

-	Creating AWS EKS cluster using script
-	Deploying FortiGate single-VM instance using AWS Marketplace
-	Preparing EKS Cluster for FortiGate integration
-	Connecting FortiGate to EKS
-	South/North egress traffic inspection through FortiGate

## Step1: Create AWS EKS Managed Kubernetes Cluster

To create EKS cluster, we will use a bash script on AWS cloudshell. To access AWS cloudshell, after accessing AWS console GUI click the button on top right as shown below:

IMAGE_ACCESS_CLOUDSHELL

When you access Cloudshell CLI screen, clone following GitHub repo to your shell:

```
cd $user
git clone https://github.com/ozanoguz/aws_tools.git
```

Access related folder and allow bash scripts that can be executed.

```
cd $user
cd aws_tools
cd EKS_demo
chmod +x deploy.sh
chmod +x cleanup.sh
```
Run the script that deploys EKS cluster with required resources (VPC, subnets, IGW, SG etc.)

```
\.deploy.sh
```

Deployment will take around 15 mins. You should see following when it successfully finishes:

IMAGE_EKS_READY

## Step2: Deploy FortiGate PAYG Instance

We will use AWS Marketplace to deploy single FortiGate-VM instance into cloud account. While we are logged in AWS console, click here to start FortiGate deployment : https://aws.amazon.com/marketplace/pp/prodview-wory773oau6wq?sr=0-1&ref_=beagle&applicationId=AWSMPContessa

IMAGE

Click "Continue to Subscribe" button on top right to continue:

IMAGE

Click "Continue to Configuration" button on top right to continue:

IMAGE

Choose "7.0.5" from "Software version" and "EU (Ireland)" from "Region" dropdown menu as shown below, then click "Continue to Launch" button on top 
right.

IMAGE



## Step3: Prepare EKS Cluster for FortiGate Integration

sdgsgsdgds

## Step4: Connect FortiGate to EKS using SDN Connector

sdgsgsdgds

## Step5: South/North Egress Traffic Inspection by FortiGate

sdgsgsgds

