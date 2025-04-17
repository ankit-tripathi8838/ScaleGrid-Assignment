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

**we can also write Terraform module for the above tasks and add autoscaling & custom VPCs**



# Deploy a single  Postgresql cluster with nodes running in the 2 Kubernetes clusters in a High Availability setup through helm charts ONLY.

For deploy one single PostgreSQL cluster whose nodes are distributed across two separate Kubernetes clusters, with high availability (HA) — and using only Helm charts.

## **Architecture**
-Primary PostgreSQL cluster in Cluster one (active).
-Read-only replica or standby cluster in Cluster two (passive).
-Replication via WAL shipping or streaming replication.
-Manual or external failover (Route53, external DNS, app logic).

# Deployment Plan (Using Helm Charts Only)

We’ll use the Bitnami PostgreSQL Helm chart with replication enabled.

## 1. Add Bitnami Helm Repo (Once on your local machine)
'''
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
'''
## 2. Deploy Primary Cluster to Cluster one
'''
# Switch to cluster-one
'''
aws eks --region us-west-2 update-kubeconfig --name cluster-one

helm install pg-ha bitnami/postgresql-ha \
  --namespace postgres --create-namespace \
  --set auth.postgresPassword=supersecurepassword \
  --set auth.replicationPassword=replpassword \
  --set service.type=LoadBalancer '''
  
**This will create a highly available primary cluster with built-in replication between pods (within Cluster one only).**

## 3. Expose Primary Cluster's LoadBalancer

Get the LoadBalancer IP:
'''
kubectl get svc -n postgres pg-ha-postgresql-ha-pgpool
'''
Note the external IP — you’ll use it for replication in Cluster two.

## 4. Deploy Standby (Replica) to Cluster two

# Switch to cluster-two
'''
aws eks --region us-west-2 update-kubeconfig --name cluster-two

helm install pg-standby bitnami/postgresql \
  --namespace postgres --create-namespace \
  --set architecture=standalone \
  --set postgresql.replicationMode=slave \
  --set postgresql.primary.host=<external-ip-from-cluster-A> \
  --set postgresql.primary.port=5432 \
  --set postgresql.replication.user=replicator \
  --set postgresql.replication.password=replpassword \
  --set auth.postgresPassword=supersecurepassword \
  --set service.type=LoadBalancer'''



## Verify Replication
Check logs in the standby node on Cluster 2:

kubectl logs -l app.kubernetes.io/name=postgresql -n postgres


## Failover Strategy (Manual or External)
You’ll need to:

Monitor primary health (e.g., with Prometheus).

Redirect traffic (e.g., update Route53 or use external load balancer).

Promote standby manually (pg_ctl promote) if primary fails.
  
