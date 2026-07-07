# TASK 2: Application Containerization & Asset Optimization

## Summary
Build a production-ready Docker image with:
- ✅ Multi-stage Dockerfile (3 stages)
- ✅ Minimal image size (~150MB vs 1GB+ full Node)
- ✅ Secure environment variable handling
- ✅ Correct port mapping (5000:5000)
- ✅ Non-root user for security
- ✅ Health checks for container readiness

---

## Step-by-Step Execution

### Step 1: Prepare Environment File
```bash
cd C:\Users\usman\progree-devops-project
cp .env.example .env
```

Expected `.env` contents:
```
DB_HOST=db
DB_PORT=5432
DB_USER=appuser
DB_PASSWORD=changeme
DB_NAME=appdb
JWT_SECRET=CODM
PORT=5000
```

### Step 2: Build the Docker Image
```bash
docker build -t myapp:latest .
```

**Expected output:**
```
[+] Building 45.2s (9/9) FINISHED
=> [runtime] copy ...
=> [runtime] RUN addgroup ...
...
Successfully tagged myapp:latest
```

### Step 3: Verify Image Size
```bash
docker images myapp
```

**Expected output:**
```
REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
myapp         latest    abc1234def     5 mins ago    156MB
```

**Why this matters for your report:**
- Alpine base (~40MB) + runtime dependencies = ~150MB
- **Without Alpine** (full Node:20 image) = **1GB+**
- **Savings: ~85% image size reduction**

### Step 4: Inspect Multi-Stage Build
```bash
docker history myapp:latest
```

Shows:
- Stage 1: Frontend build (frontend-build)
- Stage 2: Backend build (backend-build)
- Stage 3: Runtime only (final image)

**Only Stage 3 is kept in final image** — no build tools, no dev dependencies.

### Step 5: Run with docker-compose (includes PostgreSQL DB)
```bash
docker compose up -d
```

**Expected output:**
```
[+] Running 2/2
 ✔ db       Started    0.2s
 ✔ backend  Started    0.3s
```

### Step 6: Verify the App is Running
```bash
# Check container status
docker compose ps

# Expected:
# NAME      COMMAND               SERVICE   STATUS      PORTS
# db        docker-entrypoint...  db        Up 5 secs   5432/tcp
# backend   node backend/server   backend   Up 5 secs   5000/tcp
```

### Step 7: Health Check
```bash
curl http://localhost:5000/health
```

**Expected response:**
```json
{"status":"ok"}
```

### Step 8: View Logs
```bash
docker compose logs -f backend

# Expected:
# backend  | Server listening on port 5000
```

### Step 9: Verify Port Mapping
```bash
# Check which ports are mapped
docker compose port backend 5000

# Expected: 0.0.0.0:5000
```

### Step 10: Verify Non-Root User
```bash
docker compose exec backend whoami

# Expected: appuser (NOT root)
```

### Step 11: Test Environment Variables
```bash
docker compose exec backend sh -c 'echo $NODE_ENV'

# Expected: production
```

### Step 12: Cleanup
```bash
# Stop containers
docker compose down

# Remove volumes too (clean database)
docker compose down -v

# Verify cleanup
docker compose ps
# Expected: no services listed
```

---

## Deliverables for Your Report

### 1. Screenshot: Image Size
```bash
docker images myapp
```
Capture this showing:
- IMAGE: myapp
- TAG: latest
- SIZE: ~156MB (or your actual size)

### 2. Screenshot: Build Logs
```bash
docker build -t myapp:latest . 2>&1 | head -20
```
Show successful multi-stage build completion.

### 3. Screenshot: Health Check
```bash
curl -v http://localhost:5000/health
```
Show `{"status":"ok"}` response with HTTP 200.

### 4. Screenshot: Running Services
```bash
docker compose ps
```
Show backend and db both running.

### 5. Document: Port Routing Table
| Service  | Container Port | Host Port | Purpose              |
|----------|----------------|-----------|----------------------|
| backend  | 5000           | 5000      | API + serves frontend |
| db       | 5432           | 5432      | PostgreSQL database  |

### 6. Files in Repository
- ✅ [Dockerfile](Dockerfile)
- ✅ [.dockerignore](.dockerignore)
- ✅ [docker-compose.yml](docker-compose.yml)
- ✅ [.env.example](.env.example)
- ✅ [backend/package.json](backend/package.json)
- ✅ [backend/server.js](backend/server.js)
- ✅ [frontend/package.json](frontend/package.json)

---

## Key Points Explained

### Multi-Stage Build
Each `FROM` statement creates a new stage:
1. **frontend-build**: Installs deps, runs `npm run build` → compiled output only
2. **backend-build**: Installs backend deps
3. **runtime**: Copies only compiled code + backend to final image

**Result**: Final image has NO npm, NO build cache, NO dev dependencies → smaller & faster startup.

### Security
- **Non-root user**: `appuser` instead of `root`
  - Reduces container breakout risk
  - Any compromised app cannot modify system files
  
- **No secrets in Dockerfile**
  - Use `.env` file + docker-compose (gitignored)
  - Or use CI/CD secret injection (GitHub Secrets, etc.)

- **Alpine base**
  - Only essential Linux utilities (~5MB base)
  - vs Debian-based (~100MB base)

### Environment Variables
**Do NOT hardcode in Dockerfile:**
```dockerfile
# ❌ WRONG
ENV DB_PASSWORD=changeme
```

**Instead, inject at runtime:**
```dockerfile
# ✅ CORRECT
ENV PORT=5000  # Only non-secret defaults
```

Then provide secrets via:
- `.env` file (local development)
- `env_file:` in docker-compose.yml
- `--env-file` flag at runtime
- CI/CD secrets store (GitHub Secrets, GitLab Secrets, etc.)

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Cannot connect to Docker daemon" | Start Docker Desktop |
| "Port 5000 already in use" | `docker compose down` or change port in docker-compose.yml |
| "npm: command not found" | Image build failed; check Dockerfile syntax |
| "db: connection refused" | Wait 10s for PostgreSQL to start; use `docker compose logs db` |
| "Health check fails" | `docker compose logs backend` to see error |

---

## Estimated Time
- **Setup**: 5 minutes
- **First build**: 10–15 minutes (downloads base images)
- **Subsequent builds**: 2–5 minutes (caching)
- **Testing**: 5 minutes
- **Total**: ~30 minutes for full Task 2

---

## What's Next
After Task 2 verification:
→ Move to [Task 3: CI/CD Pipeline](../task3/TASK3.md)
