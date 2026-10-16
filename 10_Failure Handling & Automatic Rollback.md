

This lab is directly based on the **“Advanced Error Handling and Rollback Logic”** hands-on section in your document. The source teaches `failure()`, `always()`, `success()`, a deployment step with an `id`, an alert step, and a rollback step that runs when the deployment specifically fails. 03 Advanced Techniques and Best…

---

# LAB 10 — Failure Handling & Automatic Rollback

## 1. What are we going to build?

We are going to simulate a production deployment that **fails intentionally**.

Instead of allowing the pipeline to simply stop, GitHub Actions will:

```text
Deployment
     |
     | FAILS
     v
failure()
     |
     +--------------------+
     |                    |
     v                    v
Alert              Automatic Rollback
     |                    |
     +--------------------+
              |
              v
          Cleanup
```

The key learning is:

> A pipeline failure doesn't always mean the pipeline should simply stop. We can define what should happen after a failure.

Your source describes exactly this problem: by default, when a step fails, subsequent steps are skipped, but status-check functions allow us to control what happens next. 03 Advanced Techniques and Best…

---

# 2. What students will learn

By the end of this lab, students will understand:

### `success()`

Run something when previous steps succeeded.

### `failure()`

Run something when an earlier step failed.

### `always()`

Run something regardless of whether previous steps succeeded or failed.

### Step `id`

Give a step an identifier:

```yaml
id: deploy_step
```

Then refer to its result:

```yaml
steps.deploy_step.outcome
```

### Automatic rollback

If deployment fails:

```text
Deployment failed
       ↓
Alert
       ↓
Rollback
       ↓
Cleanup
```

The document explicitly defines these three status-check functions: `always()` runs regardless, `failure()` runs after failure, and `success()` is the default behavior. 03 Advanced Techniques and Best…

---

# PART 1 — Create the GitHub Repository

We will create one new repository.

Repository name:

```text
gha-lab-03-failure-rollback
```

---

## Step 1 — Open GitHub

Open:

```text
https://github.com
```

Log in with:

```text
discover-devops
```

---

# Step 2 — Create the repository

Click:

**+ → New repository**

Enter:

### Repository name

```text
gha-lab-03-failure-rollback
```

### Description

```text
GitHub Actions Failure Handling and Automatic Rollback Lab
```

Select:

```text
Public
```

Do **not** select:

```text
☐ Add a README file
☐ Add .gitignore
☐ Choose a license
```

Click:

**Create repository**

---

# PART 2 — Open Git Bash

We are going to use your existing working directory.

Your working directory is:

```text
/p/Scaler/DevOpsTool_2/GitHubAction
```

We are **not** creating another parent directory.

---

# Step 3 — Go to the working directory

Open Git Bash.

Run:

```bash
cd /p/Scaler/DevOpsTool_2/GitHubAction
```

Now run:

```bash
pwd
```

Expected:

```text
/p/Scaler/DevOpsTool_2/GitHubAction
```

---

# Step 4 — Check existing directories

Run:

```bash
ls
```

You should already have:

```text
gha-lab-01-central
gha-lab-01-microservice
gha-lab-02-github-env
```

Now we will add:

```text
gha-lab-03-failure-rollback
```

---

# PART 3 — Clone the Repository

## Step 5 — Clone the repository

Run:

```bash
git clone https://github.com/discover-devops/gha-lab-03-failure-rollback.git
```

You should see:

```text
Cloning into 'gha-lab-03-failure-rollback'...
```

---

# Step 6 — Verify

Run:

```bash
ls
```

You should now see:

```text
gha-lab-01-central
gha-lab-01-microservice
gha-lab-02-github-env
gha-lab-03-failure-rollback
```

---

# PART 4 — Enter LAB 3 Directory

## Step 7

Run:

```bash
cd /p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-03-failure-rollback
```

Now:

```bash
pwd
```

Expected:

```text
/p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-03-failure-rollback
```

---

# Step 8 — Check repository

Run:

```bash
ls -la
```

You should see something similar to:

```text
.
..
.git
```

The repository is empty.

---

# PART 5 — Create the Scripts Directory

We are going to create two scripts:

```text
deploy.sh
rollback.sh
```

Our structure will eventually be:

```text
gha-lab-03-failure-rollback/
│
├── scripts/
│   ├── deploy.sh
│   └── rollback.sh
│
└── .github/
    └── workflows/
        └── failure-rollback.yml
```

---

# Step 9 — Create `scripts`

Run:

```bash
mkdir scripts
```

Verify:

```bash
ls
```

Expected:

```text
scripts
```

---

# PART 6 — Create the Deployment Script

The deployment script will intentionally fail.

Why?

Because we want to demonstrate GitHub Actions failure handling.

---

# Step 10 — Create `deploy.sh`

Run:

```bash
notepad scripts/deploy.sh
```

If Windows asks whether you want to create the file, click:

**Yes**

Paste exactly:

```bash
#!/bin/bash

echo "================================="
echo "Starting Production Deployment"
echo "================================="

echo "Deploying application..."
echo "Updating production servers..."
echo "Running deployment checks..."

echo "ERROR: Deployment failed!"

exit 1
```

Save:

**Ctrl + S**

Close Notepad.

---

# Step 11 — Make the script executable

Back in Git Bash, run:

```bash
chmod +x scripts/deploy.sh
```

---

# Step 12 — Verify permissions

Run:

```bash
ls -l scripts
```

You should see something similar to:

```text
-rwxr-xr-x ... deploy.sh
```

The important part is:

```text
x
```

That means the file is executable.

---

# PART 7 — Test the Deployment Script

Before we involve GitHub Actions, let's run the script locally.

## Step 13

Run:

```bash
./scripts/deploy.sh
```

You should see:

```text
=================================
Starting Production Deployment
=================================
Deploying application...
Updating production servers...
Running deployment checks...
ERROR: Deployment failed!
```

You will also get a non-zero exit code because the script contains:

```bash
exit 1
```

That is intentional.

---

# Why `exit 1`?

In Linux shell scripting:

```bash
exit 0
```

generally means:

```text
Success
```

while:

```bash
exit 1
```

means:

```text
Failure
```

So our GitHub Actions runner will understand:

```text
deploy.sh
   |
   v
exit 1
   |
   v
Deployment step FAILED
```

---

# PART 8 — Create the Rollback Script

Now we need the recovery mechanism.

---

# Step 14 — Create `rollback.sh`

Run:

```bash
notepad scripts/rollback.sh
```

Paste:

```bash
#!/bin/bash

echo "================================="
echo "Starting Automatic Rollback"
echo "================================="

echo "Restoring previous application version..."
echo "Restoring production servers..."
echo "Running rollback checks..."

echo "Rollback completed successfully."
```

Save:

**Ctrl + S**

Close Notepad.

---

# Step 15 — Make rollback executable

Run:

```bash
chmod +x scripts/rollback.sh
```

---

# Step 16 — Verify

Run:

```bash
ls -l scripts
```

You should now see:

```text
deploy.sh
rollback.sh
```

Both should have executable permissions.

---

# Step 17 — Test rollback locally

Run:

```bash
./scripts/rollback.sh
```

Expected:

```text
=================================
Starting Automatic Rollback
=================================
Restoring previous application version...
Restoring production servers...
Running rollback checks...
Rollback completed successfully.
```

---

# PART 9 — Create the GitHub Actions Workflow

Now we will create the actual GitHub Actions workflow.

---

# Step 18 — Create workflow directory

Run:

```bash
mkdir -p .github/workflows
```

Verify:

```bash
ls -la
```

You should now see:

```text
.github
.git
scripts
```

---

# Step 19 — Verify `.github`

Run:

```bash
ls .github
```

Expected:

```text
workflows
```

Run:

```bash
ls .github/workflows
```

It should currently be empty.

---

# PART 10 — Create `failure-rollback.yml`

Run:

```bash
notepad .github/workflows/failure-rollback.yml
```

Paste exactly:

```yaml
name: Failure and Rollback Lab

on:
  workflow_dispatch:

jobs:
  deployment:
    runs-on: ubuntu-latest

    steps:

      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Production Deployment
        id: deploy_step
        run: ./scripts/deploy.sh

      - name: Alert Engineering
        if: failure()
        run: |
          echo "================================="
          echo "ALERT: Production deployment failed!"
          echo "================================="
          echo "Sending notification to engineering team..."

      - name: Automatic Rollback
        if: failure() && steps.deploy_step.outcome == 'failure'
        run: ./scripts/rollback.sh

      - name: Cleanup
        if: always()
        run: |
          echo "================================="
          echo "Cleanup"
          echo "================================="
          echo "Cleanup always runs."
```

Save:

**Ctrl + S**

Close Notepad.

---

# PART 11 — Understand the Workflow

Let's understand the important pieces.

---

## Step 20 — Deployment step

We have:

```yaml
- name: Production Deployment
  id: deploy_step
  run: ./scripts/deploy.sh
```

The important part is:

```yaml
id: deploy_step
```

We give this step a name that GitHub Actions can reference later.

The script intentionally fails:

```bash
exit 1
```

Therefore:

```text
Production Deployment
        |
        v
     FAILURE
```

---

# Step 21 — Alert step

Next:

```yaml
- name: Alert Engineering
  if: failure()
```

This means:

> If something earlier in the job failed, execute this step.

The source uses the same `failure()` mechanism for a failure notification step. 03 Advanced Techniques and Best…

So:

```text
Deployment
    |
   FAIL
    |
    v
failure()
    |
    v
Alert Engineering
```

---

# Step 22 — Rollback step

This is the most important condition:

```yaml
if: failure() && steps.deploy_step.outcome == 'failure'
```

There are two conditions.

### Condition 1

```text
failure()
```

Something earlier failed.

### Condition 2

```text
steps.deploy_step.outcome == 'failure'
```

The specific deployment step failed.

So the meaning is:

> If the workflow has entered a failure state AND the deployment step specifically failed, execute rollback.

This follows the source lab's exact logic. 03 Advanced Techniques and Best…

---

# Step 23 — Cleanup

Finally:

```yaml
if: always()
```

This means:

> Run this step whether the deployment succeeded or failed.

The source explicitly describes `always()` as running regardless of previous success or failure. 03 Advanced Techniques and Best…

---

# PART 12 — Verify the Files

Run:

```bash
ls scripts
```

Expected:

```text
deploy.sh
rollback.sh
```

Run:

```bash
ls .github/workflows
```

Expected:

```text
failure-rollback.yml
```

---

# PART 13 — Final Local Directory Structure

Run:

```bash
find . -maxdepth 3 -type f
```

You should have something similar to:

```text
./.github/workflows/failure-rollback.yml
./scripts/deploy.sh
./scripts/rollback.sh
```

The important structure is:

```text
gha-lab-03-failure-rollback/
│
├── .github/
│   └── workflows/
│       └── failure-rollback.yml
│
└── scripts/
    ├── deploy.sh
    └── rollback.sh
```

---

# PART 14 — Git Status

## Step 24

Run:

```bash
git status
```

You should see:

```text
Untracked files:
    .github/
    scripts/
```

---

# Step 25 — Add files

Run:

```bash
git add .
```

---

# Step 26 — Check status

Run:

```bash
git status
```

You should now see:

```text
Changes to be committed:
    new file: .github/workflows/failure-rollback.yml
    new file: scripts/deploy.sh
    new file: scripts/rollback.sh
```

---

# PART 15 — Commit

## Step 27

Run:

```bash
git commit -m "Add failure handling and automatic rollback"
```

You should see something similar to:

```text
[main xxxxxxx] Add failure handling and automatic rollback
```

---

# PART 16 — Push to GitHub

## Step 28

Run:

```bash
git push -u origin main
```

You should see:

```text
main -> main
```

---

# PART 17 — Verify on GitHub

Open:

```text
https://github.com/discover-devops/gha-lab-03-failure-rollback
```

You should see:

```text
.github
scripts
```

Open `scripts`.

You should see:

```text
deploy.sh
rollback.sh
```

Open:

```text
.github/workflows
```

You should see:

```text
failure-rollback.yml
```

---

# PART 18 — Run the Workflow

Now go to:

**GitHub → `gha-lab-03-failure-rollback` → Actions**

You should see:

```text
Failure and Rollback Lab
```

Click it.

Because we used:

```yaml
on:
  workflow_dispatch:
```

you will see:

**Run workflow**

Click:

**Run workflow**

---

# PART 19 — Watch the Pipeline

Click the newly created workflow run.

You should see:

```text
deployment
```

Click the job.

Now watch the steps.

You should get:

```text
✓ Checkout Code
✗ Production Deployment
✓ Alert Engineering
✓ Automatic Rollback
✓ Cleanup
```

This is the most important visual result of the lab.

---

# PART 20 — Examine Production Deployment

Click:

**Production Deployment**

You should see:

```text
=================================
Starting Production Deployment
=================================
Deploying application...
Updating production servers...
Running deployment checks...
ERROR: Deployment failed!
```

The step will have a red X.

That's intentional.

---

# PART 21 — Examine Alert Engineering

Now click:

**Alert Engineering**

You should see:

```text
=================================
ALERT: Production deployment failed!
=================================
Sending notification to engineering team...
```

Notice something important:

The deployment failed, but the alert step **did not get skipped**.

Why?

Because:

```yaml
if: failure()
```

told GitHub Actions to execute it when a previous step failed.

---

# PART 22 — Examine Automatic Rollback

Click:

**Automatic Rollback**

You should see:

```text
=================================
Starting Automatic Rollback
=================================
Restoring previous application version...
Restoring production servers...
Running rollback checks...
Rollback completed successfully.
```

This proves our rollback logic worked.

---

# PART 23 — Examine Cleanup

Click:

**Cleanup**

You should see:

```text
=================================
Cleanup
=================================
Cleanup always runs.
```

The deployment failed.

The alert ran.

The rollback ran.

And cleanup still ran.

That's because:

```yaml
if: always()
```

---

# PART 24 — The Complete Failure Flow

This is the architecture you should explain to students:

```text
             GitHub Actions
                   |
                   v
          Production Deployment
                   |
                   v
                exit 1
                   |
                   v
              FAILURE
                   |
        +----------+----------+
        |                     |
        v                     v
  failure()             failure()
        |                     |
        v                     v
     ALERT              Check deploy_step
                              |
                              v
                     outcome == failure
                              |
                              v
                         ROLLBACK
                              |
                              v
                          CLEANUP
                              |
                              v
                         always()
```

---

# PART 25 — The Three Status Functions

Students should memorize these three.

## 1. `success()`

```yaml
if: success()
```

Means:

> Run this step when previous steps succeeded.

The source identifies `success()` as the default behavior. 03 Advanced Techniques and Best…

---

## 2. `failure()`

```yaml
if: failure()
```

Means:

> Run this step when something earlier failed.

Example:

```yaml
- name: Alert Engineering
  if: failure()
  run: echo "Deployment failed!"
```

---

## 3. `always()`

```yaml
if: always()
```

Means:

> Run this step regardless of whether previous steps succeeded or failed.

Example:

```yaml
- name: Cleanup
  if: always()
  run: echo "Cleanup"
```

The source's MCQ specifically tests this behavior: when a report must upload regardless of whether tests passed or failed, the answer is `if: always()`. 03 Advanced Techniques and Best…

---

# PART 26 — Why Do We Need `steps.deploy_step.outcome`?

This is an important interview-level concept.

Suppose the workflow has several steps:

```text
Step 1
Step 2
Deployment
Step 4
```

We want rollback specifically when **deployment** failed.

So we give deployment an ID:

```yaml
id: deploy_step
```

Then we can ask:

```yaml
steps.deploy_step.outcome
```

And check:

```yaml
steps.deploy_step.outcome == 'failure'
```

So:

```text
deploy_step
     |
     v
outcome
     |
     +---- success
     |
     +---- failure
```

Our rollback is therefore specifically tied to the deployment failure.

---

# PART 27 — Real-World Analogy

Use this analogy while teaching.

Imagine Hotstar is deploying a new application version.

```text
Deploy Version 25
       |
       v
10 servers
       |
       v
Deployment starts
       |
       v
5 servers updated
       |
       X
Deployment fails
```

If we simply stop:

```text
5 servers → new version
5 servers → old version
```

Now we have an inconsistent production environment.

Instead:

```text
Deployment Failure
       |
       v
Alert Engineering
       |
       v
Rollback
       |
       v
Previous Stable Version
       |
       v
Production Stable
```

That is the idea behind **self-healing pipeline behavior**.

The source describes the same broader problem as a deployment failing midway and leaving production in a broken state until someone manually rolls it back. 03 Advanced Techniques and Best…

---

# PART 28 — One Important Teaching Point

The pipeline itself will still show a **failed workflow** because our deployment intentionally executes:

```bash
exit 1
```

That's okay.

The important thing is that:

```text
Deployment → Failed
Rollback → Successful
```

The objective of this lab is **not to make the deployment magically become successful**.

The objective is to demonstrate:

> **Failure detection + recovery action.**

This distinction is important when teaching students.

---

# PART 29 — Final Directory Structure

Your complete working area should now look like:

```text
/p/Scaler/DevOpsTool_2/GitHubAction/
│
├── gha-lab-01-central/
│
├── gha-lab-01-microservice/
│
├── gha-lab-02-github-env/
│
└── gha-lab-03-failure-rollback/
    │
    ├── .github/
    │   └── workflows/
    │       └── failure-rollback.yml
    │
    └── scripts/
        ├── deploy.sh
        └── rollback.sh
```

---

# PART 30 — Complete Command List

For students who want the commands without the explanations:

## Start

```bash
cd /p/Scaler/DevOpsTool_2/GitHubAction
```

## Clone

```bash
git clone https://github.com/discover-devops/gha-lab-03-failure-rollback.git
```

## Enter

```bash
cd /p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-03-failure-rollback
```

## Create directories

```bash
mkdir scripts
mkdir -p .github/workflows
```

## Create deployment script

```bash
notepad scripts/deploy.sh
```

Paste the `deploy.sh` content.

Then:

```bash
chmod +x scripts/deploy.sh
```

## Create rollback script

```bash
notepad scripts/rollback.sh
```

Paste the `rollback.sh` content.

Then:

```bash
chmod +x scripts/rollback.sh
```

## Test scripts

```bash
./scripts/deploy.sh
```

Expected: deployment failure.

Then:

```bash
./scripts/rollback.sh
```

Expected: rollback successful.

## Create workflow

```bash
notepad .github/workflows/failure-rollback.yml
```

Paste the complete YAML.

## Git

```bash
git status
```

```bash
git add .
```

```bash
git status
```

```bash
git commit -m "Add failure handling and automatic rollback"
```

```bash
git push -u origin main
```

## Run

GitHub:

```text
Repository
   ↓
Actions
   ↓
Failure and Rollback Lab
   ↓
Run workflow
   ↓
Run workflow
```

---

# PART 31 — What Students Learned

At the end of LAB 3, students should be able to answer:

### What happens when a GitHub Actions step fails?

By default, subsequent steps are skipped. 03 Advanced Techniques and Best…

### How do we run something after a failure?

```yaml
if: failure()
```

### How do we always run cleanup?

```yaml
if: always()
```

### How do we identify a particular step?

```yaml
id: deploy_step
```

### How do we check whether that specific step failed?

```yaml
steps.deploy_step.outcome == 'failure'
```

### How do we implement rollback?

```yaml
- name: Automatic Rollback
  if: failure() && steps.deploy_step.outcome == 'failure'
  run: ./scripts/rollback.sh
```

---

# Final takeaway

> **A production pipeline should not just detect failure; it should know what action to take when failure happens.**

In this lab:

```text
FAILURE
   ↓
DETECT
   ↓
ALERT
   ↓
ROLLBACK
   ↓
CLEANUP
```

That is the foundation for the **self-healing pipeline** we will build in **LAB 4**. The source material explicitly positions failure handling and rollback as the safety net that leads into the enterprise-grade self-healing pipeline. 03 Advanced Techniques and Best…
