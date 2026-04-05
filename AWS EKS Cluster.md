# 🚀 Amazon EKS with Terraform – Complete Guide

## 👨‍💻 Author

Suresh Kumar
Linux | DevOps | Cloud Engineer

---

# 🧠 1. What is Amazon EKS?

Amazon Web Services Elastic Kubernetes Service (EKS) is a **managed Kubernetes service**.

👉 AWS manages the control plane
👉 You manage worker nodes and applications

---

## 🔹 Simple Interview Definition

EKS is a managed Kubernetes service where AWS manages the control plane, and we manage worker nodes and workloads.

---

# 🧩 2. EKS Architecture

## 🔹 1. Control Plane (Managed by AWS)

* API Server
* etcd
* Scheduler
* Controller Manager

👉 Fully managed by AWS

---

## 🔹 2. Worker Nodes (Managed by You)

* EC2 instances
* Run Pods
* Part of Node Group

👉 You manage:

* Scaling
* Instance type
* Workloads

---

# 🔥 3. Terraform Workflow (Real-Time)

```bash
Write Terraform → terraform init → terraform apply → AWS Resources Created → Configure kubectl → Deploy apps
```

---

# ⚙️ 4. Step-by-Step Terraform Explanation

## 🔹 Step 1: Provider

```hcl id="p1"
provider "aws" {
  region = "ap-south-1"
}
```

👉 Uses AWS Mumbai region

---

## 🔹 Step 2: VPC

```hcl id="p2"
resource "aws_vpc" "eks_vpc" {
  cidr_block = "10.0.0.0/16"
}
```

👉 Creates private network

---

## 🔹 Step 3: Subnets

* 2 subnets (different AZs)

👉 Why?

* High availability
* Fault tolerance

💡 Interview point:
EKS requires at least 2 subnets in different availability zones

---

## 🔹 Step 4: Internet Gateway

👉 Enables internet access

---

## 🔹 Step 5: Route Table

```hcl id="p3"
0.0.0.0/0 → Internet Gateway
```

👉 Allows outbound traffic

---

## 🔹 Step 6: IAM Role (Cluster)

👉 Permissions for EKS control plane

---

## 🔹 Step 7: EKS Cluster

```hcl id="p4"
resource "aws_eks_cluster"
```

👉 Creates Kubernetes cluster

---

## 🔹 Step 8: Node Role

👉 Permissions for EC2 nodes:

* Pull images
* Join cluster

---

## 🔹 Step 9: Node Group

```hcl id="p5"
resource "aws_eks_node_group"
```

👉 Creates worker nodes

* Instance type → t3.micro
* Scaling → 1 to 2 nodes

---

# 🧪 5. Important Commands

## 🔹 Initialize

```bash id="c1"
terraform init
```

---

## 🔹 Plan

```bash id="c2"
terraform plan
```

---

## 🔹 Apply

```bash id="c3"
terraform apply --auto-approve
```

---

# 🔑 6. Configure kubectl (VERY IMPORTANT)

```bash id="c4"
aws eks update-kubeconfig --region ap-south-1 --name my-eks-cluster
```

👉 Without this:
❌ kubectl will not connect

---

# ✅ 7. Verify Cluster

```bash id="c5"
kubectl get nodes
```

✔ Output:

```
ip-10-0-x-x   Ready
```

---

# 🔥 8. Interview Questions

## ❓ Why EKS?

* Managed Kubernetes
* No control plane management
* High availability

---

## ❓ Why 2 subnets?

* Multi-AZ deployment
* Fault tolerance

---

## ❓ EKS vs Self-Managed Kubernetes

| Feature       | EKS         | Manual Kubernetes |
| ------------- | ----------- | ----------------- |
| Control Plane | AWS manages | You manage        |
| Setup         | Easy        | Complex           |
| Maintenance   | Low         | High              |

---

## ❓ What is Node Group?

👉 Collection of EC2 instances running pods

---

# 🚀 9. Complete Terraform (One-Shot)

👉 Save as: `main.tf`

```hcl id="full"
# (Full code same as your lab – trimmed here for readability)
provider "aws" {
  region = "ap-south-1"
}

resource "aws_vpc" "eks_vpc" {
  cidr_block = "10.0.0.0/16"
}

# Subnets, IGW, Route tables, IAM roles,
# EKS cluster and Node group configuration...
```

---

# ⚠️ 10. Important Note

Cluster creation takes:
⏳ 10–15 minutes

---

# 🏗️ 11. Real DevOps Flow

* Create infrastructure using Terraform
* Configure kubectl
* Deploy applications using Kubernetes
* Integrate with CI/CD (ArgoCD)

---

# 🎯 12. Next Level Learning

* Deploy Nginx on EKS
* Setup Ingress Controller
* Integrate ArgoCD
* Build CI/CD pipeline

---

# 🧾 Final Summary

✔ EKS = Managed Kubernetes
✔ Terraform = Infrastructure as Code
✔ Node Group = Worker nodes
✔ kubectl = Manage cluster
✔ Multi-AZ = High availability

---

# 🚀 Conclusion

Amazon EKS simplifies Kubernetes management by offloading control plane responsibilities while allowing full control over workloads and infrastructure.

---
                Lab2
# 🚀 One-Shot Amazon EKS Setup using Terraform (Full Working)

## 👨‍💻 Author

Suresh Kumar

---

# 🧠 1. Prerequisites

✔ AWS CLI configured
✔ Terraform installed
✔ IAM user with admin access

```bash
aws configure
terraform -v
```

---

# 📁 2. Create Project

```bash
mkdir eks-terraform
cd eks-terraform
vi main.tf
```

---

# 🧩 3. FULL WORKING TERRAFORM FILE (main.tf)

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

# ⚙️ 4. RUN COMMANDS (STEP-BY-STEP)

## 🔹 Initialize

```bash
terraform init
```

---

## 🔹 Plan

```bash
terraform plan
```

---

## 🔹 Apply

```bash
terraform apply --auto-approve
```

⏳ Takes ~10–15 minutes

---

# 🔑 5. Configure kubectl (IMPORTANT)

```bash
aws eks update-kubeconfig --region ap-south-1 --name my-eks-cluster
```

---

# ✅ 6. Verify Cluster

```bash
kubectl get nodes
```

✔ Output:

```
ip-10-0-x-x   Ready
```

---

# 🚀 7. Deploy Test App

```bash
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --type=NodePort --port=80
kubectl get svc
```

---

# 🎯 8. What You Created

✔ VPC
✔ Subnets (Multi-AZ)
✔ Internet Gateway
✔ EKS Cluster
✔ Node Group

---

# ⚠️ 9. Cleanup (VERY IMPORTANT 💰)

```bash
terraform destroy --auto-approve
```

---

# 🔥 10. Interview One-Liner

👉 “I have created an EKS cluster using Terraform, including VPC, subnets, IAM roles, and node groups, and deployed applications using kubectl.”

---

# 🚀 Final Conclusion

This is a complete end-to-end EKS setup using Terraform, suitable for real DevOps projects and interviews.

---
