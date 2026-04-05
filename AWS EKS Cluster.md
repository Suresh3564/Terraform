# 🚀 Amazon EKS (Elastic Kubernetes Service) – One-Shot Complete Guide

## 👨‍💻 Author

Suresh Kumar
Linux | DevOps | Cloud Engineer

---

# 🧠 1. What is Amazon EKS?

Amazon EKS is a **managed Kubernetes service by AWS**.

👉 AWS manages:

* Control Plane (API server, etcd, scheduler)

👉 You manage:

* Worker nodes (EC2)
* Applications (Pods, Deployments)

---

## 🔹 Interview Definition

EKS is a managed Kubernetes service where AWS manages the control plane, and we manage worker nodes and workloads.

---

# 🧩 2. EKS Architecture

## 🔹 Control Plane (AWS Managed)

* API Server
* etcd
* Scheduler
* Controller Manager

👉 No need to manage

---

## 🔹 Worker Nodes (User Managed)

* EC2 instances
* Run Pods
* Part of Node Group

---

# 🔥 3. Real-Time Flow

```bash
Terraform → AWS → EKS Cluster → Node Group → kubectl → Deploy App
```

---

# 📁 4. Setup Project

```bash
mkdir eks-terraform
cd eks-terraform
vi main.tf
```

---

# ⚙️ 5. FULL TERRAFORM CODE (ONE-SHOT)

```hcl
provider "aws" {
  region = "ap-south-1"
}

########################
# VPC
########################
resource "aws_vpc" "eks_vpc" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "eks-vpc"
  }
}

########################
# SUBNETS
########################
resource "aws_subnet" "subnet_1" {
  vpc_id                  = aws_vpc.eks_vpc.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-south-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "eks-subnet-1"
  }
}

resource "aws_subnet" "subnet_2" {
  vpc_id                  = aws_vpc.eks_vpc.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = "ap-south-1b"
  map_public_ip_on_launch = true

  tags = {
    Name = "eks-subnet-2"
  }
}

########################
# INTERNET GATEWAY
########################
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.eks_vpc.id
}

########################
# ROUTE TABLE
########################
resource "aws_route_table" "rt" {
  vpc_id = aws_vpc.eks_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}

########################
# ROUTE ASSOCIATION
########################
resource "aws_route_table_association" "a1" {
  subnet_id      = aws_subnet.subnet_1.id
  route_table_id = aws_route_table.rt.id
}

resource "aws_route_table_association" "a2" {
  subnet_id      = aws_subnet.subnet_2.id
  route_table_id = aws_route_table.rt.id
}

########################
# IAM ROLE - CLUSTER
########################
resource "aws_iam_role" "eks_cluster_role" {
  name = "eks-cluster-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "eks.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "eks_cluster_policy" {
  role       = aws_iam_role.eks_cluster_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
}

########################
# EKS CLUSTER
########################
resource "aws_eks_cluster" "eks" {
  name     = "my-eks-cluster"
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
# IAM ROLE - NODE
########################
resource "aws_iam_role" "node_role" {
  name = "eks-node-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "worker_node_policy" {
  role       = aws_iam_role.node_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy"
}

resource "aws_iam_role_policy_attachment" "cni_policy" {
  role       = aws_iam_role.node_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy"
}

resource "aws_iam_role_policy_attachment" "ecr_policy" {
  role       = aws_iam_role.node_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"
}

########################
# NODE GROUP
########################
resource "aws_eks_node_group" "nodes" {
  cluster_name    = aws_eks_cluster.eks.name
  node_group_name = "eks-nodes"
  node_role_arn   = aws_iam_role.node_role.arn

  subnet_ids = [
    aws_subnet.subnet_1.id,
    aws_subnet.subnet_2.id
  ]

  scaling_config {
    desired_size = 2
    max_size     = 2
    min_size     = 1
  }

  instance_types = ["t3.micro"]

  depends_on = [
    aws_iam_role_policy_attachment.worker_node_policy,
    aws_iam_role_policy_attachment.cni_policy,
    aws_iam_role_policy_attachment.ecr_policy
  ]
}
```

---

# ⚙️ 6. Commands (Run Step-by-Step)

```bash
terraform init
terraform plan
terraform apply --auto-approve
```

⏳ Wait ~10–15 minutes

---

# 🔑 7. Connect to Cluster

```bash
aws eks update-kubeconfig --region ap-south-1 --name my-eks-cluster
```

---

# ✅ 8. Verify

```bash
kubectl get nodes
```

---

# 🚀 9. Deploy Application

```bash
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --type=NodePort --port=80
kubectl get svc
```

---

# 🔥 10. Interview Questions

## Why EKS?

* Managed Kubernetes
* High availability
* No control plane management

---

## Why 2 subnets?

* Multi-AZ
* Fault tolerance

---

## What is Node Group?

👉 Group of EC2 instances running pods

---

# ⚠️ 11. Cleanup (IMPORTANT)

```bash
terraform destroy --auto-approve
```

---

# 🎯 12. Interview One-Liner

“I have created an EKS cluster using Terraform, including VPC, subnets, IAM roles, and node groups, and deployed applications using kubectl.”

---

# 🚀 Final Conclusion

This is a complete end-to-end EKS setup using Terraform for real-time DevOps use and interviews.

---
