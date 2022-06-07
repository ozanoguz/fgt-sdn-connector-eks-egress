# FortiGate SDN Connector & AWS EKS N/S Hands-on-Lab
This document describes how to protect managed Kubernetes cluster on AWS platform using FortiGate-VM deployment for North/South traffic. This hands-on-lab consists on following steps:

-	Creating AWS EKS cluster using script
-	Deploying FortiGate single-VM instance using AWS Marketplace
-	Preparing EKS Cluster for FortiGate integration
-	Connecting FortiGate to EKS
-	South/North egress traffic inspection through FortiGate

## Section1: Create AWS EKS Managed Kubernetes Cluster

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

## Section2: Deploy FortiGate PAYG Instance

First, we will create a key pair using EC2 service. Navigation path is "_AWS Console > Services > EC2 > Key Pairs > Create Key Pair_"

[Quick Access to Key Pairs](https://eu-west-1.console.aws.amazon.com/ec2/v2/home?region=eu-west-1#KeyPairs)

We will use AWS Marketplace to deploy single FortiGate-VM instance into cloud account. While we are logged in AWS console, click following link to start FortiGate deployment

[Deploy FortiGate PAYG Instance](https://aws.amazon.com/marketplace/pp/prodview-wory773oau6wq?sr=0-1&ref_=beagle&applicationId=AWSMPContessa)

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_FGT_SEARCH_MARKETPLACE.png>

Click "_Continue to Subscribe_" button on top right to continue:

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_FGT_MARKETPLACE.png>

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

IMAGE_LUNCH_FGT_DEPLOYMENT

When FortiGate deployment is ready, you can login FortiGate GUI with assigned public-IP using instance-id as admin password at once. Later, GUI will ask to change login password.

IMAGE_EC2_FGT_READY
 
IMAGE_FGT_LOGIN_GUI

## Section3: Prepare EKS Cluster for FortiGate Integration

First, we will prepare AWS Cloudshell to access EKS cluster by installing kubectl tool.

```
!
# locate to home folder
cd $user
# download kubectl to cloudshell
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/kubectl
!
# Apply execute permission
chmod +x ./kubectl
!
# Move the kubectl to different folder and add it to the path
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
!
# Update kubeconfig file to access EKS cluster
aws eks update-kubeconfig --name EKSdemocluster
```

## Section4: Connect FortiGate to EKS using SDN Connector

First, we will find out Kubernetes Master API URL created by EKS using cloudshell:

```
kubectl cluster-info
```
IMAGE_MASTER_API_URL

Resolve master URL above using a terminal/cmd prompt to find out Master API IP address that we will use for FortiGate Kubernetes SDN Connector:

IMAGE_NSLOOKUP
```
#Step1: Create required serviceaccount in EKSdemocluster
kubectl create serviceaccount fortigateconnector

#Step2: Create and apply clusterrole for SDN connector
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
 name: fgt-connector
rules:
- apiGroups: [""]
  resources: ["pods", "namespaces", "nodes" , "services"]
  verbs: ["get", "watch", "list"]
EOF

#Step3: Attach clusterrole to the service account
kubectl create clusterrolebinding fgt-connector --clusterrole=fgt-connector --serviceaccount=default:fortigateconnector

#Step4: Obtain required token that will be used during creating SDN connector

kubectl get secrets -o jsonpath="{.items[?(@.metadata.annotations['kubernetes\.io/service-account\.name']=='fortigateconnector')].data.token}"| base64 --decode
```

To create SDN connector on FortiGate, navigate the path on management GUI "_Security Fabric > External Connectors > Create New > Kubernetes_"

IMAGE_SDN_CONNECTOR_CREATE_NEW

Fill following fields accordingly:
Name: Any name can be given
Verify Certificate: Should be DISABLED
IP: Resolved IP of Master API URL using terminal/cmd_prompt
Secret token: Obtained on Step4 above

IMAGE_CREATE_SDN_CONNECTOR

Click OK

You can view objects imported from EKSdemocluster

IMAGE_OBJECTS


## Section5: South/North Egress Traffic Inspection by FortiGate

Let's create a dynamic address object using SDN connector capability. To do that, navigate the path using FortiGate management GUI "". 

Next, create an egress firewall policy using following parameters. This will ensure outgoing traffic is inspected and protected by FortiGate.

For creating egress traffic, We need to access bash of one of the deployed container pods using following command:

***
kubectl exec -it name_of_container_pod -- /bin/bash

Preparing pod to use network tools
apt-get update && apt-get install curl

While we have bash access, let's test couple of web-sites to test egress communication is traveling through FortiGate installation:

***
curl www.google.com

sdgsgsgds

