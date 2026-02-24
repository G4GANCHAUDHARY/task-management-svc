DEPLOYMENT GUIDE - TASK MANAGEMENT API

Actual Implementation & Solution Approach


TABLE OF CONTENTS
-----------------
1. Overview
2. Dockerization (Step 1)
3. Kubernetes Deployment (Step 2)
4. CI/CD Pipeline (Step 3)
5. Challenges & Solutions
6. How to Test Our Solution



1. OVERVIEW
===============================================================================

This document describes how we solved the DevOps Pipeline Challenge for the
Task Management API. Our approach focused on:

✓ Containerization with Docker best practices
✓ Kubernetes deployment with kind (local cluster)
✓ CI/CD pipeline with GitHub Actions (demo mode)
✓ Comprehensive documentation

Tools Used:
- Docker Desktop (with Kubernetes enabled)
- kind (Kubernetes in Docker) for local cluster
- GitHub Actions for CI/CD
- kubectl for cluster management



2. DOCKERIZATION (STEP 1)
===============================================================================

2.1 Dockerfile Design Decisions
-------------------------------------------------------------------------------
We created a multi-stage Dockerfile with these key decisions:


Base Image : python:3.11-slim, Minimal, secure, sufficient 
Multi-stage : Builder + Production, Smaller final image 
Health Check : HEALTHCHECK instruction For Kubernetes probes 

2.2 Final Dockerfile
-------------------------------------------------------------------------------
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

FROM python:3.11-slim
RUN useradd --create-home appuser
WORKDIR /app
COPY --from=builder /root/.local /home/appuser/.local
COPY src/ ./src/
ENV PATH=/home/appuser/.local/bin:$PATH
USER appuser
EXPOSE 8000
HEALTHCHECK CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')" || exit 1
CMD ["uvicorn", "src.app:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]

2.3 Building & Testing Locally
-------------------------------------------------------------------------------
# Build with production tag
docker build -t task-api:prod .

# Run container
docker run -d -p 8000:8000 --name task-api-test task-api:prod

# Verify
curl http://localhost:8000/health
curl http://localhost:8000/docs


3. KUBERNETES DEPLOYMENT (STEP 2)
===============================================================================

3.1 Local Cluster Setup with kind
-------------------------------------------------------------------------------
We used Docker Desktop's built-in Kubernetes with kind provisioner:

1. Enabled Kubernetes in Docker Desktop:
   - Docker Desktop → Preferences → Kubernetes → Enable Kubernetes
   - Selected "kind" as cluster provisioner
   - Set nodes to 3 for high availability

2. Verified cluster:
   kubectl get nodes

3.2 Kubernetes Manifests Created
-------------------------------------------------------------------------------
We created two main manifests as required:

A. deployment.yaml
   - 3 replicas for high availability
   - Resource limits: 256Mi memory, 500m CPU
   - Liveness probe: /health endpoint
   - Readiness probe: /health endpoint

B. service.yaml
   - ClusterIP type (internal access)
   - Port 80 → 8000 mapping

3.3 Deployment Process
-------------------------------------------------------------------------------
# Create namespace
kubectl create namespace task-management

# Load image into kind cluster
kind load docker-image task-api:prod --name kind

# Apply manifests
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Wait for pods
kubectl wait --for=condition=ready pods -l app=task-api -n task-management

# Verify deployment
kubectl get pods -n task-management

# Access the service
kubectl port-forward -n task-management service/task-api-service 8080:80 &
curl http://localhost:8080/health

3.4 Challenges Faced & Solutions
-------------------------------------------------------------------------------
Challenge and Solutions

1. ImagePullBackOff error - Used kind load docker-image to load local image 
2. Namespace not found - Added namespace: task-management to YAML files 
3. Connection refused - Verified cluster running with kubectl get nodes 


4. CI/CD PIPELINE (STEP 3)
===============================================================================

4.1 Pipeline Design Decisions
-------------------------------------------------------------------------------
Since we couldn't expose our local cluster to the internet, we implemented a
DEMO deployment that shows the actual commands without requiring a live cluster.

4.2 GitHub Actions Workflow
-------------------------------------------------------------------------------
Location: .github/workflows/ci-cd.yml

The pipeline has three jobs:

JOB 1: TEST
- Runs on: ubuntu-latest
- Steps:
  - Checkout code
  - Setup Python 3.11
  - Install dependencies
  - Run pytest (17 tests)
- Purpose: Ensures code quality

JOB 2: BUILD AND PUSH
- Runs on: ubuntu-latest (only on push to main)
- Steps:
  - Login to GitHub Container Registry
  - Build Docker image with SHA tag
  - Push to GHCR
- Purpose: Creates deployable artifact

JOB 3: DEPLOY (DEMO MODE)
- Runs on: ubuntu-latest (only on push to main)
- Steps:
  - Shows deployment commands that would run
  - Displays expected output
  - No actual cluster required
- Purpose: Demonstrates deployment knowledge

4.3 Pipeline Execution
-------------------------------------------------------------------------------
When you push to main:
1. ✓ Tests run automatically
2. ✓ Docker image built and pushed to GHCR
3. ✓ Deployment demo shows what would happen

View pipeline: https://github.com/G4GANCHAUDHARY/task-management-svc/actions

4.4 Why Demo Mode?
-------------------------------------------------------------------------------
We chose demo deployment because:
- GitHub Actions runners cannot access local kind cluster (127.0.0.1)
- Self-hosted runner would require keeping machine online
- Demo shows the exact commands without infrastructure complexity
- Meets the requirement to "Deploy to Kubernetes cluster" conceptually



6. HOW TO TEST OUR SOLUTION
===============================================================================

6.1 Prerequisites
-------------------------------------------------------------------------------
- Docker Desktop (with Kubernetes enabled)
- kubectl
- kind
- Python 3.11+

6.2 Step-by-Step Testing
-------------------------------------------------------------------------------

Step A: Test Docker Image
-------------------------
git clone https://github.com/G4GANCHAUDHARY/task-management-svc
cd task-management-svc
docker build -t task-api:test .
docker run -d -p 8000:8000 --name test-api task-api:test
curl http://localhost:8000/health
docker stop test-api && docker rm test-api

Step B: Test Kubernetes Deployment (Local)
-----------------------------------------
# Start kind cluster
kind create cluster --name test-cluster

# Load image
kind load docker-image task-api:prod --name test-cluster

# Deploy
kubectl create namespace task-management
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Verify
kubectl get pods -n task-management
kubectl port-forward -n task-management service/task-api-service 8080:80 &
curl http://localhost:8080/health

# Clean up
kubectl delete namespace task-management
kind delete cluster --name test-cluster

Step C: Test CI/CD Pipeline
--------------------------
1. Go to GitHub repository
2. Click on Actions tab
3. See latest workflow run
4. Verify test job passes (17 tests)
5. Verify build job pushes to GHCR
6. Verify deploy job shows demo output