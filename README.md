# Progree DevOps Internship — Task Log

**Intern:** Muhammad Usman Ameer (Student ID: B1/019)
**Program:** Remote DevOps Internship — Progree
**Duration:** 5th June 2026 – 4th July 2026
**Repository:** [Usman61-undercover/Progree-DevOps-Internship](https://github.com/Usman61-undercover/Progree-DevOps-Internship)

This README consolidates the progress log for all internship tasks. Each task also has its own standalone README (linked below) with full step-by-step execution detail.

## Task Status Overview

| Task | Title | Status |
|------|-------|--------|
| 1 | Professional LinkedIn Announcement | ✅ Completed |
| 2 | Application Containerization & Asset Optimization | ✅ Completed |
| 3 | Multi-Stage Automated CI/CD Deployment Pipeline | ✅ Completed |
| 4 | Infrastructure as Code & Kubernetes Orchestration | ⏳ Not yet started |

---

## TASK 1: Professional LinkedIn Announcement
**Status:** ✅ Completed | [Full details](README_TASK1.md)

Published a LinkedIn post announcing acceptance into the Progree DevOps internship, including the official offer letter, a caption covering internship learning goals, an @Progree tag, and the required hashtags (`#devops #docker #cicd #progree #kubernetes #terraform #internship`).

**Evidence:** Screenshot of the published LinkedIn post with offer letter, caption, tag, and hashtags visible.

---

## TASK 2: Application Containerization & Asset Optimization
**Status:** ✅ Completed | [Full details](README_TASK2.md)

Built a 3-stage Dockerfile (frontend-build → backend-build → runtime) producing a minimal Alpine-based image, orchestrated via `docker-compose` with a `backend` and `db` (PostgreSQL) service.

**Key results:**
- Both containers verified running in Docker Desktop
- App verified live at `http://localhost:5000`
- Health check returned `200 OK`
- `Test-NetConnection` confirmed TCP connectivity to PostgreSQL on port 5432 (`TcpTestSucceeded: True`)
- Non-root user (`appuser`) and secure `.env`-based secret handling confirmed

| Service | Container Port | Host Port | Purpose |
|---------|-----------------|-----------|---------|
| backend | 5000 | 5000 | API + serves frontend |
| db | 5432 | 5432 | PostgreSQL database |

**Evidence:** Docker Desktop containers view, compose logs, browser screenshot, PowerShell connectivity + health check screenshots.

---

## TASK 3: Multi-Stage Automated CI/CD Deployment Pipeline
**Status:** ✅ Completed | [Full details](README_TASK3.md)

Configured a GitHub Actions workflow (`.github/workflows/ci-cd.yml`) in the `Progree-DevOps-Internship` repository that triggers automatically on every push to `main`.

**Pipeline result (run "first commit #1", commit `3bcaf65`):**

| Stage | Duration | Result |
|-------|----------|--------|
| Lint Code | 10s | ✅ Passed |
| Run Unit Tests | 11s | ✅ Passed |
| Build Docker Image | 24s | ✅ Passed |
| Report Deployment Status | 4s | ✅ Passed |

**Total run duration:** 59 seconds — Status: **Success**

**Evidence:** Actions tab workflow list, filtered workflow dashboard, full pipeline visualization with all four stages passed.

---

## TASK 4: Infrastructure as Code & Kubernetes Orchestration
**Status:** ⏳ Not yet started

Planned scope: provision an isolated network with Terraform (AWS VPC or GCP equivalent), deploy the containerized application to a Kubernetes cluster (EKS/GKE/Minikube) with persistent volume claims, an Ingress controller, and a Horizontal Pod Autoscaler (HPA).
