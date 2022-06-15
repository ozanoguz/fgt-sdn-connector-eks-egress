# FortiGate SDN Connector & AWS EKS N/S Hands-on-Lab
This document describes how to protect managed Kubernetes cluster on AWS platform using FortiGate-VM deployment for North/South traffic. Same use case applies on other managed Kubernetes cluster deployments, such as AKS on Azure or GKE on GCP. This hands-on-lab consists on following steps:

-	[Section 1: Creating AWS EKS cluster using bash script](https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/README.md#section-1-creating-aws-eks-cluster-using-bash-script)
- [Section 2: Deploying FortiGate single-VM instance using AWS Marketplace](https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/README.md#section-2-deploying-fortigate-single-vm-instance-using-aws-marketplace)
-	[Section 3: Preparing EKS Cluster & Deploy Simple Application](https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/README.md#section-3-preparing-eks-cluster--deploy-simple-application)
-	[Section 4: Connecting FortiGate to EKS](https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/README.md#section-4-connecting-fortigate-to-eks)
-	[Section 5: South/North egress traffic inspection through FortiGate](https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/README.md#section-5-southnorth-egress-traffic-inspection-through-fortigate)
-	[Section 6: (Optional) Automation by scaling-up NGINX deployment](https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/README.md#section-6-optional-automation-by-scaling-up-nginx-deployment)

Here is the topology diagram showing all components that will be deployed:

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_TOPOLOGY_DIAGRAM.png width="500"/>

## Section 1: Creating AWS EKS Cluster Using Bash Script

To create EKS cluster, we will use a bash script on AWS cloudshell. To access AWS cloudshell, after logging into AWS console GUI click the button on top right as shown below. Make sure "Ireland" region is selected. 

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_CLOUDSHELL_ACCESS.png>

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
Bash script will create required resources below:
- VPC
- 3x subnets (2x for EKS, 1x for FortiGate)
- Internet Gateway
- Route table
- Security group
- IAM role & policy
- EKS Cluster
- EKS Nodegroup

Run the script using following command. 

```
/.deploy.sh
```

Deployment will take around 15 mins. You can check if EKS cluster is successfully deployed on AWS Console using path "_Services > Elastic Kubernetes Service_"

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_EKS_READY.png width='400'/>

## Section 2: Deploying FortiGate Single-VM Instance Using AWS Marketplace

In this section, we will deploy FortiGate-VM using PAYG licensing model. FortiGate-VM EC2 instance can be deployed in many ways (CloudFormation templates, using EC2 service etc). For this lab, we will use AWS Marketplace. FortiGate-VM instance will be deployed in subnet named "_FortiGateSubnet_" which is created by bash script above.

### Step 2.1: Create EC2 Key Pair 
First, we will create a key pair using EC2 service. Navigation path is "_AWS Console > Services > EC2 > Key Pairs > Create Key Pair_" or you can click quick access link below.

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_KEYPAIR.png>

[Quick Access to Key Pairs](https://eu-west-1.console.aws.amazon.com/ec2/v2/home?region=eu-west-1#KeyPairs)

### Step 2.2: Deploy FortiGate PAYG Instance
We will use AWS Marketplace to deploy single FortiGate-VM instance into cloud account. While we are logged in AWS console, click following link to start FortiGate deployment.

[Deploy FortiGate PAYG Instance](https://aws.amazon.com/marketplace/pp/prodview-wory773oau6wq?sr=0-1&ref_=beagle&applicationId=AWSMPContessa)

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_FGT_SEARCH_MARKETPLACE.png width="800"/>

Click "_Continue to Subscribe_" button on top right to continue:

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_FGT_MARKETPLACE.png width="800"/>

Click "_Continue to Configuration_" button on top right to continue:

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_FGT_CLICK_CONTINUE.png width="700"/>

Choose "_7.0.5_" from "_Software version_" and "_EU (Ireland)_" from "_Region_" dropdown menu as shown below, then click "_Continue to Launch_" button on top right.

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_FGT_SELECT_VERSION.png width="400"/>

Select following from "_Configuration Details_" screen as below. Note that VPC-ID/subnet-ID will be the ones created by EKS script

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_FGT_OPTIONS.png width="600"/>

To view VPC-ID: "_AWS Console > Services > VPC > Your VPCs > copy the VPC-ID value of "EKSdemo_"

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_VPC_ID.png width="400"/>

To view Subnet-ID: "_AWS Console > Services > VPC > Subnets > copy the subnet-ID value of "FortiGateSubnet_"

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_SUBNET_ID.png width="300"/>

After choosing "_Create New Based on Seller Settings_", give a name to security group:

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_FGT_SG.png width="600"/>

Select the Key Pair we created above:

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_SELECT_KEYPAIR.png width="600"/>

Click "Lunch" on bottom right

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_LUNCH_FGT.png width="100"/>

When FortiGate EC2 status is "_running_", you can login FortiGate GUI with assigned public-IP using instance-id as admin password at once. Later, GUI will ask to change login password. FortiGate public IP can be found using path "_Services > EC2 > Instances > select FortiGate-VM_"

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_FGT_PUBLIC_IP.png>
 
We will use instance-id to login FortiGate GUI once. After first successfull login, FortiGate will ask to change admin password.

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_FGT_FIRST_LOGIN.png width="400"/>

Before proceeding below, do not forget to disable source & destination check on FortiGate-VM network interface. Navigation path on AWS console is "_Services > EC2 > select FortiGate-EC2 > Networking > Network Interfaces > select NIC > Actions > Change source/destination check_"

Check box should NOT be selected as shown below:

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_SRCDST_DISABLE.png>

## Section 3: Preparing EKS Cluster & Deploy Simple Application

In this section, we will prepare AWS Cloudshell for our usage and deploy a simple NGINX application for tests. Last step will be disabling NAT feature on AWS CNI. 

### Step 3.1: Prepare AWS Cloudshell

Let's prepare AWS Cloudshell to access EKS cluster by installing kubectl tool and required authentication.

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
We can check node and pod status of EKS cluster using following commands.

```
kubectl get nodes -o wide
kubectl get pods -A -o wide
```
<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_NODEPODSTATUS.png>

### Step 3.2: Deploy simple NGINX application

Using AWS cloudshell, deploy a simple NGINX application that consists of 2 pods using following manifest:

```
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
EOF
```
You can check the deployment status using following command:

```
kubectl get pods -n default -o wide
```
<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_SIMPLE_APP.png>

### Step 3.3: Disable NAT on AWS CNI

Since we are using AWS CNI, egress traffic directed to Internet will be NAT'ted by CNI. That way, for outgoing traffic initiated from pods, we will see only node IP of deployed pods in FortiGate traffic logs. We can disable NAT feature of AWS CNI using following command in cloudshell

```
kubectl set env daemonset -n kube-system aws-node AWS_VPC_K8S_CNI_EXTERNALSNAT=true
```

You should see following outputon cloudshell:

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_DISABLE_SNAT.png>

### Step 3.4: Manipulate Routing within EKS VPC

When we create subnets within VPC, all use main routing table. For re-directing outgoing traffic to FortiGate interface, create new route-table using AWS console ("_Services > VPC > Route Tables > Create route table_")

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_ROUTE_TABLE.png width="600"/>

Add a default route by clicking "Edit routes" with a target pointing FortiGate-VM NIC-ID. FortiGate-VM's NIC ID can be found under networking settings of FortiGate EC2.

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_ADD_ROUTE.png>

Then, associate this route table to EKS subnets by choosing "_Subnet Associations > Edit Subnet Associations_":

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_RT_ASSOCIATE.png> 

## Section 4: Connecting FortiGate to EKS

### Step 1: Obtain Master API IP address

First, we will find out Kubernetes Master API URL created by EKS using cloudshell:

```
kubectl cluster-info
```
<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_KUBECTL_CLUSTERINFO.png>

Resolve master URL above using a terminal/cmd prompt to find out Master API IP address that we will use for FortiGate Kubernetes SDN Connector:

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_NSLOOKUP.png width="700"/>

### Step 2: Create service account in EKS Cluster

```
#create service account
kubectl create serviceaccount fortigateconnector

#create and apply clusterrole for SDN connector
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

#attach clusterrole to the service account
kubectl create clusterrolebinding fgt-connector --clusterrole=fgt-connector --serviceaccount=default:fortigateconnector

#obtain required token that will be used during creating SDN connector

kubectl get secrets -o jsonpath="{.items[?(@.metadata.annotations['kubernetes\.io/service-account\.name']=='fortigateconnector')].data.token}"| base64 --decode
```
You can copy token to a text editor, because we will use that token to enable FortiGate SDN Connector.

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_TOKEN.png width="1500"/>

To create SDN connector on FortiGate, navigate the path on management GUI "_Security Fabric > External Connectors > Create New > Kubernetes_"

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_NEW_CONNECTOR.png width="400"/>

Fill following fields accordingly:

Name: Any name can be given
Verify Certificate: Should be DISABLED
IP: Resolved IP of Master API URL using terminal/cmd_prompt
Port: 443
Secret token: Obtained above

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_CONNECTOR_SETUP.png width="400"/>

Click OK

You can view objects imported from EKSdemocluster by right clicking the connector and selecting "_View Connector Objects_"

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_VIEW_OBJECTS.png width="400"/>

FortiGate is able to collect labels, services, namespaces and other useful metadata from EKS cluster as shown below:

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_CONNECTOR_OBJECTS.png width="400"/>

## Section 5: South/North Egress Traffic Inspection Through FortiGate

### Step 5.1: Configure SDN Connector

Let's create a dynamic address object using SDN connector capability. We can use label metadata of deployed for simple NGINX app. To create a dynamic object, navigate the path using FortiGate management GUI "_Policy & Objects > Addresses > Create New > Address_"

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_NEW_DYNAMIC_ADDRESS.png width="400"/>

### Step 5.2: Create Egress Firewall Policy#1

Next, create an egress firewall policy using following parameters. This will ensure outgoing traffic is inspected and protected by FortiGate. Navigation path is "_Policy & Objects > Firewall Policy > Create New_"

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_FIREWALL_POLICY.png width="400"/>

Make sure you enabled logging for all sessions:

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_FWPOLICY_LOGGING.png width="400"/>

### Step 5.3: Create Egress Firewall Policy#2

Before proceeding, add another firewall policy to allow worker nodes' Internet connectivity as shown below:

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_FGT_ALLOW_ALL.png width="400"/>

### Step 5.4: Create Egress Traffic from POD

For creating egress traffic, We need to access bash of one of the deployed container pods using following command. You can obtain the name of the NGINX pod using "kubectl get pods -n default -o wide" commmand. Once we are inside the pod, curl should be install to test egress traffic.

```
kubectl exec -it nginx-deployment-66b6c48dd5-56rjj -- /bin/bash

#install network tools and curl into nginx pod. Type "Y" to continue installation.
apt-get update && apt-get install curl
```

While we have bash access, let's test couple of web-sites to test egress communication is traveling through FortiGate instance:

```
curl www.google.com
curl www.fortinet.com
```
<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_CURL_FROM_POD.png>

### Step 5.5: Check Traffic Log

Let's check FortiGate traffic log using navigation path "_Log and Report > Forward Traffic_"

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_TRAFFIC_LOG.png width="800"/>

As it is shown above, FortiGate is able to log by NGINX pod IP and egress traffic is inspected using specific firewall policy.

## Section 6: (Optional) Automation by scaling-up NGINX deployment

By nature of Kubernetes world, applications can horizontally scale up/down by many reasons. 

### Step 6.1: Scale-up NGINX Deployment

Let's scale up our NGINX app using following command:

```
kubectl scale --replicas=3 deployment nginx-deployment
```
<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_SCALED_UP.png>

As the output showing above, our NGIX deployment is scaled up to 3 pods running. 

### Step 6.2: FortiGate Automation

Thanks to FortiGate SDN Connector, this chance will be automatically reflected in dynamic object and firewall policy. To see this change navigate through FortiGate management GUI "_Policy & Objects > Firewall Policy_" and hover your mouse on dynamic object we created before.

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_FGT_SCALED.png width="600"/>

Click "View Matched Addresses", that we can see FortiGate is aware about the change in NGINX deployment.

<img src=https://github.com/ozanoguz/fgt-sdn-connector-eks-egress/blob/main/images/IMAGE_FGT_THREE_PODS.png width="300"/>
