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
git clone https://github.com/ozanoguz/aws_tools.git
```

Access related folder and allow bash scripts that can be executed.

```
cd aws-tools
cd EKS_demo
chmod +x deploy.sh
chmod +x cleanup.sh
```
Run the script that deploys EKS cluster with required resources (VPC, subnets, IGW, SG etc.)

```
\.deploy.sh
```




## Step2: Deploy FortiGate PAYG Instance

sdgsgsdgds

## Step3: Prepare EKS Cluster for FortiGate Integration

sdgsgsdgds

## Step4: Connect FortiGate to EKS using SDN Connector

sdgsgsdgds

## Step5: South/North Egress Traffic Inspection by FortiGate

sdgsgsgds

