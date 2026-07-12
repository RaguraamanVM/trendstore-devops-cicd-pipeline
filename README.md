# Trendstore App – Application Deployment on AWS EKS with Jenkins CI/CD

## Overview

This project demonstrates the implementation of a complete end-to-end DevOps pipeline for deploying the Trendify React application on Amazon Elastic Kubernetes Service (EKS) using modern cloud-native technologies and automation practices. The infrastructure is provisioned using Terraform, enabling automated creation of AWS resources such as the VPC, IAM roles, worker nodes, and the EKS cluster. The application is containerized using Docker, stored in DockerHub, and deployed to Kubernetes using deployment and service manifests.

A Jenkins CI/CD pipeline integrated with GitHub Webhooks automates the entire software delivery process, including source code checkout, Docker image build, image publishing to DockerHub, and deployment to the EKS cluster. Additionally, a comprehensive monitoring solution is implemented using Prometheus and Grafana to provide real-time visibility into Kubernetes cluster health, application performance, resource utilization, and operational metrics through interactive dashboards. This project showcases Infrastructure as Code (IaC), Continuous Integration, Continuous Deployment (CI/CD), container orchestration, and monitoring to achieve a scalable, automated, and production-ready application deployment workflow.

## Architecture


```text
GitHub Repository
        │
        ▼
Jenkins Pipeline
        │
        ▼
Docker Build
        │
        ▼
DockerHub Registry
        │
        ▼
AWS EKS Cluster
        │
        ▼
Kubernetes Deployment
        │
        ├──────────────► Prometheus
        │                    │
        │                    ▼
        │                Grafana Dashboard
        │
        ▼
LoadBalancer Service
        │
        ▼
React Application
```


### Components

| Component | Purpose |
|------------|----------|
| React Application | Frontend application |
| Docker | Containerization |
| DockerHub | Image Registry |
| Terraform | Infrastructure Provisioning |
| AWS EKS | Kubernetes Cluster |
| Jenkins | CI/CD Automation |
| GitHub | Source Code Management |
| Prometheus & Grafanna | Monitoring |


---

## Project Objectives

- Containerize the application using Docker.
- Push Docker images to DockerHub.
- Provision AWS infrastructure using Terraform.
- Create and manage an EKS cluster.
- Deploy the application using Kubernetes manifests.
- Configure Jenkins CI/CD pipeline.
- Automate build, push, and deployment process.
- Monitor the node and cluster health.

---

# Technology Stack

- ReactJS
- Docker
- DockerHub
- AWS
- Terraform
- Amazon EKS
- Kubernetes
- Jenkins
- GitHub
- Prometheus, Grafanna

---

## Step 1: Application build Artifacts

The application build artifacts were made available for deployment activities. Initial verification was performed to ensure the application was functioning correctly before proceeding with Dockerization and cloud deployment.

Application runs on:

```text
http://localhost:3000
```

---

# Step 2: Dockerization

## Create Dockerfile

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

## Build Docker Image

```bash
docker build -t trend-app .
```

## Run Docker Container

```bash
docker run -d -p 3000:3000 --name trend-container trend-app
```

## Verify Application

```bash
docker ps
```

Open:

```text
http://localhost:3000
```

---

# Phase 3: DockerHub Integration

## Login to DockerHub

```bash
docker login -u <dockerhub-username>
```

## Tag Image

```bash
docker tag trend-app <dockerhub-username>/trend-app:latest
```

## Push Image

```bash
docker push <dockerhub-username>/trend-app:latest
```
---

# Phase 4: Infrastructure Provisioning using Terraform

## Infrastructure Components

Terraform provisions:

- VPC
- IAM Roles
- Security Groups
- EKS Cluster
- Worker Nodes

## Initialize Terraform

```bash
terraform init
```

## Validate Configuration

```bash
terraform validate
```

## Preview Changes

```bash
terraform plan out=tfplan
```

## Provision Infrastructure

```bash
terraform apply tfplan -auto-approve
```

## Verify Resources

```bash
aws eks list-clusters
```

---

# Phase 5: Configure AWS EKS

## Configure AWS CLI

```bash
aws configure
```

## Update Kubeconfig

```bash
aws eks update-kubeconfig \
--region <region> \
--name <cluster-name>
```

## Verify Cluster

```bash
kubectl get nodes
```

Expected Result:

```text
Nodes should be in Ready state.
```

---

# Phase 6: Kubernetes Deployment

## Deployment Manifest

Create:

```yaml
deployment.yaml
```

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: trend-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: trend-app
  template:
    metadata:
      labels:
        app: trend-app
    spec:
      containers:
      - name: trend-app
        image: <dockerhub-username>/trend-app:latest
        ports:
        - containerPort: 3000
```

---

## Service Manifest

Create:

```yaml
service.yaml
```

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: trend-service
spec:
  selector:
    app: trend-app
  ports:
  - port: 80
    targetPort: 3000
  type: LoadBalancer
```

---

## Deploy Application

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

## Verify Deployment

```bash
kubectl get deployments
kubectl get pods
kubectl get svc
```

---

# Phase 7: Version Control

## Git Configuration

```bash
git init
git add .
git commit -m "Initial Commit"
```

## Create Repository

Create a GitHub repository and push code.

```bash
git remote add origin <repository-url>

git branch -M main

git push -u origin main
```
---

# Phase 8: Jenkins Setup

## Install Jenkins

Run Jenkins as Docker container:

```bash
docker run -d \
-p 8080:8080 \
-p 50000:50000 \
--name jenkins \
jenkins/jenkins:lts
```

## Required Plugins

Install:

- Docker Pipeline
- Kubernetes
- Git
- GitHub Integration
- Pipeline
- AWS Credentials

---

## Configure GitHub Webhook

GitHub Repository

→ Settings

→ Webhooks

→ Add Webhook

Payload URL:

```text
http://<jenkins-server>:8080/github-webhook/
```

Trigger builds automatically on every commit.

---

# Jenkins Pipeline

## Jenkinsfile

Pipeline stages:

1. Checkout Code
2. Build Application
3. Build Docker Image
4. Push Docker Image
5. Deploy to Kubernetes
6. Verify Deployment

Example Flow:

```text
Checkout
    ↓
Build
    ↓
Docker Build
    ↓
Docker Push
    ↓
Kubernetes Deploy
    ↓
Success
```

---

# CI/CD Workflow

```text
Developer Push
        ↓
GitHub Repository
        ↓
Webhook Trigger
        ↓
Jenkins Pipeline
        ↓
Docker Build
        ↓
DockerHub Push
        ↓
EKS Deployment
        ↓
Kubernetes Service
        ↓
Application Available
```

---

# Validation Steps

## Docker

```bash
docker ps
docker images
```

## Kubernetes

```bash
kubectl get pods
kubectl get deployments
kubectl get svc
```

## Jenkins

Verify:

- Successful Pipeline Execution
- Docker Image Push
- Kubernetes Deployment

---

## Monitoring with Prometheus & Grafana

Prometheus was deployed to collect real-time metrics from the Kubernetes cluster and application workloads, while Grafana was configured to visualize these metrics through dashboards. This monitoring solution provides insights into pod status, CPU and memory utilization, application availability, and overall cluster health, enabling effective performance monitoring and troubleshooting.

# Results

Successfully:

- Dockerized the React application.
- Built and pushed image to DockerHub.
- Provisioned AWS infrastructure using Terraform.
- Created AWS EKS cluster.
- Deployed application using Kubernetes manifests.
- Configured Jenkins CI/CD automation.
- Enabled automatic deployment using GitHub Webhooks.

---

# Author
**Raguraaman V M**


**DevOps Deployment Project**

React Application Deployment using Docker, Terraform, AWS EKS, Kubernetes, DockerHub, GitHub, and Jenkins CI/CD.
