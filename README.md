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

First, we will create a key pair using EC2 service. Navigation path is "_AWS Console > Services > EC2 > Key Pairs > Create Key Pair_"

[Quick Access to Key Pairs](https://eu-west-1.console.aws.amazon.com/ec2/v2/home?region=eu-west-1#KeyPairs)

We will use AWS Marketplace to deploy single FortiGate-VM instance into cloud account. While we are logged in AWS console, click following link to start FortiGate deployment

[Deploy FortiGate PAYG Instance](https://aws.amazon.com/marketplace/pp/prodview-wory773oau6wq?sr=0-1&ref_=beagle&applicationId=AWSMPContessa)

IMAGE

Click "_Continue to Subscribe_" button on top right to continue:

IMAGE

Click "_Continue to Configuration_" button on top right to continue:

IMAGE

Choose "_7.0.5_" from "_Software version_" and "_EU (Ireland)_" from "_Region_" dropdown menu as shown below, then click "_Continue to Launch_" button on top right.

IMAGE

Select following from "_Configuration Details_" screen as below. Note that VPC-ID/subnet-ID will be the ones created by EKS script

To view VPC-ID: "_AWS Console > Services > VPC > Your VPCs > copy the VPC-ID value of "EKSdemo_"
To view Subnet-ID: "_AWS Console > Services > VPC > Subnets > copy the subnet-ID value of "FortiGateSubnet_"

IMAGE

After choosing "_Create New Based on Seller Settings_", give a name to security group:

IMAGE

Select the Key Pair we created above:

IMAGE

Click "Lunch" on bottom right:


## Step3: Prepare EKS Cluster for FortiGate Integration

First, we will prepare AWS Cloudshell to access EKS cluster by installing kubectl tool.

```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
aws eks update-kubeconfig --name EKSdemocluster

```

## Step4: Connect FortiGate to EKS using SDN Connector

sdgsgsdgds

## Step5: South/North Egress Traffic Inspection by FortiGate

sdgsgsgds

