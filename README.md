# ScaleGrid-Assignment
Create 2 Kubernetes clusters in your preferred cloud platform. Deploy a single Postgresql cluster with nodes running in the 2 Kubernetes clusters in a High Availability setup through helm charts ONLY.


# create 2 Kubernetes clusters on AWS, the most common and scalable method is using Amazon EKS (Elastic Kubernetes Service).
You can also use tools like eksctl or Terraform for ease and repeatability. Here's a high-level overview and step-by-step using eksctl.


# Prerequisites
Before you start, ensure you have the following installed/configured:

- AWS CLI (aws configure)

- eksctl: Install eksctl

 - IAM permissions to create EKS clusters and related resources (VPC, IAM roles, etc.)

Step-by-Step: Creating Two EKS Clusters Using eksctl

# Step 1: Configure AWS CLI
 aws configure

Provide:
Access key ID
Secret access key
Region (e.g., us-west-2)
Output format (e.g., json)

# Step 2: Create First primary Cluster

eksctl create cluster \
  --name cluster-one \
  --region us-west-2 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --managed
  
# Step 3: Create Second secondary Cluster

eksctl create cluster \
  --name cluster-two \
  --region us-west-2 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --managed
  
**Each cluster will be in its own dedicated VPC by default. You can customize the networking or share VPCs if needed, but it's generally cleaner to isolate clusters unless you have specific networking requirements.**

**Optional: Use Terraform
If you prefer infrastructure-as-code, I can generate a Terraform script to create the two clusters instead.**

# Next Steps After Cluster Creation

## Use kubectl to interact with each cluster:
aws eks --region us-west-2 update-kubeconfig --name cluster-one
kubectl get nodes

## Switch between clusters by updating kubeconfig or using context switching:
kubectl config get-contexts
kubectl config use-context <context-name>


