# Progree DevOps Internship — Complete Project

This repository contains all scaffolding and configuration for **Tasks 2–4** of the Progree DevOps internship. Task 1 (LinkedIn announcement) is handled separately.

## Project Structure

```
progree-devops-project/
├── backend/                 # Node.js API server
├── frontend/                # React app (built with Vite)
├── Dockerfile               # Multi-stage build for container
├── docker-compose.yml       # Local development with PostgreSQL
├── .dockerignore            # Excludes unnecessary files from build context
├── .env.example             # Template for environment variables
├── .github/
│   └── workflows/
│       └── ci-cd.yml        # GitHub Actions CI/CD pipeline
├── terraform/
│   └── main.tf              # AWS VPC infrastructure as code
├── k8s/                     # Kubernetes manifests
│   ├── config.yaml          # ConfigMap + Secret
│   ├── pvc.yaml             # PersistentVolumeClaim
│   ├── deployment.yaml      # Deployment (2 replicas)
│   ├── service.yaml         # ClusterIP service
│   ├── ingress.yaml         # NGINX Ingress
│   └── hpa.yaml             # Horizontal Pod Autoscaler
└── README.md                # This file
```

---

## TASK 2: Application Containerization & Asset Optimization

### Overview
- **Multi-stage Dockerfile** with 3 stages: frontend build → backend build → minimal runtime
- **Alpine base image** (`node:20-alpine` ~40MB) for minimal footprint
- **Non-root user** (`appuser`) for security
- **Health check** for container readiness
- **Environment variable mapping** via `.env` file (never hardcoded)

### Quick Start

**1. Copy the example environment file:**
```bash
cp .env.example .env
```

**2. Build the Docker image:**
```bash
docker build -t myapp:latest .
```

**3. Check image size:**
```bash
docker images myapp
# Should be ~150-200MB (Alpine-based, much smaller than non-Alpine)
```

**4. Run with docker-compose (includes PostgreSQL):**
```bash
docker compose up -d
```

**5. Verify the app:**
```bash
curl http://localhost:5000/health
# Expected: {"status":"ok"}
```

**6. View logs:**
```bash
docker compose logs -f backend
```

**7. Cleanup:**
```bash
docker compose down
docker compose down -v  # Also removes persistent volumes
```

### Key Points for Your Report
- **Image size documented**: `docker images myapp` shows final size
- **Multi-stage layers**: Frontend and backend built separately, only runtime copied
- **Security**: Non-root user (`appuser`), no dev dependencies, no build cache
- **Port mapping**: `5000:5000` (container:host)
- **Secrets**: Never in Dockerfile — injected via `.env` + docker-compose

### Files
- [Dockerfile](Dockerfile) — Multi-stage build
- [.dockerignore](.dockerignore) — Excludes node_modules, .env, etc.
- [docker-compose.yml](docker-compose.yml) — Service orchestration (backend + PostgreSQL)
- [.env.example](.env.example) — Environment template

---

## TASK 3: Multi-Stage CI/CD Pipeline (GitHub Actions)

### Overview
- **Lint** → **Test** → **Build** → **Notify**
- Runs on every push to `main` or `develop`; on PRs to `main`
- Artifacts: coverage reports uploaded; Docker image built with commit SHA tag

### Quick Start

**1. Push to GitHub:**
```bash
git init
git add .
git commit -m "Initial Progree DevOps project setup"
git branch -M main
git remote add origin https://github.com/<YOUR_USERNAME>/<YOUR_REPO>.git
git push -u origin main
```

**2. Check Actions tab on GitHub:**
- Open https://github.com/<YOUR_USERNAME>/<YOUR_REPO>/actions
- You'll see the workflow running through lint → test → build → notify
- Each stage shows pass/fail status

**3. Take a screenshot of the passing pipeline** for your report.

### Pipeline Stages

| Stage  | Trigger            | Action                                   |
|--------|-------------------|------------------------------------------|
| lint   | Every push/PR     | `npm run lint` (runs ESLint)              |
| test   | After lint passes | `npm test -- --coverage` (Jest + coverage) |
| build  | After tests pass  | `docker build -t myapp:${{ github.sha }}` |
| notify | Always runs       | Writes summary to Actions tab             |

### Key Points for Your Report
- **Automated on every push**: No manual builds needed
- **Sequential dependencies**: lint → test → build (parallelizable, but shown sequentially for clarity)
- **Coverage artifact**: Uploaded to GitHub after tests
- **Docker image tagged with commit SHA**: Traceability
- **Status summary**: Visible in GitHub Actions tab

### Files
- [.github/workflows/ci-cd.yml](.github/workflows/ci-cd.yml) — GitHub Actions workflow

### Note on npm scripts
If you get errors like `lint` or `test` not found:
```bash
npm run lint  # Will show "Lint step placeholder" (add real linter as needed)
npm test      # Will show "Test step placeholder" (add real tests as needed)
```
For production, replace placeholders in [package.json](package.json) with real tools (ESLint, Jest, etc.).

---

## TASK 4: Infrastructure as Code + Kubernetes Orchestration

### Part A: Terraform — AWS VPC Provisioning

**Overview:**
- Creates an isolated `10.0.0.0/16` VPC with 2 public subnets across availability zones
- Internet Gateway for public access
- Route table for subnet routing

**Quick Start:**

**1. Install Terraform:**
```bash
# macOS
brew install terraform

# Windows (via Chocolatey)
choco install terraform

# Or download from https://www.terraform.io/downloads
```

**2. Configure AWS credentials** (if applying to real AWS):
```bash
aws configure
# Enter your AWS Access Key ID and Secret Access Key
```

**3. Initialize Terraform:**
```bash
cd terraform
terraform init
```

**4. Plan the infrastructure** (review changes without applying):
```bash
terraform plan
# Shows: VPC, 2 subnets, IGW, route table, 2 associations
```

**5. Apply** (create resources on AWS):
```bash
terraform apply -auto-approve
```

**6. Get the VPC ID output:**
```bash
terraform output vpc_id
```

**7. Cleanup** (destroy resources):
```bash
terraform destroy -auto-approve
```

### Key Points for Your Report
- **VPC isolation**: `10.0.0.0/16` network with no overlaps
- **Multi-AZ**: 2 subnets across different availability zones
- **Public routing**: Internet Gateway + route table
- **Infrastructure as Code**: All defined in `main.tf`, reproducible and versioned

### Files
- [terraform/main.tf](terraform/main.tf) — AWS VPC infrastructure

---

### Part B: Kubernetes Orchestration — Minikube

**Overview:**
- **Deployment** (2 replicas) running the containerized app
- **PersistentVolumeClaim** (1Gi storage) for data persistence
- **Service** (ClusterIP) for internal routing
- **Ingress** (NGINX) for external access at `myapp.local`
- **HPA** (Horizontal Pod Autoscaler) scales replicas based on CPU usage

**Quick Start:**

**1. Install Minikube** (local Kubernetes for testing):
```bash
# macOS
brew install minikube

# Windows (via Chocolatey)
choco install minikube

# Or download from https://minikube.sigs.k8s.io/docs/start/
```

**2. Start Minikube:**
```bash
minikube start --driver=docker
```

**3. Enable required addons:**
```bash
minikube addons enable ingress
minikube addons enable metrics-server  # Required for HPA CPU metrics
```

**4. Build the Docker image directly into Minikube:**
```bash
eval $(minikube docker-env)  # Windows: run 'minikube docker-env | Invoke-Expression'
docker build -t myapp:latest .
```

**5. Deploy all manifests:**
```bash
kubectl apply -f k8s/config.yaml
kubectl apply -f k8s/pvc.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingress.yaml
kubectl apply -f k8s/hpa.yaml
```

**6. Verify deployment:**
```bash
# Check pods are running
kubectl get pods
# Expected: myapp-deployment-xxxx with 2 replicas

# Check service
kubectl get svc myapp-service
# Expected: ClusterIP service on port 80

# Check PVC is bound
kubectl get pvc
# Expected: myapp-pvc bound to a PersistentVolume

# Check HPA is tracking
kubectl describe hpa myapp-hpa
# Expected: Metrics: <unknown> / 70%
```

**7. Access the app:**
```bash
minikube service myapp-service --url
# Opens in browser (or gives URL to curl)
```

**8. Simulate load for HPA scaling:**
```bash
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://myapp-service; done"
# In another terminal, watch pods scale:
kubectl get hpa -w
# Expected: replicas increase from 2 to 3–6 as CPU usage rises
```

**9. Cleanup:**
```bash
kubectl delete -f k8s/
minikube stop
minikube delete
```

### Key Points for Your Report
- **High availability**: 2 replicas, automatic restart on failure
- **Persistent storage**: PVC ensures data survives pod restarts
- **Service discovery**: ClusterIP service routes traffic
- **Ingress routing**: NGINX ingress exposes app externally
- **Auto-scaling**: HPA scales from 2–6 replicas based on CPU
- **Screenshots**: Capture `kubectl get pods`, `kubectl get svc`, `kubectl describe hpa` output

### Files
- [k8s/config.yaml](k8s/config.yaml) — ConfigMap + Secret
- [k8s/pvc.yaml](k8s/pvc.yaml) — PersistentVolumeClaim
- [k8s/deployment.yaml](k8s/deployment.yaml) — Deployment (2 replicas)
- [k8s/service.yaml](k8s/service.yaml) — ClusterIP Service
- [k8s/ingress.yaml](k8s/ingress.yaml) — NGINX Ingress
- [k8s/hpa.yaml](k8s/hpa.yaml) — Horizontal Pod Autoscaler

---

## Submission Checklist

- [ ] **Task 1**: LinkedIn post live, Progree tagged, offer letter attached *(handled separately)*
- [ ] **Task 2**: 
  - [ ] Dockerfile builds successfully
  - [ ] `docker images myapp` shows image size (capture screenshot)
  - [ ] `docker compose up` runs backend + PostgreSQL
  - [ ] `curl http://localhost:5000/health` returns `{"status":"ok"}`
  - [ ] Files in repo: `Dockerfile`, `.dockerignore`, `docker-compose.yml`, `.env.example`
- [ ] **Task 3**:
  - [ ] `.github/workflows/ci-cd.yml` pushed to main branch
  - [ ] GitHub Actions tab shows green checkmarks for lint, test, build, notify
  - [ ] Screenshot of passing pipeline
  - [ ] Coverage artifact uploaded (visible in Actions tab)
- [ ] **Task 4a — Terraform**:
  - [ ] `terraform init` succeeds
  - [ ] `terraform plan` shows VPC, 2 subnets, IGW, route table
  - [ ] `terraform apply -auto-approve` completes (or documented if skipping real AWS spend)
  - [ ] `terraform output vpc_id` shows VPC ID
- [ ] **Task 4b — Kubernetes**:
  - [ ] `minikube start` and addons enabled
  - [ ] `kubectl apply -f k8s/` applies all manifests
  - [ ] `kubectl get pods` shows 2 myapp-deployment replicas
  - [ ] `kubectl get svc` shows myapp-service ClusterIP
  - [ ] `kubectl get pvc` shows myapp-pvc Bound
  - [ ] `kubectl describe hpa myapp-hpa` shows HPA tracking CPU
  - [ ] Screenshots of each `kubectl get` command

---

## Troubleshooting

### Docker Issues
- **"Cannot connect to Docker daemon"**: Ensure Docker Desktop is running
- **"image not found"**: Run `docker build -t myapp:latest .` first
- **Port already in use**: `docker compose down` or change ports in `docker-compose.yml`

### GitHub Actions Issues
- **"npm run lint" not found**: Lint script is a placeholder; add ESLint if needed
- **"npm test" not found**: Test script is a placeholder; add Jest if needed
- **Workflow not triggering**: Ensure `.github/workflows/ci-cd.yml` is on main branch

### Kubernetes Issues
- **Pods not starting**: `kubectl describe pod <pod-name>` for error logs
- **HPA not scaling**: Ensure `metrics-server` addon is enabled
- **"myapp:latest not found"**: Build image into Minikube: `eval $(minikube docker-env)` then `docker build -t myapp:latest .`
- **Ingress not working**: Use `minikube service myapp-service --url` instead

---

## Quick Reference Commands

### Docker
```bash
docker build -t myapp:latest .
docker images
docker run -p 5000:5000 myapp:latest
docker compose up -d
docker compose logs -f backend
docker compose down
```

### GitHub & Git
```bash
git add .
git commit -m "message"
git push origin main
```

### Terraform
```bash
cd terraform
terraform init
terraform plan
terraform apply -auto-approve
terraform destroy -auto-approve
```

### Kubernetes / Minikube
```bash
minikube start --driver=docker
minikube addons enable ingress metrics-server
kubectl apply -f k8s/
kubectl get pods,svc,pvc,hpa
kubectl describe hpa myapp-hpa
kubectl logs -f deployment/myapp-deployment
kubectl delete -f k8s/
minikube stop
```

---

## Next Steps

1. **Task 2**: Test Docker build locally, capture image size
2. **Task 3**: Push to GitHub, verify Actions tab, screenshot pipeline
3. **Task 4a**: Run Terraform (AWS costs apply if real; can skip for demo)
4. **Task 4b**: Deploy to Minikube, verify all components, capture screenshots

Good luck with your Progree internship! 🚀
#   P r o g r e e - D e v O p s - I n t e r n s h i p  
 