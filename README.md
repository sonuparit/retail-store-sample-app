# 🚀 Reverse engineered Terraform & Created GitOps-Driven CI/CD pipeline from scratch

> [!NOTE]
> This project is based on an existing application that I reverse-engineered and enhanced — [(original repo)](https://github.com/aws-containers/retail-store-sample-app)
>
> Productionized the same microservices application with observability stack — [(see here)](https://github.com/sonuparit/retail-store-reverse-engineered)

Took an existing AWS microservices platform, reverse engineered it, redesigned the Terraform workflow, and implemented a GitOps-based delivery pipeline from scratch.

## 📚 Table of Contents

- [Overview](#-overview)
- [Project Goals](#-project-goals)
- [Architecture](#️-architecture)
- [What This Project Demonstrates](#-what-this-project-demonstrates)
- [Repository Structure](#-repository-structure)
- [Tech Stack](#️-tech-stack)
- [Deployment Guide](#-deployment-guide)
- [Features](#-features)
- [Core Implementation](#️-core-implementation)
- [Architectural Decisions](#️-architectural-decisions)
- [Challenges & Solutions](#️-challenges--solutions)
- [Operational Outcomes](#-operational-outcomes)
- [Key Learnings](#-key-learnings)
- [Future Improvements](#-future-improvements)
- [Acknowledgments](#-acknowledgments)

## 📝 Overview

This project started from an existing AWS microservices application.

Instead of treating the infrastructure as a black box, I reverse-engineered the Terraform codebase, analyzed infrastructure dependencies, and redesigned the deployment workflow to improve maintainability, visibility, and operational control.

Key enhancements include:

- Modular Terraform provisioning workflow
- GitOps-based deployment strategy using ArgoCD
- CI pipeline built from scratch using GitHub Actions
- Automated Docker image publishing to Amazon ECR
- Kubernetes Secret encryption with AWS KMS
- Controlled deployment promotion through branch-based workflows

The result is a production-oriented platform demonstrating Infrastructure as Code, GitOps, CI/CD automation, cloud security, and Kubernetes operations.

## 🎯 Project Goals

- Understand and reverse engineer a real-world Terraform codebase
- Design a GitOps-driven deployment workflow
- Build CI automation from scratch
- Improve infrastructure visibility and maintainability
- Apply production-oriented security practices
- Gain operational understanding of Kubernetes deployments on AWS

## 🏗️ Architecture

![System Architecture](./docs/terraform-with-cicd.png)

## 🎯 What This Project Demonstrates

- Infrastructure as Code (Terraform)
- Kubernetes Platform Engineering (EKS)
- GitOps (ArgoCD)
- CI/CD Automation (GitHub Actions)
- Containerization (Docker)
- Cloud Security (IAM, KMS, Secrets)
- AWS Networking (VPC)
- Deployment Automation

## 📂 Repository Structure

```text
.
├── README.md
├── LICENSE
├── config.conf         # IAM & Secrets configuration
├── docs/
├── snaps/
├── src/                # application source code
├── argocd
│   ├── applications/   # retail store app
│   └── projects/
└── terraform
    ├── addons.tf       # Add-ons certmanager/ingress
    ├── argocd.tf
    ├── kube-conf.tf
    ├── locals.tf
    ├── main.tf
    ├── outputs.tf
    ├── security.tf
    ├── variables.tf
    └── versions.tf
```

## 🛠️ Tech Stack

| Category | Technologies |
|-----------|-------------|
| ☁️ **Cloud & Infrastructure** | AWS • EKS • ECR • IAM • KMS • VPC |
| 🏗️ **Infrastructure as Code** | Terraform |
| 📦 **Containerization** | Docker |
| ☸️ **Kubernetes Ecosystem** | Kubernetes • ArgoCD • NGINX Ingress • Cert Manager |
| 🚀 **CI/CD & GitOps** | GitHub Actions • GitOps |
| 🔒 **Security** | IAM Roles • Kubernetes Secrets • AWS KMS Encryption |
| 🌿 **Version Control** | Git • GitHub |

## 📦 Deployment Guide

> [!NOTE]
> This project currently uses GitHub Secrets for AWS authentication.
>
> Future enhancement: GitHub OIDC federation to eliminate long-lived AWS credentials.

### 1. Prerequisites

| Tool          | Version | Installation                                                                         |
| ------------- | ------- | ------------------------------------------------------------------------------------ |
| **AWS CLI**   | v2+     | [Install Guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) |
| **Terraform** | 1.0+    | [Install Guide](https://developer.hashicorp.com/terraform/install)                   |
| **kubectl**   | 1.33+   | [Install Guide](https://kubernetes.io/docs/tasks/tools/)                             |
| **Docker**    | 20.0+   | [Install Guide](https://docs.docker.com/get-docker/)                                 |
| **Helm**      | 3.0+    | [Install Guide](https://helm.sh/docs/intro/install/)                                 |
| **Git**       | 2.0+    | [Install Guide](https://git-scm.com/downloads)                                       |

### 2. `AWS Account` with appropriate permissions

### 3. Setup Steps

1. Clone Repository:

    ```bash
    git clone https://github.com/sonuparit/terraform-gitops-pipeline

    cd terraform-gitops-pipeline
    ```

2. Configure AWS**

    ```bash
    # Configure AWS CLI
    aws configure

    # Verify configuration
    aws sts get-caller-identity
    aws eks list-clusters --region ap-south-1
    ```

3. Setup GitHub Secrets (Required for GitOps)

    Go to your GitHub repository → **Settings** → **Secrets and variables** → **Actions**

    Add these secrets:

    | Secret Name             | Description    | Example        |
    | ----------------------- | -------------- | -------------- |
    | `AWS_ACCESS_KEY_ID`     | AWS Access Key | `AKIA...`      |
    | `AWS_SECRET_ACCESS_KEY` | AWS Secret Key | `wJalrXUt...`  |
    | `AWS_REGION`            | AWS Region     | `us-west-2`    |
    | `AWS_ACCOUNT_ID`        | AWS Account ID | `123456789012` |

### 4. Deployment

```bash
cd terraform/
```

1. Initialize:

    ```sh
    # Initialize Terraform
    terraform init
    ```

    ![image](./snaps/ss03.png)

2. Provision VPC:

    ```sh
    terraform apply -target=module.vpc
    ```

    ![image](./snaps/ss05.png)

    ⏱️ Expected time: **10 minutes**

    This creates:

    - ✅ VPC with public/private subnets

3. Provision EKS Cluster:

    ```sh
    terraform apply -target=module.retail_app_eks
    ```

    ![image](./snaps/ss07.png)

    ⏱️ Expected time: **10 minutes**

    This creates:

    - ✅ EKS cluster with Auto Mode
    - ✅ Security groups and IAM roles

4. Install addons:

    ```sh
    terraform apply -target=module.eks_addons
    ```

    ![image](./snaps/ss10.png)

    ⏱️ Expected time: **3 minutes**

    This creates:

    - ✅ NGINX Ingress Controller
    - ✅ Cert Manager for SSL

5. Configure kubectl

    ```bash
    # Get cluster name (with random suffix)
    terraform output cluster_name

    # Update kubeconfig
    aws eks update-kubeconfig --region ap-south-1 --name $(terraform output -raw cluster_name)

    # Verify connection
    kubectl get nodes
    ```

6. Deploy Applications

    ```bash
    # Deploy ArgoCD and add-ons
    terraform apply --auto-approve
    ```

    ![image](./snaps/ss13.png)

    **⏱️ Expected time: 05-10 minutes**

    This deploys:

    - ✅ ArgoCD for GitOps
    - ✅ ArgoCD applications

7. Get LoadBalancer URL and Access Application

    ```bash
    # Get load balancer URL
    kubectl get svc -n ingress-nginx
    ```

    ![image](./snaps/ss15.png)

    ![alt text](./snaps/ss23.png)

### 5. GitOps Workflow

How It Works

```text
Developer
    │
    ▼
push to GitOps Branch
    │
    ▼
GitHub Actions
    │
    ▼
Amazon ECR
    │
    ▼
Helm Values Update
    │
    ▼
ArgoCD Sync
    │
    ▼
Amazon EKS
```

The workflow automatically detects which services changed:

1. Made changes across any microservices and pushed to the `gitops` branch

    ![alt text](./snaps/ss33.png)

2. Validated CI trigger

    ![alt text](./snaps/ss36.png)

3. Validated ECR Uploads

    ![alt text](./snaps/ss37.png)

4. Validated ArgoCD sync

    ![alt text](./snaps/ss39.png)

5. Validated changed deployment

    ![alt text](./snaps/ss41.png)

6. Validated the full application

    ![alt text](./snaps/ss24.png)

## ✨ Features

- Modular Terraform provisioning workflow (VPC → EKS → Add-ons → Applications)
- GitOps-driven application delivery using ArgoCD
- Automated Docker build and publish pipeline using GitHub Actions
- Reduced unnecessary image builds by only rebuilding changed services
- Parallel image builds using GitHub Actions matrix strategy
- Secure container registry integration with Amazon ECR
- Kubernetes Secret encryption using AWS KMS
- Branch-based deployment control for safer releases
- Infrastructure provisioning and application deployment fully managed as code

## ⚙️ Core Implementation

- ### 1. Terraform Reverse Engineering

    The original project used a single Terraform apply for everything.

    I restructured it into four logical stages:

  - VPC provisioning
  - EKS cluster setup
  - Add-ons installation (CertManager, Nginx)
  - Application deployment via ArgoCD

  Instead of applying everything at once, I applied each layer individually to:

  - Understand dependencies
  - Observe resource creation behavior
  - Debug issues more effectively
  - See how state evolves over time

  This helped me move from “running Terraform” to actually **understanding** it.

- ### 2. CI Pipeline (Built from Scratch)

    The original setup had **no CI**.

    I designed and implemented 4 step CI pipeline using **GitHub Actions**:

  1. Build Docker images:

        ![System Architecture](./snaps/ss35.png)

  2. Tag images properly
  3. Authenticate securely with AWS:

        ![System Architecture](./snaps/ss31.png)

  4. Push to Amazon ECR:

        ![System Architecture](./snaps/ss37.png)

    Built step-by-step without relying on templates, focusing on understanding each component.

- ### 3. GitOps-Based Deployment Control

    I introduced a branch-based deployment strategy:

  - `main` → safe branch ( no deployment trigger )  
  - `gitops` → triggers CI pipeline and deployment

    This separation ensures:

    - Development work doesn’t accidentally deploy
    - Deployment is intentional and controlled

- ### 4. Additional Implementations

  - Introduced **AWS KMS integration** to enable encryption for Kubernetes Secrets, ensuring sensitive data is securely stored in etcd  
  - Added **Cert Manager** and **Nginx Ingress Controller** via Terraform add-ons  
  - Implemented retry logic for image push to handle race conditions

## 🏛️ Architectural Decisions

### 1. Modular Terraform Instead of Single Apply

- **Decision:**\
Break Terraform into multiple stages instead of one execution.

- **Why:**\
To understand infrastructure behavior and improve debugging.

- **Result:**\
Better visibility into dependencies and resource lifecycle.

### 2. Four stage CI pipeline

- **Decision:**\
Modular code with separate responsibility for separate tasks.

- **Why:**\
Easy to maintain and debug.

- **Result:**\
Achieved fully modular code with separation of concern.

### 3. Separate Deployment Trigger Using GitOps Branch

- Decision:\
Only trigger CI/CD when changes are pushed to *`gitops`* branch.

- Why:\
To clearly separate development activity from deployment actions.

- Result:\
More predictable and controlled release process.

## ⚔️ Challenges & Solutions

### 🏗️ Terraform

🔹 **1. Challenge**: Lack of visibility during execution

- Running `terraform apply` was provisioning everything, but I had no visibility into what was being created or in what order. It felt like things were happening behind the scenes without context.

    **🔹Solution**:

  - I broke the setup into smaller modules (**VPC → EKS → Add-ons → App deployment**) and applied them step-by-step. This made dependencies, resource creation, and execution flow much clearer.

🔹 **2. Challenge**: Understanding resource dependencies

- It wasn’t obvious how different AWS resources were connected or why certain components were required before others.

    **🔹Solution**:

  - *By applying Terraform in stages and observing the state after each step, I was able to map out the dependency chain and understand how the infrastructure is actually built.

### 🔁 GitOps

GitOps took significantly more effort than I initially expected.

🔹 **1. Challenge**: Detecting changed services

- *I needed the pipeline to rebuild only the services that actually changed, instead of rebuilding everything on every push.*

    **🔹Solution**:

  - Implemented change detection using git diff and filtered changes based on service directories (`src/<service>`), reducing unnecessary builds.

🔹 **2. Challenge**: Inefficient image builds

- Initially, I was building Docker images one-by-one for all services, which was slow and inefficient.

    **🔹Solution:**

  - Switched to a matrix-based strategy in GitHub Actions to build only changed services in parallel, improving speed and scalability.

🔹 **3. Challenge**: ECR authentication and repository handling

- *Faced issues with AWS authentication and pipeline failures when the ECR repository didn’t exist.*

    **🔹Solution:**

  - Configured secure AWS access using GitHub Secrets and added logic to create the ECR repository if it doesn’t exist, making deployments more reliable.

🔹 **4. Challenge**: CI and ArgoCD triggering conflicts

- Initially, ArgoCD was pointed to the entire repository, causing both CI and ArgoCD to react to the same changes and trigger simultaneously.

    **🔹Solution:**

  - Restricted ArgoCD to watch only the Helm chart path, creating a clear separation:

    - CI → builds and updates images
    - ArgoCD → handles deployment

## 📈 Operational Outcomes

- Successfully provisioned a complete AWS-based Kubernetes platform using Terraform
- Implemented end-to-end GitOps deployment workflow using ArgoCD
- Automated container build, tagging, and publishing through GitHub Actions
- Reduced unnecessary builds by implementing service-level change detection
- Improved CI scalability using parallel matrix-based Docker builds
- Introduced controlled deployment promotion through branch-based release workflow
- Enabled Kubernetes Secret encryption using AWS KMS
- Improved infrastructure visibility by separating Terraform provisioning into logical execution stages
- Established reproducible infrastructure provisioning and application deployment process

## 🎓 Key Learnings

- Large Terraform deployments become easier to troubleshoot when infrastructure is provisioned in logical dependency layers
- GitOps is most effective when deployment responsibilities are clearly separated from CI responsibilities
- Kubernetes operational issues are frequently caused by resource readiness and timing dependencies rather than configuration errors
- Matrix-based CI pipelines significantly improve scalability for multi-service architectures
- Infrastructure observability is critical for understanding provisioning failures and dependency chains
- Security should be integrated into platform design early through IAM controls, secret management, and encryption mechanisms
- Building delivery pipelines from scratch provides deeper operational understanding than relying solely on managed templates

## ⭐ Future Improvements

- CI Quality Gates
- GitHub OIDC federation
- Application reverse engineering
- Multi Environment Architecture
- Secrets Manager Integration
- Monitoring stack integration
- Centralized logging
- Alerting (Slack/Email)
- Different Deployment strategies
- Automated Rollback
- Disaster Recovery
- Terraform Remote State + Locking
- Fully automate infrastructure and application delivery processes

## 🙏 Acknowledgments

- **AWS Containers Team** for the original sample application
- **ArgoCD Community** for the excellent GitOps tooling
- **Terraform Community** for the AWS modules
- **GitHub Actions** for the CI/CD platform
