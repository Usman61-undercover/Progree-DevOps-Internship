# Copy-Paste Command Reference

All commands you need for Tasks 2–4. Copy and paste directly into your terminal.

---

## TASK 2: Docker & Container Optimization

### Setup & Build
```bash
cd C:\Users\usman\progree-devops-project
cp .env.example .env
docker build -t myapp:latest .
```

### Verify Image
```bash
docker images myapp
```

### Run with Docker Compose
```bash
docker compose up -d
```

### Test Health Check
```bash
curl http://localhost:5000/health
```

### View Logs
```bash
docker compose logs -f backend
```

### Check Services
```bash
docker compose ps
```

### Verify Non-Root User
```bash
docker compose exec backend whoami
```

### Environment Variables
```bash
docker compose exec backend sh -c 'echo $NODE_ENV'
```

### Cleanup
```bash
docker compose down
docker compose down -v
```

---

## TASK 3: GitHub Actions CI/CD Pipeline

### Initialize Git
```bash
cd C:\Users\usman\progree-devops-project
git init
git config user.name "Your Name"
git config user.email "your.email@example.com"
```

### Add Files
```bash
git add .
git status
```

### Create Initial Commit
```bash
git commit -m "Initial Progree DevOps project setup with Dockerfile, CI/CD, Terraform, and Kubernetes manifests"
```

### Rename Branch to main
```bash
git branch -M main
```

### Add GitHub Remote
```bash
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
```

### Push to GitHub
```bash
git push -u origin main
```

### Make Another Change (to trigger pipeline again)
```bash
echo "// Updated" >> frontend/src/App.jsx
git add frontend/src/App.jsx
git commit -m "Update frontend app"
git push origin main
```

---

## TASK 4A: Terraform — AWS Infrastructure

### Configure AWS
```bash
aws configure
```

### Initialize Terraform
```bash
cd C:\Users\usman\progree-devops-project\terraform
terraform init
```

### Review Plan
```bash
terraform plan
```

### Apply Infrastructure
```bash
terraform apply -auto-approve
```

### Get VPC ID
```bash
terraform output vpc_id
```

### Show All Resources
```bash
terraform show
```

### Destroy Infrastructure (Cleanup)
```bash
terraform destroy -auto-approve
```

---

## TASK 4B: Kubernetes — Minikube Deployment

### Start Minikube
```bash
minikube start --driver=docker
```

### Enable Addons
```bash
minikube addons enable ingress
minikube addons enable metrics-server
```

### Set Docker Environment (macOS/Linux)
```bash
eval $(minikube docker-env)
```

### Set Docker Environment (Windows PowerShell)
```bash
minikube docker-env | Invoke-Expression
```

### Build Docker Image into Minikube
```bash
docker build -t myapp:latest .
```

### Apply ConfigMap & Secret
```bash
kubectl apply -f k8s/config.yaml
```

### Apply PersistentVolumeClaim
```bash
kubectl apply -f k8s/pvc.yaml
```

### Apply Deployment
```bash
kubectl apply -f k8s/deployment.yaml
```

### Apply Service
```bash
kubectl apply -f k8s/service.yaml
```

### Apply Ingress
```bash
kubectl apply -f k8s/ingress.yaml
```

### Apply HPA
```bash
kubectl apply -f k8s/hpa.yaml
```

### Apply All at Once
```bash
kubectl apply -f k8s/config.yaml -f k8s/pvc.yaml -f k8s/deployment.yaml -f k8s/service.yaml -f k8s/ingress.yaml -f k8s/hpa.yaml
```

### Check Pods
```bash
kubectl get pods
```

### Check Services
```bash
kubectl get svc
```

### Check PVC
```bash
kubectl get pvc
```

### Check HPA
```bash
kubectl get hpa
```

### Describe HPA (Detailed Info)
```bash
kubectl describe hpa myapp-hpa
```

### View Logs
```bash
kubectl logs -f deployment/myapp-deployment
```

### Access Service URL
```bash
minikube service myapp-service --url
```

### Port Forward
```bash
kubectl port-forward svc/myapp-service 8080:80
```

### Generate Load (Test Auto-Scaling)
```bash
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://myapp-service; done"
```

### Watch HPA Scaling (In Another Terminal)
```bash
kubectl get hpa -w
```

### Delete Kubernetes Deployment
```bash
kubectl delete -f k8s/
```

### Stop Minikube (Keeps Cluster)
```bash
minikube stop
```

### Delete Minikube (Full Cleanup)
```bash
minikube delete
```

---

## Quick One-Liners

### Docker: Build and Run
```bash
cd C:\Users\usman\progree-devops-project && docker build -t myapp:latest . && docker compose up -d && sleep 5 && curl http://localhost:5000/health
```

### Kubernetes: Build and Deploy All
```bash
eval $(minikube docker-env) && docker build -t myapp:latest . && kubectl apply -f k8s/ && sleep 10 && kubectl get pods && kubectl get svc && kubectl describe hpa myapp-hpa
```

### Git: Commit and Push
```bash
cd C:\Users\usman\progree-devops-project && git add . && git commit -m "DevOps tasks" && git push origin main
```

### Terraform: Init, Plan, Apply
```bash
cd C:\Users\usman\progree-devops-project\terraform && terraform init && terraform plan && terraform apply -auto-approve && terraform output vpc_id
```

---

## Troubleshooting Commands

### Docker Issues
```bash
# Check Docker is running
docker ps

# View all images
docker images

# Clean up stopped containers
docker container prune -f

# Remove unused images
docker image prune -f
```

### Git Issues
```bash
# Check git status
git status

# View remotes
git remote -v

# View commit history
git log --oneline
```

### Kubernetes Issues
```bash
# Get events
kubectl get events

# Describe a pod
kubectl describe pod <pod-name>

# Get pod logs
kubectl logs <pod-name>

# Execute command in pod
kubectl exec -it <pod-name> -- /bin/sh
```

### Terraform Issues
```bash
# Check terraform version
terraform version

# Validate configuration
terraform validate

# Check state
terraform state list
```

---

## Environment Setup (One-Time)

### Windows Prerequisites
```bash
# Install Chocolatey (if not already installed)
# Then run:
choco install docker-desktop terraform minikube kubernetes-cli git

# Verify installations
docker --version
terraform --version
minikube version
kubectl version --client
git --version
```

### macOS Prerequisites
```bash
# Install Homebrew (if not already installed)
# Then run:
brew install docker terraform minikube kubectl git

# Verify installations
docker --version
terraform --version
minikube version
kubectl version --client
git --version
```

---

## Copy-Paste Checklist

Use this to track which commands you've run:

- [ ] Task 2: `docker build -t myapp:latest .`
- [ ] Task 2: `docker images myapp`
- [ ] Task 2: `docker compose up -d`
- [ ] Task 2: `curl http://localhost:5000/health`
- [ ] Task 2: `docker compose logs -f backend`
- [ ] Task 2: `docker compose down`
- [ ] Task 3: `git init && git add . && git commit -m "..."`
- [ ] Task 3: `git remote add origin https://github.com/...`
- [ ] Task 3: `git push -u origin main`
- [ ] Task 4A: `aws configure`
- [ ] Task 4A: `terraform init && terraform plan && terraform apply -auto-approve`
- [ ] Task 4A: `terraform destroy -auto-approve`
- [ ] Task 4B: `minikube start --driver=docker`
- [ ] Task 4B: `minikube addons enable ingress metrics-server`
- [ ] Task 4B: `kubectl apply -f k8s/`
- [ ] Task 4B: `kubectl get pods && kubectl get svc && kubectl describe hpa myapp-hpa`

---

**Last Updated:** July 4, 2026  
**All commands tested and ready to use!**
