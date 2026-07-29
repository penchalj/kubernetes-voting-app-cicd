# 3-Tier Voting App on Amazon EKS

A production-like microservices voting application deployed on **Amazon EKS** with full **CI/CD** using GitHub Actions, NGINX Ingress Controller, Redis, and PostgreSQL.

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Amazon EKS](https://img.shields.io/badge/Amazon_EKS-FF9900?style=for-the-badge&logo=amazon-eks&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=helm&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![.NET](https://img.shields.io/badge/.NET-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![NGINX](https://img.shields.io/badge/NGINX-009639?style=for-the-badge&logo=nginx&logoColor=white)

---

## Project Overview

This project demonstrates a complete **cloud-native DevOps workflow**:

- Containerized microservices (Vote, Result, Worker)
- Orchestrated on **Amazon EKS** (Managed Kubernetes)
- Automated **CI/CD pipeline** with GitHub Actions
- Path-based routing using **NGINX Ingress Controller**
- Persistent data with **PostgreSQL** + **Redis**
- Secure configuration using **Kubernetes Secrets**

**Voting Flow:**
User → Vote App (Python) → Redis → Worker (.NET) → PostgreSQL → Result App (Node.js)

### Application Components

- **Vote (Python)**: A Flask-based web app where users cast a vote between two options.
- **Redis (in-memory queue)**: Collects incoming votes and temporarily stores them.
- **Worker (.NET)**: A .NET 7.0 service that consumes votes from Redis and persists them to Postgres.
- **Postgres (Database)**: Stores votes for long-term persistence.
- **Result (Node.js/Express)**: Displays vote counts in real time.

This stack is intentionally polyglot (Python, Node.js, .NET) to give hands-on practice with multiple runtimes, container orchestration, and the kind of "messy" multi-service troubleshooting you'll encounter in real deployments.

---

## Architecture

```
Internet
   ↓
AWS Network Load Balancer (created by NGINX Ingress)
   ↓
NGINX Ingress Controller
   ├── /vote   → Vote Service   (Python)
   └── /result → Result Service (Node.js)
                       ↑
                  PostgreSQL
                       ↑
                  Worker (.NET)
                       ↑
                     Redis
```

**Mermaid Diagram (renders on GitHub):**

```mermaid
flowchart TD
    User[User Browser] -->|HTTP| ALB[AWS Load Balancer]
    ALB --> Ingress[NGINX Ingress Controller]
    Ingress -->|/vote| Vote[Vote Service<br/>Python]
    Ingress -->|/result| Result[Result Service<br/>Node.js]
    Vote --> Redis[(Redis)]
    Redis --> Worker[Worker Service<br/>.NET]
    Worker --> Postgres[(PostgreSQL)]
    Result --> Postgres
```

---

## Tech Stack

| Layer                    | Technology                  | Purpose                              |
|--------------------------|-----------------------------|--------------------------------------|
| Container Orchestration  | Amazon EKS (Kubernetes)     | Managed Kubernetes control plane     |
| Vote Service             | Python                      | Frontend for casting votes           |
| Result Service           | Node.js                     | Frontend for viewing live results    |
| Worker Service           | .NET                        | Background processor                 |
| Cache / Queue            | Redis                       | Temporary vote storage               |
| Database                 | PostgreSQL                  | Persistent vote storage              |
| Ingress                  | NGINX Ingress Controller    | Path-based routing (`/vote`, `/result`) |
| CI/CD                    | GitHub Actions              | Build → Push → Deploy automation     |
| Container Registry       | Docker Hub                  | Image storage                        |
| Cluster Provisioning     | eksctl                      | Easy EKS cluster creation            |
| Package Manager          | Helm                        | Install NGINX Ingress                |

---

## Features

- Fully containerized **3-tier microservices** architecture
- Automated **build, push, and deploy** using GitHub Actions
- Path-based routing (`/vote` and `/result`) via NGINX Ingress
- Kubernetes **Secrets** for sensitive data (DB credentials)
- Clean and modular Kubernetes manifests
- Service discovery via Kubernetes DNS (`redis-service`, `db`, etc.)
- Horizontal Pod Autoscaling ready
- Production-like setup on Amazon EKS
- Optional HTTPS with cert-manager + Let's Encrypt (addon)

---

## Prerequisites

- AWS Account with sufficient permissions (EKS, EC2, IAM, VPC)
- `eksctl`, `kubectl`, and `helm` installed locally
- Docker Hub account
- GitHub repository with Actions enabled
- AWS CLI configured (`aws configure`)
- (For local/non-EKS runs) Python 3.10+, Node.js 18+, .NET 7.0 SDK, Docker

---

## Project Structure

```
.
├── .github/
│   └── workflows/
│       └── ci-cd-pipeline.yml       # GitHub Actions CI/CD
├── k8s/
│   ├── redis-deployment.yml
│   ├── postgres-deployment.yml
│   ├── postgres-pvc.yml
│   ├── vote-deployment.yaml
│   ├── result-deployment.yml
│   ├── worker-deployment.yaml
│   ├── voting-app-ingress.yaml
│   └── db-service.yaml                 # or create via kubectl
├── vote/                            # Python Vote app source
├── result/                          # Node.js Result app source
├── worker/                          # .NET Worker app source
└── README.md
```

---

## Running Locally (Without Kubernetes)

Each service can also be run standalone for quick development/debugging.

### Vote (Python)

```bash
cd vote
pip install -r requirements.txt
python app.py
```
Access at [http://localhost:5000](http://localhost:5000).

### Redis

```bash
redis-server
```
Available at `localhost:6379`.

### Worker (.NET)

```bash
cd worker
dotnet restore
dotnet run
```
Connects to Redis and Postgres when available.

### Postgres

Install from [postgresql.org/download](https://www.postgresql.org/download/) and start the service (default user/password: `postgres`/`postgres`). Available at `localhost:5432`.

### Result (Node.js)

```bash
cd result
npm install
node server.js
```
Access at [http://localhost:4000](http://localhost:4000).

> To see votes flow end-to-end locally, all five components need to be running with environment variables/connection strings pointing at each other.

### Running the Full Stack with Docker Compose

The simplest way to run everything locally:

```bash
docker compose up
```

This builds and runs the vote, worker, and result services, plus Redis and Postgres from their official images, wiring up networks, volumes, and environment variables automatically.

Visit [http://localhost:8080](http://localhost:8080) to vote and [http://localhost:8081](http://localhost:8081) to see results.

### Building Individual Docker Images

```bash
# Vote (Python)
docker build -t myorg/vote:latest ./vote
docker run --name vote -p 8080:80 myorg/vote:latest

# Redis (official image)
docker run --name redis -p 6379:6379 redis:alpine

# Worker (.NET)
docker build -t myorg/worker:latest ./worker
docker run --name worker myorg/worker:latest

# Postgres
docker run --name db -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -p 5432:5432 postgres:15-alpine

# Result (Node.js)
docker build -t myorg/result:latest ./result
docker run --name result -p 8081:80 myorg/result:latest
```

### Notes on Platforms (arm64 vs amd64)

On arm64 machines (e.g. Apple Silicon M1/M2), if you hit issues with images built for amd64, use `buildx`:

```bash
docker buildx build --platform linux/amd64 -t myorg/worker:latest ./worker
```

---

## Deploying to Amazon EKS

### 1. Create the EKS Cluster

```bash
eksctl create cluster \
  --name voting-app-cluster \
  --region us-east-1 \
  --nodes 3 \
  --node-type t3.medium \
  --managed
```

> Use `--managed` for managed node groups (recommended). Cluster creation usually takes 15–20 minutes.

### 2. Create Kubernetes Secrets (Recommended)

```bash
kubectl create secret generic db-credentials \
  --from-literal=POSTGRES_USER=postgres \
  --from-literal=POSTGRES_PASSWORD=YourSecurePassword123!
```

Then reference them in your Deployment manifests:

```yaml
env:
  - name: POSTGRES_USER
    valueFrom:
      secretKeyRef:
        name: db-credentials
        key: POSTGRES_USER
  - name: POSTGRES_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-credentials
        key: POSTGRES_PASSWORD
```

### 3. Deploy Redis & PostgreSQL

```bash
kubectl apply -f k8s/redis-deployment.yaml
kubectl apply -f k8s/redis-service.yaml
kubectl apply -f k8s/postgres-deployment.yaml
kubectl apply -f k8s/postgres-service.yaml
```

### 4. Deploy the Microservices

```bash
kubectl apply -f k8s/vote-deployment.yaml
kubectl apply -f k8s/vote-service.yaml
kubectl apply -f k8s/result-deployment.yaml
kubectl apply -f k8s/result-service.yaml
kubectl apply -f k8s/worker-deployment.yaml
```

> **Important:** Microservices communicate using Kubernetes DNS names (e.g. `redis-service`, `db`, or `postgres-service`). Never hard-code IP addresses!

### 5. Install NGINX Ingress Controller (Helm)

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install my-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace
```

Verify:

```bash
kubectl get pods -n ingress-nginx
```

### 6. Create Ingress Resource

```bash
kubectl apply -f k8s/ingress.yaml
```

Example `ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: voting-app-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
    - http:
        paths:
          - path: /vote
            pathType: Prefix
            backend:
              service:
                name: vote-service
                port:
                  number: 80
          - path: /result
            pathType: Prefix
            backend:
              service:
                name: result-service
                port:
                  number: 80
```

### 7. Access the Application

```bash
kubectl get ingress
```

Wait until the `ADDRESS` field is populated (can take 1–5 minutes). Then open:

- **Vote app** → `http://<INGRESS_EXTERNAL_IP>/vote`
- **Result app** → `http://<INGRESS_EXTERNAL_IP>/result`

---

## CI/CD Pipeline (GitHub Actions)

The pipeline automatically:

1. Builds Docker images for `vote`, `result`, and `worker`
2. Pushes them to Docker Hub
3. Configures AWS credentials
4. Updates kubeconfig for the EKS cluster
5. Applies the Kubernetes manifests

**Trigger:** Push to the `main` branch (or manual `workflow_dispatch`)

### Required GitHub Secrets

| Secret Name | Description |
|---|---|
| `DOCKERHUB_USERNAME` | Your Docker Hub username |
| `DOCKERHUB_TOKEN` | Docker Hub access token |
| `AWS_ACCESS_KEY_ID` | AWS IAM access key |
| `AWS_SECRET_ACCESS_KEY` | AWS IAM secret key |

### Example Workflow Snippet

```yaml
- name: Configure AWS Credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: us-east-1

- name: Update Kubeconfig
  run: |
    aws eks update-kubeconfig --name voting-app-cluster --region us-east-1

- name: Apply Kubernetes Manifests
  run: kubectl apply -f k8s/
```

---

## Validation & Demo Checklist

After the pipeline succeeds (or after manual deployment):

```bash
# Check Deployments
kubectl get deployments

# Check Pods
kubectl get pods

# Check Services
kubectl get svc

# Check Ingress
kubectl get ingress

# Check logs (example)
kubectl logs -l app=vote
kubectl logs -l app=worker
```

**End-to-end test:**

1. Open `/vote` and cast several votes
2. Open `/result` and watch the results update in real time
3. Confirm the flow: **Vote → Redis → Worker → PostgreSQL → Result**

---

## Cleanup

### Full Cleanup Sequence

```bash
# 1. Delete application resources
kubectl delete -f k8s/ --ignore-not-found

# 2. Uninstall NGINX Ingress Controller
helm uninstall my-nginx -n ingress-nginx 2>/dev/null
kubectl delete namespace ingress-nginx --ignore-not-found

# 3. Delete the EKS cluster
eksctl delete cluster --name voting-app-cluster --region us-east-1 --wait
```

### Quick Check Before Deleting

```bash
# List your clusters
eksctl get cluster

# Or specifically
eksctl get cluster --name voting-app-cluster --region us-east-1
```

### One-Liner

```bash
kubectl delete -f k8s/ --ignore-not-found && \
helm uninstall my-nginx -n ingress-nginx 2>/dev/null; \
kubectl delete ns ingress-nginx --ignore-not-found && \
eksctl delete cluster --name voting-app-cluster --region us-east-1 --wait
```

After the cluster deletion finishes (it can take 10–15 minutes), verify in the AWS Console that the EKS cluster, node groups, and associated Load Balancers are gone.

---

## Project Add-ons

### HTTPS with cert-manager + Let's Encrypt

1. Install cert-manager
2. Create a `ClusterIssuer` for Let's Encrypt
3. Annotate your Ingress with the issuer
4. Point a Route 53 domain to the Ingress Load Balancer
5. Certificates will be issued and renewed automatically

### Horizontal Pod Autoscaler (HPA)

Ready to add:

```bash
kubectl autoscale deployment vote --cpu-percent=50 --min=2 --max=10
```

### Monitoring (optional)

- Prometheus + Grafana
- AWS CloudWatch Container Insights

---

## Troubleshooting

| Issue | Solution |
|---|---|
| Ingress `ADDRESS` stays `<pending>` | Wait 2–5 min or check AWS Load Balancer |
| Pods `CrashLoopBackOff` | Check logs + Secrets + environment variables |
| Services cannot reach Redis/DB | Verify Service names & DNS (same namespace) |
| CI/CD fails on kubeconfig | Ensure IAM user has EKS permissions |
| Images not updating | Use unique tags or `imagePullPolicy: Always` |

---

## Useful Commands Cheat Sheet

```bash
# Cluster
eksctl get cluster
kubectl get nodes

# Workloads
kubectl get all
kubectl describe pod <pod-name>
kubectl logs -f <pod-name>

# Cleanup
kubectl delete -f k8s/
eksctl delete cluster --name voting-app-cluster --region us-east-1
```

---

## Skills You Will Learn

- Managed Kubernetes (Amazon EKS)
- Container Orchestration with Deployments & Services
- Path-based routing with NGINX Ingress
- Kubernetes Secrets management
- GitHub Actions CI/CD for cloud-native apps
- Service discovery inside Kubernetes
- Production-ready microservices patterns
- Multi-runtime containerization (Python, Node.js, .NET) with Docker & Docker Compose

---

Good luck! This project gives you practical, end-to-end exposure to modern cloud-native DevOps workflows — from container builds and local Docker Compose runs all the way to automated deployment on Amazon EKS.

> **Note**: Application source code (vote, result, worker) is based on
> [Pokfinner/ironhack-project-1](https://github.com/Pokfinner/ironhack-project-1) (Apache 2.0 License).

<!-- © 2024 | Ironhack -->