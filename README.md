![Banner](./docs/images/banner.png)

# 🏪 Retail Store Sample App – Cloud-Native Deployment on AWS EKS

## 📋 Overview
This project deploys the **AWS Retail Store Sample Application** — a microservices-based retail demo — to **Amazon EKS (Elastic Kubernetes Service)** using **Terraform** for infrastructure provisioning and **Helm/Kubernetes manifests** for workload orchestration.

The application simulates a modern retail web store consisting of multiple microservices (UI, Catalog, Carts, Checkout, and Orders) connected via REST APIs, with persistence layers using **AWS RDS MySQL**, **PostgreSQL**, **DynamoDB**, and **Redis**.

---

## 🚀 Architecture Overview

### Microservices

| Service      | Description                  | Backend                       |
|--------------|-----------------------------|-------------------------------|
| 🖥️ UI        | Frontend web application     | Connects to Catalog, Carts, Checkout |
| 📦 Catalog   | Product catalog microservice | AWS RDS MySQL (Aurora)        |
| 🛒 Carts     | Shopping cart service        | DynamoDB                      |
| 💳 Checkout  | Checkout/order processing    | Redis                         |
| 📑 Orders    | Order management             | PostgreSQL + RabbitMQ         |

### Cloud Services

| Service                        | Description                                   |
|--------------------------------|-----------------------------------------------|
| ☁️ Amazon EKS                  | Orchestrates Kubernetes workloads             |
| 🗄️ Amazon RDS (Aurora MySQL & PostgreSQL) | Persistent relational databases      |
| 🧩 Amazon DynamoDB             | NoSQL backend for Carts                       |
| ⚡ Amazon ElastiCache (Redis)   | In-memory cache for Checkout                  |
| 🌐 AWS Application Load Balancer (ALB) | Public entry point for the UI         |
| 🔐 AWS IAM + S3                | Credentials and Terraform remote state         |

---

## 🧱 Project Structure

```text
.
├── terraform/
│   ├── eks/          # EKS cluster and node configuration
│   ├── lib/          # Common modules and dependencies
│   ├── ecs/          # (Optional) ECS variant
│   └── apprunner/    # (Optional) App Runner variant
├── charts/           # Helm charts for each microservice
├── k8s/              # Kubernetes manifests
├── scripts/          # Deployment helper scripts
├── .github/workflows/# CI/CD pipelines
└── README.md         # Documentation
```

---

## ⚙️ Deployment Steps

### 1️⃣ Prerequisites

Ensure the following are installed locally:

| Tool      | Link                                              |
|-----------|---------------------------------------------------|
| AWS CLI   | [Install](https://aws.amazon.com/cli/)            |
| kubectl   | [Install](https://kubernetes.io/docs/tasks/tools/) |
| Helm      | [Install](https://helm.sh/docs/intro/install/)    |
| Terraform | [Install](https://developer.hashicorp.com/terraform/downloads) |

---

### 2️⃣ AWS Authentication

Verify AWS CLI credentials:

```bash
aws configure list
```

---

### 3️⃣ Infrastructure Deployment via Terraform

```bash
cd terraform/eks
terraform init
terraform apply -auto-approve
```

This creates:

- EKS cluster and worker nodes
- RDS instances (Aurora MySQL & PostgreSQL)
- DynamoDB and Redis
- IAM roles, VPC, and ALB

After deployment, view outputs:

```bash
terraform output
```

---

### 4️⃣ Configure Secrets in Kubernetes

After Terraform completes, set the database credentials for the Catalog service:

```bash
kubectl delete secret catalog -n default
kubectl create secret generic catalog \
  -n default \
  --from-literal=DB_USER=catalog_user \
  --from-literal=DB_PASSWORD='********'
```

Confirm the secret:

```bash
kubectl get secret catalog -n default -o yaml
```

---

### 5️⃣ Deploy the Application

```bash
kubectl apply -f k8s/
```

Wait for all pods to become ready:

```bash
kubectl get pods -n default -w
```

**Expected output:**

| NAME                        | READY | STATUS  | RESTARTS | AGE |
|-----------------------------|-------|---------|----------|-----|
| carts-7dc985f489-hw9p2      | 1/1   | Running | 0        | 5m  |
| catalog-7b58794d5c-fp8t9    | 1/1   | Running | 0        | 5m  |
| checkout-5dbdfd89db-vzbc9   | 1/1   | Running | 0        | 5m  |
| orders-5c6f4f984-flrbc      | 1/1   | Running | 0        | 5m  |
| ui-645868f79b-flg8l         | 1/1   | Running | 0        | 5m  |

---

### 6️⃣ Access the Application

Get the LoadBalancer URL:

```bash
kubectl get svc ui -n default
```

**Sample output:**

| NAME | TYPE         | CLUSTER-IP      | EXTERNAL-IP                                         | PORT(S)      | AGE |
|------|--------------|-----------------|-----------------------------------------------------|--------------|-----|
| ui   | LoadBalancer | 172.20.68.228   | k8s-default-ui-312340c72a-3ca99e18625c8fb9.elb.us-east | 80:30474/TCP | 38h |

---

## 🧩 Troubleshooting

| 🧠 Issue                                   | ⚙️ Cause                                 | 🩺 Solution                                                        |
|--------------------------------------------|------------------------------------------|--------------------------------------------------------------------|
| ❌ Error 1045 (28000): Access denied for user | Wrong DB credentials or secret not mounted | Recreate secret with correct password and restart pod              |
| ⚠️ CrashLoopBackOff on catalog             | DB unreachable / subnet / password mismatch | Check RDS endpoint, update secret, re-rollout deployment           |
| 💥 UI shows 500 error                      | Backend microservice unavailable         | Verify catalog, carts, checkout pods are healthy                   |
| 🌍 DNS_PROBE_FINISHED_NXDOMAIN             | ALB DNS not ready                        | Wait a few minutes or re-check `kubectl get svc ui`                |
| 🔒 Access denied by security group          | Wrong VPC or subnet config               | Check RDS SG inbound rules (allow EKS node SG)                     |

View service logs:

```bash
kubectl logs -l app.kubernetes.io/name=catalog -n default --tail=50
kubectl logs -l app.kubernetes.io/name=ui -n default --tail=50
```

---

## 🔁 CI/CD Setup (GitHub Actions)

The included workflow automates:

- Terraform validation and apply
- Kubernetes deployments
- Helm packaging and push

**Required Secrets (in GitHub → Repo → Settings → Secrets → Actions):**

| Name                | Description                  |
|---------------------|-----------------------------|
| AWS_ACCESS_KEY_ID   | Your AWS access key         |
| AWS_SECRET_ACCESS_KEY | Your AWS secret key       |
| AWS_REGION          | Region (e.g. us-east-1)     |

---

## ✅ Verification Checklist

| Check                        | Status |
|------------------------------|--------|
| All pods Running             | ✅     |
| UI accessible via LoadBalancer| ✅     |
| Catalog connected to MySQL   | ✅     |
| Terraform apply completed cleanly | ✅ |
| README, manifests committed  | ✅     |

---

## 👩‍💻 Author

**Evelyn Ita**  
Cloud & DevOps Engineer — AWS | Terraform | Kubernetes | CI/CD

---
