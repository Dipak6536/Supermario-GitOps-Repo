# 🎮 Super Mario – End-to-End DevSecOps & GitOps

An end-to-end **DevSecOps, CI/CD and GitOps project** that containerizes a JavaScript-based Super Mario game, performs automated source-code and container security scanning, publishes versioned Docker images, updates Kubernetes manifests automatically, and deploys the application to **Azure Kubernetes Service (AKS)** using **Argo CD**.

The project demonstrates how application development, security, containerization, continuous integration, GitOps and Kubernetes deployment can be integrated into a single automated delivery workflow.

---

## 🚀 Project Overview

This project implements the following delivery lifecycle:

```text
Developer
    │
    │ Code Change
    ▼
GitHub Repository
    │
    │ Push to main
    ▼
GitHub Actions
    │
    ├── SAST / SonarQube
    │
    ├── Docker Image Build
    │
    ├── Push Image to Docker Hub
    │
    ├── Trivy Container Security Scan
    │
    └── Update Kubernetes Manifest
            │
            ├── deployment.yaml
            └── version.txt
                    │
                    ▼
              GitHub Repository
                    │
                    │ GitOps
                    ▼
                 Argo CD
                    │
                    ▼
              Azure AKS Cluster
                    │
                    ▼
             Super Mario Game
```

---

# 🏗️ Architecture

```text
                         ┌─────────────────────┐
                         │     Developer       │
                         │                     │
                         │ Code Changes        │
                         └──────────┬──────────┘
                                    │
                                    │ git push
                                    ▼
                         ┌─────────────────────┐
                         │       GitHub        │
                         │                     │
                         │ Application Code    │
                         │ Dockerfile          │
                         │ Kubernetes YAML     │
                         └──────────┬──────────┘
                                    │
                                    │ Push to main
                                    ▼
                  ┌──────────────────────────────────┐
                  │          GitHub Actions           │
                  │                                  │
                  │  1. SonarQube SAST              │
                  │             ↓                    │
                  │  2. Docker Build                │
                  │             ↓                    │
                  │  3. Docker Hub Push             │
                  │             ↓                    │
                  │  4. Trivy Container Scan        │
                  │             ↓                    │
                  │  5. Update deployment.yaml       │
                  │     and version.txt              │
                  └─────────────────┬────────────────┘
                                    │
                                    │ Git Commit / Push
                                    ▼
                         ┌─────────────────────┐
                         │   GitHub GitOps     │
                         │                     │
                         │ deployment.yaml     │
                         │ image: :VERSION     │
                         └──────────┬──────────┘
                                    │
                                    │ Reconciliation
                                    ▼
                         ┌─────────────────────┐
                         │      Argo CD        │
                         │                     │
                         │ GitOps Controller   │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │    Azure AKS        │
                         │                     │
                         │ Deployment          │
                         │ Service             │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │   Super Mario App   │
                         │                     │
                         │ HTTP :8600          │
                         └─────────────────────┘
```

---

# 🛠️ Technologies Used

| Category            | Technology                     |
| ------------------- | ------------------------------ |
| Source Control      | Git / GitHub                   |
| CI/CD               | GitHub Actions                 |
| SAST                | SonarQube                      |
| Containerization    | Docker                         |
| Container Registry  | Docker Hub                     |
| Container Security  | Trivy                          |
| Orchestration       | Kubernetes                     |
| Cloud Platform      | Microsoft Azure                |
| Kubernetes Platform | Azure Kubernetes Service (AKS) |
| GitOps              | Argo CD                        |
| Application Server  | Apache Tomcat                  |
| Application         | HTML5 / JavaScript             |
| Configuration       | YAML                           |
| Automation          | Bash / Shell                   |

---

# 📂 Repository Structure

```text
Supermario-GitOps-Repo/
│
├── .github/
│   └── workflows/
│       ├── e2e-gitops.yaml
│       ├── gitops-build-push-supermario-image.yaml
│       ├── gitops-sast-sonar.yaml
│       └── run-container-scan-supermario-image.yaml
│
├── demo/
│   └── demo.PNG
│
├── webapp/
│   ├── Enjine/
│   ├── code/
│   ├── index.html
│   └── ...
│
├── Dockerfile
├── deployment.yaml
├── sonar-project.properties
├── version.txt
└── README.md
```

The repository currently contains separate workflow definitions for the individual DevSecOps stages as well as an end-to-end workflow.

---

# 🔄 CI/CD Pipeline

The primary end-to-end workflow is:

```text
e2e-gitops.yaml
```

It is triggered when code is pushed to the `main` branch.

## Pipeline Stages

### 1. Checkout Source Code

GitHub Actions checks out the latest application source code.

```yaml
- name: Checkout Repository
  uses: actions/checkout@v7
```

The E2E pipeline also uses a full Git history for SonarQube analysis.

---

### 2. Static Application Security Testing – SAST

SonarQube analyzes the application source code.

```text
Source Code
     │
     ▼
 SonarQube
     │
     ├── Code Analysis
     ├── Security Analysis
     └── Quality Gate
```

The repository contains a SonarQube configuration file and a dedicated SonarQube workflow. The consolidated pipeline performs the SonarQube scan before the Docker build stage.

---

### 3. Dynamic Image Versioning

The project maintains the current version in:

```text
version.txt
```

The pipeline calculates the next image version automatically.

For example:

```text
Current version:
3

Next version:
4
```

This version is then used consistently for the Docker image and Kubernetes deployment.

---

### 4. Docker Image Build

The application is packaged into a Docker image.

```text
Super Mario Source Code
          │
          ▼
      Dockerfile
          │
          ▼
    Docker Image
```

The Dockerfile uses Apache Tomcat as the application server, copies the `webapp` content into Tomcat's ROOT application directory and exposes port `8080`.

Example image:

```text
dhdipak/supermariogitopsproject:5
```

---

### 5. Push Image to Docker Hub

After the image is built, GitHub Actions authenticates with Docker Hub using GitHub Secrets and pushes the versioned image.

```text
GitHub Actions
      │
      ▼
Docker Build
      │
      ▼
Docker Hub
      │
      └── supermariogitopsproject:<VERSION>
```

The E2E workflow uses Docker Hub credentials stored as GitHub Secrets rather than hard-coding credentials in the workflow.

---

# 🔐 Container Security – Trivy

After the Docker image is pushed, the pipeline pulls the image and exports it as a Docker image archive.

```text
Docker Hub
    │
    ▼
Docker Pull
    │
    ▼
Docker Save
    │
    ▼
Image TAR
    │
    ▼
Trivy
```

Trivy scans the container image for:

```text
HIGH
CRITICAL
```

vulnerabilities.

The project uses Trivy in tarball mode against the saved Docker image.

---

# 🛡️ DevSecOps Quality Gates

Security is integrated into the delivery pipeline rather than being performed manually after deployment.

```text
Code
 │
 ▼
SonarQube
 │
 ▼
Quality Gate
 │
 ▼
Docker Build
 │
 ▼
Container Scan
 │
 ▼
Deployment
```

This demonstrates the **Shift-Left Security** principle by introducing security checks early in the software delivery lifecycle.

> **Note:** The current Trivy configuration uses `exit-code: '0'`, meaning the scan reports HIGH/CRITICAL vulnerabilities without failing the pipeline. If strict vulnerability enforcement is desired, this can be changed to `exit-code: '1'`.

---

# ☸️ Kubernetes Deployment

The application is deployed to an **Azure Kubernetes Service (AKS)** cluster.

The Kubernetes configuration is maintained in:

```text
deployment.yaml
```

The manifest contains two Kubernetes resources:

```text
Deployment
    +
Service
```

---

## Kubernetes Deployment

The Deployment runs the Super Mario application container.

```yaml
kind: Deployment
```

The application container listens on:

```text
8080
```

The image tag is automatically updated by the CI/CD pipeline.

For example:

```yaml
image: dhdipak/supermariogitopsproject:5
```

The current repository manifest follows this versioned-image pattern.

---

## Kubernetes Service

The application is exposed using:

```yaml
type: LoadBalancer
```

The current configuration uses:

```text
Service Port: 8600
Container Port: 8080
```

Therefore:

```text
Client
  │
  │ HTTP :8600
  ▼
Azure LoadBalancer
  │
  ▼
Kubernetes Service
  │
  │ targetPort: 8080
  ▼
Super Mario Pod
```

The repository's current manifest defines the Service as a LoadBalancer with port `8600` forwarding to container port `8080`.

---

# 🔁 GitOps Deployment with Argo CD

The project follows the GitOps principle that the Kubernetes manifest stored in Git represents the desired application state.

```text
GitHub
   │
   │ deployment.yaml
   ▼
Argo CD
   │
   │ Reconcile
   ▼
AKS
   │
   ▼
Application
```

When the CI pipeline updates:

```text
deployment.yaml
```

from:

```yaml
image: dhdipak/supermariogitopsproject:4
```

to:

```yaml
image: dhdipak/supermariogitopsproject:5
```

Argo CD detects the Git change and reconciles the AKS cluster toward the new desired state.

This separates:

```text
CI → Build, Test, Scan, Publish
```

from:

```text
CD → GitOps reconciliation and deployment
```

---

# 🔢 Automated Version Management

The project uses:

```text
version.txt
```

as the application image version source.

Example:

```text
4
```

A new code change results in:

```text
4 → 5
```

The pipeline then updates both:

```text
version.txt
```

and:

```text
deployment.yaml
```

to reference the same image version.

Example:

```text
version.txt
    ↓
5

deployment.yaml
    ↓
dhdipak/supermariogitopsproject:5
```

This prevents the Kubernetes deployment from accidentally referencing an older image.

---

# 🔄 Complete Release Flow

Suppose the current production version is:

```text
4
```

A developer modifies the application and pushes the change:

```bash
git add .
git commit -m "Update Super Mario application"
git push origin main
```

GitHub Actions then performs:

```text
1. Checkout
      ↓
2. SonarQube SAST
      ↓
3. Quality Gate
      ↓
4. Calculate version = 5
      ↓
5. Docker Build :5
      ↓
6. Docker Push :5
      ↓
7. Trivy Container Scan
      ↓
8. Update deployment.yaml → :5
      ↓
9. Update version.txt → 5
      ↓
10. Commit GitOps changes
      ↓
11. Push changes to GitHub
      ↓
12. Argo CD detects Git change
      ↓
13. Argo CD syncs AKS
      ↓
14. Kubernetes runs image :5
```

---

# 📸 Application

The final application is the browser-based Infinite Mario game running from the Kubernetes deployment.

```text
Browser
   │
   │ http://<LoadBalancer-IP>:8600
   ▼
Azure Load Balancer
   │
   ▼
Kubernetes Service
   │
   ▼
Super Mario Pod
   │
   ▼
Tomcat
   │
   ▼
HTML5 / JavaScript Game
```

---

# 🔑 GitHub Secrets

The pipeline uses GitHub Actions Secrets for sensitive configuration.

Typical secrets used by the workflows include:

```text
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
SONAR_HOST_URL
SONAR_TOKEN
GIT_USERNAME
GIT_EMAIL
```

Secrets should never be hard-coded into workflow files or application source code.

---

# 📋 Key DevSecOps Concepts Demonstrated

This project demonstrates practical implementation of:

* Git-based source control
* GitHub Actions CI/CD
* Continuous Integration
* Static Application Security Testing
* SonarQube code analysis
* Quality Gates
* Docker containerization
* Docker image versioning
* Docker Hub image publishing
* Container vulnerability scanning
* Trivy
* Kubernetes Deployments
* Kubernetes Services
* Azure Kubernetes Service
* GitOps
* Argo CD
* Declarative infrastructure
* Automated manifest updates
* Continuous reconciliation
* Version-controlled deployments
* Shift-Left Security

---

# 🎯 Project Objectives

The main objectives of this project are to demonstrate how a traditional web application can be transformed into a modern cloud-native delivery model.

### Before DevOps

```text
Developer
   ↓
Build manually
   ↓
Copy application
   ↓
Deploy manually
```

### After DevSecOps + GitOps

```text
Developer
   ↓
Git Push
   ↓
Automated CI/CD
   ↓
Security Scan
   ↓
Docker Image
   ↓
Docker Registry
   ↓
GitOps Manifest Update
   ↓
Argo CD
   ↓
AKS
   ↓
Application
```

---

# 🧪 Validation

Useful commands for validating the deployment:

### Check Kubernetes nodes

```bash
kubectl get nodes
```

### Check application pods

```bash
kubectl get pods
```

### Check Service

```bash
kubectl get svc
```

### Check deployment

```bash
kubectl get deployment
```

### Check deployed image

```bash
kubectl describe deployment supermariogame-deployment
```

### Check Argo CD applications

```bash
kubectl get applications -n argocd
```

### Check application logs

```bash
kubectl logs <pod-name>
```

---

# 🧠 What This Project Demonstrates in an Interview

A concise way to explain the project:

> **"I built an end-to-end DevSecOps and GitOps pipeline for a containerized Super Mario web application. A code push to GitHub triggers GitHub Actions, where SonarQube performs SAST and a quality gate validates the source code. The application is then containerized using Docker, versioned dynamically, and pushed to Docker Hub. The resulting image is scanned with Trivy for HIGH and CRITICAL vulnerabilities. After successful validation, the pipeline updates the Kubernetes deployment manifest and version file in Git. Argo CD monitors the Git repository and reconciles the updated manifest to an AKS cluster, providing a fully automated GitOps-based deployment."**

---

# 📌 Future Improvements

The project can be extended further with:

* [ ] Use `exit-code: 1` for strict Trivy vulnerability enforcement
* [ ] Pin all GitHub Actions to immutable versions/SHA instead of floating references
* [ ] Add unit/integration tests before Docker build
* [ ] Add Docker image signing with Cosign
* [ ] Add SBOM generation
* [ ] Add Kubernetes resource requests and limits
* [ ] Add Kubernetes liveness/readiness probes
* [ ] Add non-root container execution
* [ ] Upgrade the legacy Tomcat/JRE base image
* [ ] Add Kubernetes NetworkPolicies
* [ ] Add separate Dev / QA / Production environments
* [ ] Implement pull-request based GitOps promotion
* [ ] Add automated rollback strategy
* [ ] Add monitoring with Prometheus and Grafana
* [ ] Add centralized logging
* [ ] Add Argo CD notifications
* [ ] Implement GitHub Actions concurrency controls
* [ ] Use GitHub OIDC instead of long-lived cloud credentials where applicable

---

# ⭐ Skills Demonstrated

```text
DevOps
DevSecOps
CI/CD
GitHub Actions
Git
Docker
Docker Hub
SonarQube
SAST
Trivy
Container Security
Kubernetes
AKS
GitOps
Argo CD
YAML
Linux
Shell Scripting
Cloud-Native Deployment
```


## 📄 Project Repository

The complete source code, GitHub Actions workflows, Dockerfile, Kubernetes manifests and application source are maintained in this repository.
