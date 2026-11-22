# Kubernetes Microservices Orchestration (Task 5)

##  Overview

This directory contains the declarative Kubernetes manifests required to deploy and orchestrate the `iVolve` microservices application. The workloads are isolated inside the dedicated `ivolve` namespace with persistent storage, configuration management, stateful database reliability, and ingress routing.

---

##  Manifests Breakdown

| Order | File | Kind | Description |
| :---: | :--- | :--- | :--- |
| **01** | `01-namespace.yaml` | `Namespace` | Creates isolated `ivolve` namespace for all resources. |
| **02** | `02-configmap-secret.yaml` | `ConfigMap` & `Secret` | Separates plain config (URLs, DB names) from sensitive secrets (passwords). |
| **03** | `03-storageclass.yaml` | `StorageClass` | Storage provisioner for dynamic volume provisioning. |
| **04** | `04-mysql-statefulset.yaml` | `StatefulSet` & `Service` | MySQL 8.0 StatefulSet with Headless Service (`clusterIP: None`) & PVC template. |
| **05** | `05-auth-service.yaml` | `Deployment` & `Service` | Python/Flask authentication service (Port 5000, ClusterIP). |
| **06** | `06-roadmap-service.yaml` | `Deployment` & `Service` | Java/Spring Boot roadmap microservice (Port 8080, ClusterIP). |
| **07** | `07-frontend.yaml` | `Deployment` & `Service` | Node.js/Express web frontend (Port 3000, ClusterIP). |
| **08** | `08-ingress.yaml` | `Ingress` | NGINX Ingress resource routing root HTTP traffic (`/`) to the frontend service. |

---

##  Key Architectural Highlights

1. **StatefulSet & Headless Service for MySQL**:
   - Ensures stable network identity and predictable persistent volume binding.
   - The Headless service (`clusterIP: None`) allows direct DNS resolution (`mysql.ivolve.svc.cluster.local`).
2. **Configuration Decoupling**:
   - Microservices consume configurations through `ConfigMaps` and `Secrets` rather than hardcoded environment variables (12-Factor App methodology).
3. **Resilient Service Communication**:
   - Frontend reaches backend services via Kubernetes internal CoreDNS:
     - Auth: `http://auth-service:5000`
     - Roadmap: `http://roadmap-service:8080`

---

##  Deployment Guide

### Prerequisites
- Running Kubernetes cluster (Docker Desktop Kubernetes, Minikube, or AWS EKS).
- `kubectl` CLI tool configured.

### Step 1: Apply All Manifests
Deploy all manifests into the cluster:
```bash
kubectl apply -f .
```
*(Or from root directory: `kubectl apply -f kubernetes/`)*

### Step 2: Verify Workloads
Check that all Pods are running and healthy:
```bash
kubectl get pods -n ivolve
```

Expected output:
```text
NAME                               READY   STATUS    RESTARTS   AGE
auth-service-7d8674bcc7-xxxxx      1/1     Running   0          2m
frontend-764fd84795-xxxxx          1/1     Running   0          2m
mysql-0                            1/1     Running   0          2m
roadmap-service-6744b7fc44-xxxxx   1/1     Running   0          2m
```

Verify Services and PVCs:
```bash
kubectl get svc,pvc -n ivolve
```

### Step 3: Access the Application
Forward the frontend service port to your host machine:
```bash
kubectl port-forward svc/frontend -n ivolve 3001:3000
```
Open your browser at: **`http://localhost:3001`**

---

##  Verification & Health Checks

1. **Check Auth Service Database Connection:**
   ```bash
   kubectl logs -l app=auth-service -n ivolve --tail=30
   ```
2. **Inspect User Registration Directly in MySQL:**
   ```bash
   kubectl exec -it mysql-0 -n ivolve -- mysql -u ivolve_user -pivolve_password ivolve -e "SELECT id, username, created_at FROM users;"
   ```

---

##  Teardown
To remove all deployed Kubernetes workloads:
```bash
kubectl delete -f .
```
