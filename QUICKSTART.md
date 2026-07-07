# Quick Start Guide — Progree DevOps Tasks

## All-in-One Command Reference

### Task 2: Docker & Container Optimization
```bash
cd C:\Users\usman\progree-devops-project

# Setup
cp .env.example .env

# Build & verify
docker build -t myapp:latest .
docker images myapp                    # Check size (~156MB)
docker compose up -d
curl http://localhost:5000/health      # Should return {"status":"ok"}
docker compose ps                       # Verify both services running
docker compose logs -f backend          # View logs
docker compose down                     # Cleanup
```

**Key deliverables:**
- ✅ Docker image built (156MB or less)
- ✅ Health check passes
- ✅ docker-compose up/down works
- ✅ Port 5000 accessible

---

### Task 3: CI/CD Pipeline (GitHub Actions)
```bash
# Git setup
git init
git add .
git commit -m "Initial Progree DevOps project"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git push -u origin main

# Check GitHub Actions tab
# Expected: lint → test → build → notify all ✓
```

**Key deliverables:**
- ✅ Workflow file pushed: `.github/workflows/ci-cd.yml`
- ✅ Pipeline runs automatically on push
- ✅ All stages (lint, test, build, notify) pass
- ✅ Screenshot of Actions tab

---

### Task 4A: Terraform (AWS Infrastructure)
```bash
# Setup
aws configure                          # Enter AWS credentials
cd terraform
terraform init

# Plan & apply
terraform plan                         # Review resources
terraform apply -auto-approve         # Create VPC (costs may apply)
terraform output vpc_id               # Get VPC ID
terraform destroy -auto-approve       # Cleanup when done
```

**Key deliverables:**
- ✅ Terraform plan shows 6 resources
- ✅ VPC created (10.0.0.0/16)
- ✅ 2 subnets, IGW, route table visible in AWS Console

---

### Task 4B: Kubernetes (Minikube + Orchestration)
```bash
# Setup
minikube start --driver=docker
minikube addons enable ingress
minikube addons enable metrics-server

# Build image into Minikube
eval $(minikube docker-env)            # Windows: minikube docker-env | Invoke-Expression
docker build -t myapp:latest .

# Deploy all Kubernetes manifests
kubectl apply -f k8s/config.yaml
kubectl apply -f k8s/pvc.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingress.yaml
kubectl apply -f k8s/hpa.yaml

# Verify
kubectl get pods                        # 2 replicas running
kubectl get svc                         # myapp-service ClusterIP
kubectl get pvc                         # myapp-pvc Bound
kubectl describe hpa myapp-hpa         # HPA tracking CPU
kubectl logs -f deployment/myapp-deployment

# Access app
minikube service myapp-service --url

# Test auto-scaling
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://myapp-service; done"
kubectl get hpa -w                      # Watch replicas increase

# Cleanup
kubectl delete -f k8s/
minikube stop
minikube delete
```

**Key deliverables:**
- ✅ 2 pods running
- ✅ Service, PVC, Ingress, HPA all created
- ✅ HPA auto-scales under load
- ✅ Screenshots of kubectl get/describe commands

---

## Checklist: All Tasks

### Task 1: LinkedIn (Not in this repo)
- [ ] Post written with offer letter attached
- [ ] Progree tagged
- [ ] Professional caption

### Task 2: Docker
- [ ] Dockerfile builds successfully
- [ ] Image size < 200MB (captured in screenshot)
- [ ] `docker compose up` works
- [ ] Health check passes
- [ ] Port 5000 accessible
- [ ] Files committed: Dockerfile, docker-compose.yml, .dockerignore, .env.example

### Task 3: CI/CD
- [ ] Pushed to GitHub main branch
- [ ] Actions tab shows passing workflow
- [ ] All stages (lint, test, build, notify) ✓
- [ ] Screenshot of Actions tab
- [ ] Workflow file: `.github/workflows/ci-cd.yml` in repo

### Task 4A: Terraform
- [ ] `terraform init` succeeds
- [ ] `terraform plan` shows 6 resources
- [ ] `terraform apply` creates VPC
- [ ] AWS Console shows vpc-xxx (10.0.0.0/16)
- [ ] Screenshot of terraform output
- [ ] File: `terraform/main.tf` in repo

### Task 4B: Kubernetes
- [ ] `minikube start` succeeds
- [ ] Addons (ingress, metrics-server) enabled
- [ ] `kubectl apply -f k8s/` applies all manifests
- [ ] All pods running: `kubectl get pods` shows 2 replicas
- [ ] Service created: `kubectl get svc` shows myapp-service
- [ ] PVC bound: `kubectl get pvc` shows Bound status
- [ ] HPA active: `kubectl describe hpa` shows metrics
- [ ] Screenshots of all `kubectl get` commands
- [ ] Auto-scaling demo (HPA scales from 2 → 4–6 under load)

---

## File Structure

```
C:\Users\usman\progree-devops-project/
├── Dockerfile                 # Multi-stage build (3 stages)
├── docker-compose.yml        # Backend + PostgreSQL
├── .dockerignore             # Excludes for build context
├── .env.example              # Environment template
├── .gitignore                # Git ignore rules
├── package.json              # Root workspace config
├── README.md                 # Full project documentation
├── TASK2.md                  # Docker detailed guide
├── TASK3.md                  # CI/CD detailed guide
├── TASK4.md                  # Terraform + Kubernetes detailed guide
├── backend/
│   ├── package.json          # Node.js backend dependencies
│   └── server.js             # Express.js API server
├── frontend/
│   ├── package.json          # React app dependencies
│   ├── vite.config.js        # Vite build config
│   ├── index.html            # HTML entry point
│   └── src/
│       ├── main.jsx          # React entry
│       └── App.jsx           # React component
├── .github/
│   └── workflows/
│       └── ci-cd.yml         # GitHub Actions pipeline
├── terraform/
│   └── main.tf               # AWS VPC infrastructure
└── k8s/                      # Kubernetes manifests
    ├── config.yaml           # ConfigMap + Secret
    ├── pvc.yaml              # PersistentVolumeClaim
    ├── deployment.yaml       # Deployment (2 replicas)
    ├── service.yaml          # ClusterIP Service
    ├── ingress.yaml          # NGINX Ingress
    └── hpa.yaml              # Horizontal Pod Autoscaler
```

---

## Estimated Total Time

| Task | Setup | Execution | Total |
|------|-------|-----------|-------|
| Task 2 (Docker) | 5 min | 25 min | **30 min** |
| Task 3 (CI/CD) | 5 min | 15 min | **20 min** |
| Task 4A (Terraform) | 10 min | 10 min | **20 min** |
| Task 4B (Kubernetes) | 15 min | 30 min | **45 min** |
| **Total** | **35 min** | **80 min** | **~2.5 hours** |

---

## Common Errors & Fixes

### Docker
| Error | Fix |
|-------|-----|
| "Cannot connect to Docker daemon" | Start Docker Desktop |
| "Port 5000 already in use" | `docker compose down` or change port |
| "npm run build failed" | Check frontend/package.json; ensure Vite installed |

### Git & GitHub
| Error | Fix |
|-------|-----|
| "fatal: not a git repository" | Run `git init` in project root |
| "fatal: Could not read from remote repository" | Check Git credentials; run `git remote -v` |
| Workflow not showing | Wait 30s; refresh Actions tab; check `.github/workflows/ci-cd.yml` on main branch |

### Terraform
| Error | Fix |
|-------|-----|
| "AWS credentials not configured" | Run `aws configure` |
| "No provider found" | Run `terraform init` |
| "Apply failed" | Check terraform.tfstate permissions |

### Kubernetes
| Error | Fix |
|-------|-----|
| "Cannot connect to cluster" | Run `minikube start --driver=docker` |
| "myapp:latest image not found" | Run `eval $(minikube docker-env)` then `docker build -t myapp:latest .` |
| "Pods stuck in Pending" | Run `kubectl describe pod <name>`; usually image/storage issue |
| "HPA not scaling" | Run `minikube addons enable metrics-server` |

---

## Next Steps After Completion

1. **Prepare final report** with screenshots from all tasks
2. **Document lessons learned** (challenges, solutions, insights)
3. **Link to GitHub repo** in submission
4. **Prepare demo** for Progree (optional but impressive):
   - Show Docker build output
   - Show GitHub Actions pipeline
   - Show Kubernetes pods scaling under load

---

## Support Resources

- **Docker**: https://docs.docker.com/
- **GitHub Actions**: https://docs.github.com/en/actions
- **Terraform**: https://www.terraform.io/docs
- **Kubernetes**: https://kubernetes.io/docs/
- **Minikube**: https://minikube.sigs.k8s.io/docs/
- **AWS**: https://docs.aws.amazon.com/

---

Good luck with your Progree DevOps internship! 🚀
