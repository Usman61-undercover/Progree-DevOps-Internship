# TASK 3: Multi-Stage CI/CD Pipeline (GitHub Actions)

## Summary
Automate code quality checks and container builds on every push:
- ✅ Lint stage (ESLint / code style)
- ✅ Test stage (Unit tests + coverage)
- ✅ Build stage (Docker image with commit SHA tag)
- ✅ Notify stage (Pipeline status summary)
- ✅ Artifacts (Coverage reports)

---

## Step-by-Step Execution

### Step 1: Initialize Git Repository (if not done)
```bash
cd C:\Users\usman\progree-devops-project
git init
git config user.name "Your Name"
git config user.email "your.email@example.com"
```

### Step 2: Add All Files to Git
```bash
git add .
git status
# Expected: Shows Dockerfile, docker-compose.yml, .github/workflows/ci-cd.yml, k8s/, terraform/, etc.
```

### Step 3: Create Initial Commit
```bash
git commit -m "Initial Progree DevOps project setup with Dockerfile, CI/CD, Terraform, and Kubernetes manifests"
```

### Step 4: Rename Branch to main (GitHub default)
```bash
git branch -M main
```

### Step 5: Add GitHub Remote
```bash
git remote add origin https://github.com/<YOUR_USERNAME>/<YOUR_REPO_NAME>.git
```

**Replace:**
- `<YOUR_USERNAME>`: Your GitHub username
- `<YOUR_REPO_NAME>`: Your repo name (e.g., `progree-devops-project`)

### Step 6: Push to GitHub
```bash
git push -u origin main
```

**Expected output:**
```
Enumerating objects: 45, done.
Counting objects: 100% (45/45), done.
...
To https://github.com/<YOUR_USERNAME>/<YOUR_REPO_NAME>.git
 * [new branch]      main -> main
```

### Step 7: Check GitHub Actions Tab
1. Go to: `https://github.com/<YOUR_USERNAME>/<YOUR_REPO_NAME>`
2. Click the **Actions** tab
3. You should see a workflow run in progress or completed

**Expected workflow stages:**
- ✅ lint (Pass)
- ✅ test (Pass)
- ✅ build (Pass)
- ✅ notify (Pass with summary)

### Step 8: Wait for All Stages to Complete
- **Lint**: Runs first (5–10 seconds)
- **Test**: Runs after lint (5–10 seconds)
- **Build**: Runs after test (10–20 seconds)
- **Notify**: Runs last, always runs (1–2 seconds)

**Expected total time**: ~30–50 seconds

### Step 9: Click on the Workflow Run
Click on the commit message to see detailed logs for each stage.

**Expected log output:**
```
Checkout code ✓
Setup Node.js ✓
Install dependencies ✓
Run lint ✓
...
Summarize pipeline result ✓
```

### Step 10: View Pipeline Summary
At the bottom of the workflow run, you'll see:
```
## Pipeline Summary
- Lint: success
- Test: success
- Build: success
```

### Step 11: Screenshot the Passing Pipeline
Capture a screenshot showing:
- Workflow name: "CI/CD Pipeline"
- All stages (lint, test, build, notify) with green checkmarks ✓
- Run duration (e.g., "Completed in 45 seconds")

### Step 12: Check Coverage Artifact (Optional)
If tests generate coverage:
1. Scroll down to "Artifacts"
2. You'll see `coverage-report` (if any)
3. Download to inspect

---

## Make a Change and Trigger Pipeline Again

### Step 1: Edit a File
```bash
# For example, edit frontend/src/App.jsx
echo "// Updated" >> frontend/src/App.jsx
```

### Step 2: Commit and Push
```bash
git add frontend/src/App.jsx
git commit -m "Update frontend app"
git push origin main
```

### Step 3: Check Actions Tab Again
The workflow should trigger automatically:
- Lint → Test → Build → Notify

This demonstrates that the CI/CD pipeline runs on **every push**.

---

## Pipeline Stage Details

### Stage 1: Lint
```yaml
- name: Run ESLint
  run: npm run lint
```

Currently a placeholder. **For production**, replace with:
```bash
npm install --save-dev eslint
npm run lint  # Uses .eslintrc.json config
```

**What it checks:**
- Code style (indentation, semicolons, etc.)
- Potential bugs (unused variables, undefined functions)
- Best practices

**Fails if**: Linting errors found → stops pipeline (test/build won't run)

### Stage 2: Test
```yaml
- name: Run tests with coverage
  run: npm test -- --coverage
```

Currently a placeholder. **For production**, replace with:
```bash
npm install --save-dev jest
npm test  # Runs all *.test.js files
```

**Uploads coverage artifact** (code coverage percentage).

**Fails if**: Tests fail → stops pipeline (build won't run)

### Stage 3: Build
```yaml
- name: Build Docker image
  run: docker build -t myapp:${{ github.sha }} .
```

Tags the Docker image with the commit SHA (e.g., `myapp:abc123def456`).

**Why commit SHA?**
- Each image is tied to a specific code commit
- Traceability: Can always know which code version is running
- Example: If a production issue occurs, you can check the exact commit

**Fails if**: Dockerfile has errors → notified

### Stage 4: Notify
```yaml
- name: Summarize pipeline result
  run: echo "## Pipeline Summary" >> $GITHUB_STEP_SUMMARY
```

Writes a summary visible in the Actions tab **regardless of stage results**.

**Always runs** (`if: always()`), so you get feedback even if earlier stages fail.

---

## Deliverables for Your Report

### 1. Screenshot: Workflow List
Show the "Actions" tab with CI/CD Pipeline runs.

### 2. Screenshot: Passing Pipeline Run
Click on a run and show:
- Lint: ✓ success
- Test: ✓ success
- Build: ✓ success
- Notify: ✓ success

### 3. Screenshot: Job Logs
Click into one of the jobs (e.g., "Build") and show:
- Step-by-step execution logs
- Docker build completion

### 4. Screenshot: Pipeline Summary
Show the "Pipeline Summary" output in the Notify job:
```
## Pipeline Summary
- Lint: success
- Test: success
- Build: success
```

### 5. Document: Workflow File
Include the workflow definition (already in repo):
- Path: `.github/workflows/ci-cd.yml`
- Shows all 4 jobs and their dependencies

### 6. Evidence of Automation
Demonstrate that pipeline runs automatically:
- Push a second commit
- Show that Actions tab automatically triggered another run
- Compare two pipeline runs

---

## Key Points Explained

### Sequential vs. Parallel Execution
**Current pipeline:**
```
lint → test → build → notify
```
(Sequential: each stage waits for previous)

**Alternative (faster):**
```
lint ┐
test ├→ build → notify
     ┘
```
(Lint and test run in parallel; build waits for both)

**For this project:** Sequential is clearer for learning; adjust `needs:` in `.github/workflows/ci-cd.yml` for parallelization.

### Environment Variables in Actions
```yaml
- name: Build Docker image
  run: docker build -t myapp:${{ github.sha }} .
```

**`${{ github.sha }}`** is a GitHub context variable that contains:
- Current commit SHA (e.g., `abc123def456789`)
- Unique per commit
- Used to tag Docker image with exact code version

Other useful variables:
- `${{ github.ref_name }}` — Branch name (main, develop, etc.)
- `${{ github.actor }}` — User who pushed the code
- `${{ github.server_url }}` — https://github.com

### Artifacts & Coverage
If you add real Jest tests:
```yaml
- name: Upload coverage report
  uses: actions/upload-artifact@v4
  with:
    name: coverage-report
    path: coverage/
```

Creates a downloadable artifact (coverage reports, test logs, etc.) visible in the Actions tab.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Workflow not showing in Actions tab | Wait 30s; refresh page; check branch is `main` |
| "npm run lint" command not found | Placeholder script; add ESLint if needed |
| "npm test" fails | Placeholder script; add Jest tests if needed |
| Docker build fails | Check Dockerfile syntax; run `docker build -t myapp:latest .` locally first |
| Workflow file not recognized | Ensure `.github/workflows/ci-cd.yml` is on `main` branch; check YAML syntax |
| "Remote origin not found" | Run `git remote add origin https://github.com/...` |

---

## Upgrading the Pipeline (Production Ready)

### Add Real Linting
```bash
npm install --save-dev eslint
npx eslint --init
```
Update `package.json`:
```json
"lint": "eslint . --ext .js,.jsx"
```

### Add Real Testing
```bash
npm install --save-dev jest
npx jest --init
```
Update `package.json`:
```json
"test": "jest --coverage"
```

### Push to Trigger Pipeline Again
```bash
git add .eslintrc.json package.json
git commit -m "Add ESLint and Jest"
git push origin main
```
Check Actions tab — pipeline will run with real checks now.

---

## Estimated Time
- **Setup & push**: 10 minutes
- **First pipeline run**: 30–50 seconds
- **Troubleshooting (if needed)**: 5–10 minutes
- **Total**: ~15–20 minutes

---

## What's Next
After Task 3 verification:
→ Move to [Task 4: Infrastructure as Code + Kubernetes](../task4/TASK4.md)
