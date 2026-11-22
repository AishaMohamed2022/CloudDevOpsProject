# CloudDevOpsProject
# Cloud-Native GitOps Microservices Architectur

---

## 📌 Project Overview

This repository contains an end-to-end cloud-native microservices web application. The platform provides user authentication (sign up & login) and displays a **DevOps Roadmap** upon successful authentication.

The project demonstrates modern DevOps engineering practices:
- **Containerization**: Multi-container Docker environments and multi-stage builds.
- **Orchestration**: Fully declarative Kubernetes manifests with persistent storage, decoupled configurations, and service discovery.
- **Continuous Integration (CI)**: Automated GitHub Actions pipeline using Reusable Workflows and Trivy security scanning.
- **Continuous Deployment (CD)**: Automated GitOps synchronization via **ArgoCD**.

---

## 🏛️ System Architecture

```text
+---------------------------------------------------------------------------------------------------+
|                                  DEVELOPMENT & VERSION CONTROL                                    |
+---------------------------------------------------------------------------------------------------+
                                                  |
                                                  | (1) Developer Push
                                                  v
                                      +-----------------------+
                                      |   GitHub Repository   |
                                      +-----------+-----------+
                                                  |
                                                  | (2) Trigger Workflow
                                                  v
+---------------------------------------------------------------------------------------------------+
|                                CONTINUOUS INTEGRATION (GitHub Actions)                            |
+---------------------------------------------------------------------------------------------------+
|                                                                                                   |
|  [Stage 1: Build Image] --> [Stage 2: Scan (Trivy)] --> [Stage 3: Push Image to Registry]         |
|                                                                                                   |
|  [Stage 4: Delete Local Image] --> [Stage 5: Update K8s Manifest] --> [Stage 6: Push to Git]      |
|                                                                                                   |
+---------------------------------------------+-----------------------------------------------------+
                                              |
                                              | (3) GitOps Commit Loop
                                              v
+---------------------------------------------------------------------------------------------------+
|                                 CONTINUOUS DEPLOYMENT (ArgoCD)                                    |
+---------------------------------------------------------------------------------------------------+
|                                                                                                   |
|        ArgoCD watches GitHub repo & automatically synchronizes manifests to live cluster          |
|                                                                                                   |
+---------------------------------------------+-----------------------------------------------------+
                                              |
                                              | (4) Auto-Sync & Rolling Update
                                              v
+---------------------------------------------------------------------------------------------------+
|                               KUBERNETES CLUSTER (Namespace: ivolve)                               |
+---------------------------------------------------------------------------------------------------+
|                                                                                                   |
|   [ Ingress Routing ]                                                                             |
|            |                                                                                      |
|            v                                                                                      |
|   [ Frontend Service (Node.js / Express) - Port 3000 ]                                            |
|         |                                 |                                                       |
|         v (HTTP :5000)                    v (HTTP :8080)                                          |
|   [ Auth Service (Python / Flask) ]      [ Roadmap Service (Java / Spring Boot) ]                 |
|         |                                                                                         |
|         v (TCP :3306 via Headless Service)                                                        |
|   [ MySQL StatefulSet (Persistent Volume via StorageClass) ]                                      |
|                                                                                                   |
+---------------------------------------------------------------------------------------------------+
```

---

## 📂 Repository Structure

```text
CloudDevOpsProject/
├── .github/
│   └── workflows/
│       ├── reusable-ci.yml           # Reusable CI Workflow (Shared Library equivalent)
│       ├── ci-frontend.yml           # CI trigger for Frontend Service
│       ├── ci-auth.yml               # CI trigger for Auth Service
│       └── ci-roadmap.yml            # CI trigger for Roadmap Service
├── argocd/
│   └── application.yaml              # ArgoCD GitOps Application CRD definition
├── auth-service/                     # Python Flask Authentication API & Dockerfile
├── frontend/                         # Node.js Express UI & Dockerfile
├── roadmap-service/                  # Java 21 Spring Boot API & Dockerfile
├── kubernetes/
│   ├── 01-namespace.yaml             # Isolated 'ivolve' namespace
│   ├── 02-configmap-secret.yaml      # Decoupled configurations & encrypted secrets
│   ├── 03-storageclass.yaml          # Dynamic persistent volume provisioner
│   ├── 04-mysql-statefulset.yaml     # Stateful MySQL DB with Headless Service
│   ├── 05-auth-service.yaml          # Deployment & ClusterIP service (Port 5000)
│   ├── 06-roadmap-service.yaml       # Deployment & ClusterIP service (Port 8080)
│   ├── 07-frontend.yaml              # Deployment & ClusterIP service (Port 3000)
│   ├── 08-ingress.yaml               # Ingress routing external traffic to frontend
│   └── README.md                     # Dedicated Kubernetes documentation
├── docker-compose.yml                # Local multi-container development & testing
└── README.md                         # Project documentation
```

---

## 🧩 Microservices Specification

| Service | Technology Stack | Port | Purpose |
| :--- | :--- | :---: | :--- |
| **Frontend** | Node.js (Express & EJS) | `3000` | Web UI, handles sessions, calls backend APIs |
| **Auth Service** | Python 3.12 (Flask) | `5000` | User registration & authentication with bcrypt |
| **Roadmap Service** | Java 21 (Spring Boot) | `8080` | Provides DevOps roadmap content API |
| **Database** | MySQL 8.0 | `3306` | Relational storage for user accounts |

---

## 🚀 Quickstart & Setup Guide

### 1. Local Testing with Docker Compose
To run and test the complete stack locally using Docker Compose:
```bash
docker compose up --build -d
```
Access the application at: `http://localhost:3001` (or `http://localhost:3000`).

---

### 2. Kubernetes Deployment
Deploy the full architecture to the local Kubernetes cluster:
```bash
# Apply all manifests to the cluster
kubectl apply -f kubernetes/

# Verify that all 4 pods are running
kubectl get pods -n ivolve
```

Expected output:
```text
NAME                               READY   STATUS    RESTARTS   AGE
auth-service-7d8674bcc7-xxxxx      1/1     Running   0          1m
frontend-764fd84795-xxxxx          1/1     Running   0          1m
mysql-0                            1/1     Running   0          1m
roadmap-service-6744b7fc44-xxxxx   1/1     Running   0          1m
```

Access the frontend via port-forwarding:
```bash
kubectl port-forward svc/frontend -n ivolve 3001:3000
```
Open **`http://localhost:3001`** in your browser.

---

### 3. Continuous Deployment with ArgoCD (GitOps)
ArgoCD monitors this repository and ensures automated synchronization:
```bash
# Forward ArgoCD web server port
kubectl port-forward svc/argocd-server -n argocd 8085:443

# Apply ArgoCD Application CRD
kubectl apply -f argocd/application.yaml
```
Open **`https://localhost:8085`** to view the real-time visual GitOps synchronization tree.

---

## 🛡️ Key Implementations
- **CI with Reusable Workflows**: DRY (Don't Repeat Yourself) principle applied across all microservices.
- **Trivy Vulnerability Scanning**: Integrated directly into the CI pipeline to scan images before registry push.
- **GitOps Continuous Deployment**: Zero manual deployments; changes pushed to Git are automatically synced to the cluster by ArgoCD.
- **Stateful Persistence**: Database durability guaranteed through Kubernetes `StatefulSet` and dynamic `StorageClass`.
- **Decoupled Architecture**: Secrets and configurations managed independently via `Secrets` and `ConfigMaps`.
