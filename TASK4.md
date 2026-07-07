# TASK 4: Infrastructure as Code + Kubernetes Orchestration

## Summary
Deploy production-grade infrastructure:
- ✅ **Part A**: Terraform VPC on AWS (isolated network, multi-AZ)
- ✅ **Part B**: Kubernetes deployment (2 replicas, PVC, Ingress, HPA)
- ✅ High availability, auto-scaling, persistent storage

---

## PART A: Terraform — AWS VPC Provisioning

### Prerequisites
- AWS account (free tier eligible; costs ~$0 if using minimal resources)
- AWS CLI configured with credentials
- Terraform installed

### Step 1: Install Terraform

**macOS (Homebrew):**
```bash
brew install terraform
terraform version
# Expected: Terraform v1.5.x or later
```

**Windows (Chocolatey):**
```bash
choco install terraform
terraform --version
```

**Or download directly:**
Visit https://www.terraform.io/downloads and add to PATH.

### Step 2: Install AWS CLI

**macOS:**
```bash
brew install awscli
aws --version
```

**Windows (Chocolatey):**
```bash
choco install awscli
aws --version
```

### Step 3: Configure AWS Credentials
```bash
aws configure
```

**Prompts:**
```
AWS Access Key ID [None]: YOUR_ACCESS_KEY
AWS Secret Access Key [None]: YOUR_SECRET_KEY
Default region name [None]: us-east-1
Default output format [None]: json
```

**To get keys:**
1. Go to AWS Console → IAM → Users → Your user
2. Security credentials tab → Create access key
3. Copy Access Key ID and Secret Access Key

### Step 4: Initialize Terraform
```bash
cd C:\Users\usman\progree-devops-project\terraform
terraform init
```

**Expected output:**
```
Initializing the backend...
Initializing provider plugins...
- Reusing previous version of hashicorp/aws from the lock file
- Using previously-installed hashicorp/aws v5.x.x
...
Terraform has been successfully configured!
```

### Step 5: Review the Terraform Plan
```bash
terraform plan
```

**Expected output shows:**
```
Plan: 6 to add, 0 to change, 0 to destroy.

+ aws_vpc.main                                # VPC (10.0.0.0/16)
+ aws_subnet.public[0]                        # Subnet 1 (10.0.0.0/24)
+ aws_subnet.public[1]                        # Subnet 2 (10.0.1.0/24)
+ aws_internet_gateway.gw                     # IGW for public access
+ aws_route_table.public                      # Routing config
+ aws_route_table_association.public[0]       # Route subnet 1
+ aws_route_table_association.public[1]       # Route subnet 2
```

**Never apply without reviewing the plan first!**

### Step 6: Apply Terraform (Create AWS Resources)

**⚠️ WARNING: This creates real AWS resources. AWS charges may apply (usually $0–2 per hour if left running).**

```bash
terraform apply -auto-approve
```

**Expected output:**
```
Apply complete! Resources: 6 added, 0 changed, 0 destroyed.

Outputs:
vpc_id = "vpc-abc1234def567890"
```

### Step 7: Verify Infrastructure in AWS Console
1. Go to AWS Console → VPC → VPCs
2. Find `progree-vpc` (10.0.0.0/16)
3. Check subnets: `progree-public-0` and `progree-public-1`
4. Check Internet Gateway: `progree-igw`

### Step 8: Get Infrastructure Details
```bash
terraform show
# Shows all created resources in detail

terraform output vpc_id
# Shows just the VPC ID
```

### Step 9: Destroy Infrastructure (Clean Up)
```bash
terraform destroy -auto-approve
```

**Expected output:**
```
Destroy complete! Resources: 6 destroyed.
```

**Why destroy?**
- Stops AWS billing
- Cleans up unused resources
- Safe to re-create anytime with `terraform apply`

---

## PART B: Kubernetes Orchestration — Minikube

### Prerequisites
- Docker installed and running
- Minikube installed
- kubectl installed (usually comes with Minikube)

### Step 1: Install Minikube

**macOS (Homebrew):**
```bash
brew install minikube
minikube version
# Expected: minikube version: v1.32.x
```

**Windows (Chocolatey):**
```bash
choco install minikube
minikube version
```

**Or download from:** https://minikube.sigs.k8s.io/docs/start/

### Step 2: Install kubectl

**macOS:**
```bash
brew install kubectl
kubectl version --client
```

**Windows:**
```bash
choco install kubernetes-cli
kubectl version --client
```

### Step 3: Start Minikube
```bash
minikube start --driver=docker
```

**Expected output:**
```
😄  minikube v1.32.x on Darwin
✨  Automatically selected the docker driver
👍  Starting control plane node minikube in cluster minikube
...
🎉  minikube successfully started!
```

**What this does:**
- Creates a single-node Kubernetes cluster in Docker
- Downloads Kubernetes (~1.5GB)
- Starts the cluster (takes 1–2 minutes)

### Step 4: Enable Required Addons

**Ingress controller (for external access):**
```bash
minikube addons enable ingress
```

**Metrics server (for HPA CPU monitoring):**
```bash
minikube addons enable metrics-server
```

**Verify addons:**
```bash
minikube addons list
# Expected: ingress ENABLED, metrics-server ENABLED
```

### Step 5: Build Docker Image into Minikube

**Set Docker environment to Minikube's Docker:**
```bash
# macOS/Linux
eval $(minikube docker-env)

# Windows PowerShell
minikube docker-env | Invoke-Expression
```

**Build the image:**
```bash
docker build -t myapp:latest .
```

**Why this step?**
- Minikube has its own Docker daemon
- Building locally doesn't make image available to Minikube
- By setting the environment, builds go directly into Minikube

### Step 6: Deploy Kubernetes Manifests

**Apply configuration & secrets:**
```bash
kubectl apply -f k8s/config.yaml
# Expected: configmap/myapp-config created, secret/myapp-secrets created
```

**Apply PersistentVolumeClaim:**
```bash
kubectl apply -f k8s/pvc.yaml
# Expected: persistentvolumeclaim/myapp-pvc created
```

**Apply Deployment:**
```bash
kubectl apply -f k8s/deployment.yaml
# Expected: deployment.apps/myapp-deployment created
```

**Apply Service:**
```bash
kubectl apply -f k8s/service.yaml
# Expected: service/myapp-service created
```

**Apply Ingress:**
```bash
kubectl apply -f k8s/ingress.yaml
# Expected: ingress.networking.k8s.io/myapp-ingress created
```

**Apply HPA:**
```bash
kubectl apply -f k8s/hpa.yaml
# Expected: horizontalpodautoscaler.autoscaling/myapp-hpa created
```

### Step 7: Verify All Components

**Check pods are running:**
```bash
kubectl get pods
```

**Expected output:**
```
NAME                               READY   STATUS    RESTARTS   AGE
myapp-deployment-abc1234-def56     1/1     Running   0          10s
myapp-deployment-abc1234-ghi78     1/1     Running   0          10s
```

**Check services:**
```bash
kubectl get svc
```

**Expected output:**
```
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP
myapp-service   ClusterIP   10.98.123.45    <none>        80/TCP
```

**Check PersistentVolumeClaim:**
```bash
kubectl get pvc
```

**Expected output:**
```
NAME        STATUS   VOLUME                                     CAPACITY
myapp-pvc   Bound    pvc-abc1234-def5-6789-ghij-klmn0pqrst     1Gi
```

**Check HPA:**
```bash
kubectl get hpa
```

**Expected output:**
```
NAME        REFERENCE                     TARGETS   MINPODS   MAXPODS
myapp-hpa   Deployment/myapp-deployment   0%/70%    2         6
```

### Step 8: Access the Application

**Get the service URL:**
```bash
minikube service myapp-service --url
# Expected: http://192.168.49.2:30000 (or similar)
```

**Or open in browser:**
```bash
minikube service myapp-service
# Automatically opens browser
```

**Or use port-forward for direct access:**
```bash
kubectl port-forward svc/myapp-service 8080:80
# Opens on http://localhost:8080
```

### Step 9: Test Auto-Scaling (HPA)

**Generate load:**
```bash
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://myapp-service; done"
```

**In another terminal, watch pods scale:**
```bash
kubectl get hpa -w
# Watch until "REPLICAS" column increases (e.g., 2 → 4 → 6)
```

**Expected behavior:**
- As CPU usage rises above 70%, HPA scales up
- Eventually 2–6 replicas running
- Press Ctrl+C to stop watching

### Step 10: Check Pod Logs

**View backend logs:**
```bash
kubectl logs -f deployment/myapp-deployment
# Shows "Server listening on port 5000" for each pod
```

### Step 11: Describe HPA for Detailed Status

```bash
kubectl describe hpa myapp-hpa
```

**Expected output:**
```
Name:                                                myapp-hpa
Namespace:                                          default
...
Metrics:                                            ( current / target )
  resource cpu on pods  (as a percentage of request):  15% / 70%
Min replicas:                                       2
Max replicas:                                       6
Behavior:
  Scale Up:
    - type: Percent
      value: 100%
...
Events:
  Type    Reason             Age   From                       Message
  ----    ------             ---   ----                       -------
  Normal  SuccessfulRescale  45s   horizontal-pod-autoscaler  New size: 2; reason: All metrics below target
```

---

## Deliverables for Your Report

### Part A: Terraform

**1. Screenshot: terraform plan output**
```bash
terraform plan
```
Show: 6 resources to add (VPC, subnets, IGW, routes, associations)

**2. Screenshot: terraform apply output**
```bash
terraform apply -auto-approve
```
Show: VPC ID and "Apply complete" message

**3. Screenshot: AWS Console**
Navigate to VPC → VPCs and show:
- `progree-vpc` (10.0.0.0/16)
- `progree-public-0`, `progree-public-1` subnets
- `progree-igw` Internet Gateway

**4. Document: Terraform File**
Include `terraform/main.tf` showing:
- VPC (10.0.0.0/16)
- 2 subnets across AZs
- Internet Gateway
- Route table

### Part B: Kubernetes

**1. Screenshot: kubectl get pods**
```bash
kubectl get pods
```
Show: 2 myapp-deployment replicas, STATUS Running

**2. Screenshot: kubectl get svc**
```bash
kubectl get svc
```
Show: myapp-service ClusterIP, Port 80 → 5000

**3. Screenshot: kubectl get pvc**
```bash
kubectl get pvc
```
Show: myapp-pvc Bound, 1Gi capacity

**4. Screenshot: kubectl get hpa**
```bash
kubectl get hpa
```
Show: myapp-hpa, minReplicas 2, maxReplicas 6, TARGETS 0%/70%

**5. Screenshot: kubectl describe hpa myapp-hpa**
Show: Full HPA configuration, CPU metrics, events

**6. Screenshot: kubectl logs**
```bash
kubectl logs -f deployment/myapp-deployment | head -10
```
Show: "Server listening on port 5000" logs

**7. Evidence of Auto-Scaling**
- Run load generator
- Show HPA scaling pods from 2 → 4–6
- Show increased CPU usage

**8. Document: Kubernetes Manifests**
Include all files from `k8s/`:
- config.yaml (ConfigMap + Secret)
- pvc.yaml (PersistentVolumeClaim)
- deployment.yaml (Deployment, 2 replicas)
- service.yaml (ClusterIP Service)
- ingress.yaml (NGINX Ingress)
- hpa.yaml (Horizontal Pod Autoscaler)

---

## Key Points Explained

### Terraform: Infrastructure as Code
**Benefits:**
- Version control for infrastructure (`.tf` files in Git)
- Reproducible: `terraform apply` always creates same resources
- Declarative: You describe the end state; Terraform handles the how
- Reversible: `terraform destroy` removes everything

**State file:**
- `terraform.tfstate` tracks current AWS resources
- **Keep secret** (contains sensitive data)
- Add to `.gitignore` in production

### Kubernetes: Container Orchestration

**Deployment (2 replicas):**
- Manages pods (containerized apps)
- Restarts failed pods automatically
- Handles rolling updates (0 downtime)

**Service (ClusterIP):**
- Internal DNS name: `myapp-service.default.svc.cluster.local`
- Routes traffic to pods
- Survives pod restarts (constant IP)

**Ingress (NGINX):**
- External HTTP/HTTPS entry point
- Maps hostname (myapp.local) → service
- Handles SSL termination, routing rules

**PersistentVolumeClaim (1Gi):**
- Persistent storage (survives pod deletion)
- Database data, user uploads, logs, etc.
- Auto-provisioned on Minikube

**HPA (Horizontal Pod Autoscaler):**
- Monitors CPU usage of pods
- If avg CPU > 70%: scales up (add pods)
- If avg CPU < 70%: scales down (remove pods)
- Min 2, Max 6 replicas

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Cannot connect to Docker daemon" | Start Docker Desktop |
| "minikube: command not found" | Install Minikube; add to PATH |
| "kubectl: command not found" | Install kubectl; usually comes with Minikube |
| Pods stuck in Pending | `kubectl describe pod <name>` — usually image pull issue or resource constraints |
| "myapp:latest image not found" | Run `eval $(minikube docker-env)` then `docker build -t myapp:latest .` |
| HPA not scaling | Ensure metrics-server addon is enabled: `minikube addons enable metrics-server` |
| Ingress not accessible | Use `minikube service myapp-service --url` instead; Ingress in Minikube has limited DNS support |
| PVC not binding | Check storage class: `kubectl get storageclass` — Minikube has default provisioner |
| "Terraform apply failed" | Check AWS credentials: `aws sts get-caller-identity` |

---

## Estimated Time
- **Terraform setup**: 10 minutes (init + credentials)
- **Terraform apply**: 2–5 minutes (creates VPC + subnets)
- **Minikube setup**: 5–10 minutes (first start takes longer)
- **Kubernetes deployment**: 5 minutes (apply all manifests)
- **Verification**: 5 minutes
- **Auto-scaling demo**: 5 minutes
- **Total**: ~40–60 minutes

---

## Cleanup

**Destroy Kubernetes deployment:**
```bash
kubectl delete -f k8s/
```

**Stop Minikube (keeps cluster):**
```bash
minikube stop
```

**Delete Minikube (full cleanup):**
```bash
minikube delete
```

**Destroy AWS infrastructure:**
```bash
cd terraform
terraform destroy -auto-approve
```

---

## What's Next
After Task 4 verification:
→ All DevOps tasks complete!
→ Prepare final report with screenshots from all tasks
→ Submit to Progree with evidence artifacts
