[AWS EKS.txt](https://github.com/user-attachments/files/26447093/AWS.EKS.txt)
🚀 What is Amazon EKS?

Amazon Web Services EKS (Elastic Kubernetes Service) is a managed Kubernetes service.

👉 Instead of installing Kubernetes manually (control plane, etcd, API server), AWS handles it for you.

🔹 Simple Definition (INTERVIEW)

👉 “EKS is a managed Kubernetes service by AWS where AWS manages the control plane, and we manage worker nodes and applications.”

🧠 ARCHITECTURE (IMPORTANT)
4
🔹 2 Main Parts
1️⃣ Control Plane (Managed by AWS)
API Server
etcd
Scheduler
Controller Manager

👉 You don’t manage this ❌

2️⃣ Worker Nodes (Managed by YOU)
EC2 instances
Run Pods
Part of Node Group

👉 You manage scaling, instance type, etc ✅

🔥 HOW YOUR TERRAFORM WORKS (STEP BY STEP)

Now I explain YOUR code like real-time project 👇

🟢 STEP 1: Provider
provider "aws" {
  region = "ap-south-1"
}

👉 Tells Terraform:

Use AWS
Region = Mumbai
🟢 STEP 2: VPC (NETWORK)
resource "aws_vpc" "eks_vpc" {
  cidr_block = "10.0.0.0/16"
}

👉 Creates private network
👉 All EKS resources run inside this

🟢 STEP 3: Subnets
aws_subnet.subnet_1
aws_subnet.subnet_2

👉 Why 2 subnets?

High availability
Different AZ (ap-south-1a, 1b)

👉 Interview point:
“EKS requires at least 2 subnets in different AZs”

🟢 STEP 4: Internet Gateway
aws_internet_gateway

👉 Gives internet access to nodes

🟢 STEP 5: Route Table
0.0.0.0/0 → IGW

👉 Allows outbound internet

🟢 STEP 6: IAM ROLE (CLUSTER)
eks_cluster_role

👉 Permissions for EKS control plane

🟢 STEP 7: EKS CLUSTER
resource "aws_eks_cluster"

👉 Creates Kubernetes cluster

🟢 STEP 8: NODE ROLE
node_role

👉 EC2 instances permissions:

Pull images
Join cluster
🟢 STEP 9: NODE GROUP
aws_eks_node_group

👉 Creates worker nodes

Instance type → t3.micro
Scaling → 1 to 2 nodes
🔄 COMPLETE FLOW (REAL TIME)
🔁 Flow:
Write Terraform
terraform init
terraform apply
AWS creates:
VPC
Subnets
EKS Cluster
Node Group
Configure kubeconfig
Use kubectl
Deploy apps
⚠️ IMPORTANT COMMAND (YOU MUST RUN)
aws eks update-kubeconfig --region ap-south-1 --name my-eks-cluster

👉 Without this:
❌ kubectl will NOT connect

✅ VERIFY
kubectl get nodes

👉 Output:

ip-10-0-x-x   Ready
🔥 INTERVIEW QUESTIONS (VERY IMPORTANT)
❓ Why EKS?
Managed Kubernetes
No control plane management
High availability
❓ Why 2 subnets?
Multi-AZ deployment
Fault tolerance
❓ Difference EKS vs self-managed Kubernetes?
Feature	EKS	Manual Kubernetes
Control plane	AWS manages	You manage
Setup	Easy	Complex
Maintenance	Low	High
❓ What is Node Group?

👉 Collection of EC2 instances running pods

🚀 NEXT LEVEL (WHAT YOU SHOULD LEARN NEXT)

I suggest next steps:

👉 Deploy NGINX on EKS
👉 Setup Ingress Controller
👉 Connect with ArgoCD (your skill 🔥)
👉 CI/CD pipeline

If you want, I’ll give you FULL REAL-TIME PROJECT

👉 Terraform + EKS + ArgoCD + App Deployment (resume ready)

Just say: "give project" 💥

hello give me with all one shot code also

Got it Suresh 👍 — you want everything in ONE FILE + ready-to-run + real-time usable 🚀
I’ll give you:

✅ Complete Terraform (EKS setup)
✅ Commands
✅ kubectl setup
✅ End-to-end flow

🚀 ONE-SHOT TERRAFORM FILE (COPY & USE)

👉 Save this as: main.tf

Writing

provider "aws" {
region = "ap-south-1"
}

########################

VPC

########################
resource "aws_vpc" "eks_vpc" {
cidr_block = "10.0.0.0/16"

tags = {
Name = "eks-vpc"
}
}

########################

SUBNETS

########################
resource "aws_subnet" "subnet_1" {
vpc_id = aws_vpc.eks_vpc.id
cidr_block = "10.0.1.0/24"
availability_zone = "ap-south-1a"
map_public_ip_on_launch = true

tags = {
Name = "eks-subnet-1"
}
}

resource "aws_subnet" "subnet_2" {
vpc_id = aws_vpc.eks_vpc.id
cidr_block = "10.0.2.0/24"
availability_zone = "ap-south-1b"
map_public_ip_on_launch = true

tags = {
Name = "eks-subnet-2"
}
}

########################

INTERNET GATEWAY

########################
resource "aws_internet_gateway" "igw" {
vpc_id = aws_vpc.eks_vpc.id
}

########################

ROUTE TABLE

########################
resource "aws_route_table" "rt" {
vpc_id = aws_vpc.eks_vpc.id

route {
cidr_block = "0.0.0.0/0"
gateway_id = aws_internet_gateway.igw.id
}
}

########################

ROUTE ASSOCIATION

########################
resource "aws_route_table_association" "a1" {
subnet_id = aws_subnet.subnet_1.id
route_table_id = aws_route_table.rt.id
}

resource "aws_route_table_association" "a2" {
subnet_id = aws_subnet.subnet_2.id
route_table_id = aws_route_table.rt.id
}

########################

IAM ROLE - CLUSTER

########################
resource "aws_iam_role" "eks_cluster_role" {
name = "eks-cluster-role"

assume_role_policy = jsonencode({
Version = "2012-10-17"
Statement = [{
Action = "sts"
Effect = "Allow"
Principal = {
Service = "eks.amazonaws.com"
}
}]
})
}

resource "aws_iam_role_policy_attachment" "eks_cluster_policy" {
role = aws_iam_role.eks_cluster_role.name
policy_arn = "arn:aws:iam::aws/AmazonEKSClusterPolicy"
}

########################

EKS CLUSTER

########################
resource "aws_eks_cluster" "eks" {
name = "my-eks-cluster"
role_arn = aws_iam_role.eks_cluster_role.arn

vpc_config {
subnet_ids = [
aws_subnet.subnet_1.id,
aws_subnet.subnet_2.id
]
}

depends_on = [
aws_iam_role_policy_attachment.eks_cluster_policy
]
}

########################

IAM ROLE - NODE

########################
resource "aws_iam_role" "node_role" {
name = "eks-node-role"

assume_role_policy = jsonencode({
Version = "2012-10-17"
Statement = [{
Action = "sts"
Effect = "Allow"
Principal = {
Service = "ec2.amazonaws.com"
}
}]
})
}

resource "aws_iam_role_policy_attachment" "worker_node_policy" {
role = aws_iam_role.node_role.name
policy_arn = "arn:aws:iam::aws/AmazonEKSWorkerNodePolicy"
}

resource "aws_iam_role_policy_attachment" "cni_policy" {
role = aws_iam_role.node_role.name
policy_arn = "arn:aws:iam::aws/AmazonEKS_CNI_Policy"
}

resource "aws_iam_role_policy_attachment" "ecr_policy" {
role = aws_iam_role.node_role.name
policy_arn = "arn:aws:iam::aws/AmazonEC2ContainerRegistryReadOnly"
}

########################

NODE GROUP

########################
resource "aws_eks_node_group" "nodes" {
cluster_name = aws_eks_cluster.eks.name
node_group_name = "eks-nodes"
node_role_arn = aws_iam_role.node_role.arn

subnet_ids = [
aws_subnet.subnet_1.id,
aws_subnet.subnet_2.id
]

scaling_config {
desired_size = 2
max_size = 2
min_size = 1
}

instance_types = ["t3.micro"]

depends_on = [
aws_iam_role_policy_attachment.worker_node_policy,
aws_iam_role_policy_attachment.cni_policy,
aws_iam_role_policy_attachment.ecr_policy
]
}
