
# LAB 10 — Building an Enterprise-Grade, Self-Healing Pipeline

## What are we building?

We are going to combine everything from Labs 1–3:

```text
LAB 1
Reusable Workflow
       +
LAB 2
$GITHUB_ENV
       +
LAB 3
Failure + Rollback
       +
Security Hardening
       ↓
================================
ENTERPRISE SELF-HEALING PIPELINE
================================
```

The document's intended architecture is:

```text
Developer Push
      |
      v
GitHub Repository
      |
      v
Central Reusable Build
      |
      v
Build + Test
      |
      v
Deployment Phase
      |
      +----------------------+
      |                      |
    SUCCESS                FAILURE
      |                      |
      v                      v
 Application              failure()
    Live                     |
                             v
                         Rollback
                             |
                             v
                           Alert
```

The document specifically describes this as combining reusable workflows, dynamic variables, error handling and security hardening. 03 Advanced Techniques and Best…

---

# PART 1 — What Students Should Understand Before Starting

This is no longer a new isolated GitHub Actions feature.

We are taking the previous three labs and putting them together.

### LAB 1 taught:

```text
workflow_call
```

and:

```text
uses:
```

### LAB 2 taught:

```text
$GITHUB_ENV
```

### LAB 3 taught:

```text
failure()
```

and:

```text
steps.<id>.outcome
```

and rollback.

### Security section taught:

```yaml
permissions:
  contents: read
```

and SHA pinning for third-party Actions. The document specifically presents least-privilege permissions and pinning Actions to immutable commit SHAs as security practices. 03 Advanced Techniques and Best…

Now we combine them.

---

# PART 2 — Our Repository

We will create one new repository.

### Repository name

```text
gha-lab-04-enterprise-self-healing
```

Your GitHub username:

```text
discover-devops
```

Therefore the repository URL will be:

```text
https://github.com/discover-devops/gha-lab-04-enterprise-self-healing
```

---

# PART 3 — Our Local Directory

You told me your working directory is:

```text
/p/Scaler/DevOpsTool_2/GitHubAction
```

We will continue using exactly that.

We are **not** creating another parent directory.

Our final directory structure will be:

```text
/p/Scaler/DevOpsTool_2/GitHubAction/
│
├── gha-lab-01-central/
├── gha-lab-01-microservice/
├── gha-lab-02-github-env/
├── gha-lab-03-failure-rollback/
│
└── gha-lab-04-enterprise-self-healing/
    │
    ├── .github/
    │   └── workflows/
    │       └── enterprise-deploy.yml
    │
    ├── scripts/
    │   ├── deploy.sh
    │   └── safe-rollback.sh
    │
    ├── package.json
    ├── package-lock.json
    └── test.js
```

Why do we need `package.json` and `package-lock.json`?

Because our **LAB 1 reusable build workflow** contains:

```yaml
- run: npm ci
- run: npm test
```

The document's reusable workflow uses the same `npm ci` / `npm test` pattern. 03 Advanced Techniques and Best…

Therefore this Lab 4 repository needs a small Node.js application for the reusable build to process.

---

# PART 4 — Create the GitHub Repository

## Step 1 — Open GitHub

Go to:

```text
https://github.com
```

Sign in as:

```text
discover-devops
```

---

## Step 2 — Create repository

Click:

**+ → New repository**

Enter:

### Repository name

```text
gha-lab-04-enterprise-self-healing
```

### Description

```text
Enterprise Grade Self-Healing GitHub Actions Pipeline
```

Select:

```text
Public
```

Do NOT select:

```text
☐ Add a README file
☐ Add .gitignore
☐ Choose a license
```

Click:

**Create repository**

---

# PART 5 — Open Git Bash

Open **Git Bash**.

---

## Step 3 — Go to your working directory

Run:

```bash
cd /p/Scaler/DevOpsTool_2/GitHubAction
```

Verify:

```bash
pwd
```

Expected:

```text
/p/Scaler/DevOpsTool_2/GitHubAction
```

---

# PART 6 — Clone the Repository

## Step 4 — Clone

Run:

```bash
git clone https://github.com/discover-devops/gha-lab-04-enterprise-self-healing.git
```

You should see:

```text
Cloning into 'gha-lab-04-enterprise-self-healing'...
```

---

## Step 5 — Verify

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
gha-lab-04-enterprise-self-healing
```

---

# PART 7 — Enter LAB 4

## Step 6

Run:

```bash
cd /p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-04-enterprise-self-healing
```

Verify:

```bash
pwd
```

Expected:

```text
/p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-04-enterprise-self-healing
```

---

## Step 7 — Check the repository

Run:

```bash
ls -la
```

You should see:

```text
.
..
.git
```

The repository is empty.

---

# PART 8 — Create the Application Files

We need a tiny application because our reusable workflow will execute:

```text
npm ci
npm test
```

---

# Step 8 — Create `package.json`

Run:

```bash
notepad package.json
```

Paste exactly:

```json
{
  "name": "gha-enterprise-self-healing",
  "version": "1.0.0",
  "description": "Enterprise self-healing GitHub Actions lab",
  "scripts": {
    "test": "node test.js"
  }
}
```

Save:

**Ctrl + S**

Close Notepad.

---

# Step 9 — Create `test.js`

Run:

```bash
notepad test.js
```

Paste:

```javascript
console.log("=================================");
console.log("Running Application Tests");
console.log("=================================");

console.log("Test 1: Application configuration");
console.log("Test 2: Application startup");
console.log("Test 3: Application health");

console.log("All tests passed!");
```

Save and close Notepad.

---

# PART 9 — Generate `package-lock.json`

This is important because our reusable workflow uses:

```bash
npm ci
```

`npm ci` requires a lock file.

---

## Step 10 — Generate the lock file

Run:

```bash
npm install
```

You should see npm output and eventually something similar to:

```text
added 1 package
```

or:

```text
up to date
```

Now run:

```bash
ls
```

You should have:

```text
package.json
package-lock.json
test.js
```

---

# Step 11 — Test locally

Before involving GitHub Actions, run:

```bash
npm test
```

Expected:

```text
=================================
Running Application Tests
=================================
Test 1: Application configuration
Test 2: Application startup
Test 3: Application health
All tests passed!
```

This is important.

We know our application can pass the build/test phase before we move to the deployment phase.

---

# PART 10 — Create the Scripts Directory

Run:

```bash
mkdir scripts
```

Verify:

```bash
ls
```

You should now see:

```text
package.json
package-lock.json
scripts
test.js
```

---

# PART 11 — Create the Deployment Script

The document intentionally makes the deployment fail using:

```bash
exit 1
```

so that we can demonstrate the self-healing mechanism. 03 Advanced Techniques and Best…

---

## Step 12 — Create `deploy.sh`

Run:

```bash
notepad scripts/deploy.sh
```

Paste:

```bash
#!/bin/bash

echo "================================="
echo "Enterprise Production Deployment"
echo "================================="

echo "Deploying release: $RELEASE_TAG"
echo "Connecting to production servers..."
echo "Uploading application..."
echo "Starting application..."
echo "Running deployment validation..."

echo "ERROR: Production deployment failed!"

exit 1
```

Save and close Notepad.

---

## Step 13 — Make it executable

Run:

```bash
chmod +x scripts/deploy.sh
```

---

## Step 14 — Test it

Run:

```bash
./scripts/deploy.sh
```

Expected:

```text
=================================
Enterprise Production Deployment
=================================
Deploying release:
Connecting to production servers...
Uploading application...
Starting application...
Running deployment validation...
ERROR: Production deployment failed!
```

The command will return a failure because of:

```bash
exit 1
```

That is intentional.

---

# PART 12 — Create the Rollback Script

The document calls this:

```text
scripts/safe-rollback.sh
```

and executes it after deployment failure. 03 Advanced Techniques and Best…

---

## Step 15 — Create the file

Run:

```bash
notepad scripts/safe-rollback.sh
```

Paste:

```bash
#!/bin/bash

echo "================================="
echo "AUTOMATIC SAFE ROLLBACK"
echo "================================="

echo "Stopping failed release..."
echo "Restoring previous stable release..."
echo "Restoring production configuration..."
echo "Restarting application..."
echo "Running health checks..."

echo "Rollback completed successfully."
echo "Production application is stable."
```

Save and close Notepad.

---

## Step 16 — Make executable

Run:

```bash
chmod +x scripts/safe-rollback.sh
```

---

## Step 17 — Test rollback

Run:

```bash
./scripts/safe-rollback.sh
```

Expected:

```text
=================================
AUTOMATIC SAFE ROLLBACK
=================================
Stopping failed release...
Restoring previous stable release...
Restoring production configuration...
Restarting application...
Running health checks...
Rollback completed successfully.
Production application is stable.
```

---

# PART 13 — Create GitHub Actions Directory

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
package.json
package-lock.json
scripts
test.js
```

---

# PART 14 — Create the Enterprise Workflow

This is the main file for the lab:

```text
.github/workflows/enterprise-deploy.yml
```

The document explicitly asks us to create this workflow for Section 7. 03 Advanced Techniques and Best…

---

## Step 18 — Create the workflow

Run:

```bash
notepad .github/workflows/enterprise-deploy.yml
```

Paste this **exactly**:

```yaml
name: Enterprise Self-Healing Deployment

on:
  push:
    branches:
      - main

permissions:
  contents: read
  deployments: write

jobs:

  build-phase:
    uses: discover-devops/gha-lab-01-central/.github/workflows/reusable-build.yml@main
    with:
      node_version: '20'

  deploy-phase:
    needs: build-phase
    runs-on: ubuntu-latest

    steps:

      - name: Secure Checkout
        uses: actions/checkout@1d96c772d19495a3b5c517cd2bc0cb401ea0529f

      - name: Generate Release Tag
        run: |
          echo "RELEASE_TAG=v1.0.${{ github.run_number }}" >> $GITHUB_ENV
          echo "Generated release: v1.0.${{ github.run_number }}"

      - name: Attempt Deployment
        id: deploy
        run: |
          echo "Deploying $RELEASE_TAG to production servers..."
          ./scripts/deploy.sh

      - name: Execute Rollback
        if: failure()
        run: |
          echo "WARNING: Deployment of $RELEASE_TAG failed!"
          echo "Initiating automatic rollback procedures..."
          ./scripts/safe-rollback.sh

      - name: Alert Engineering Team
        if: failure()
        run: |
          echo "Sending critical failure alert to Slack..."
          echo "Deployment failed."
          echo "Automatic rollback was triggered."
```

Save:

**Ctrl + S**

Close Notepad.

---

# PART 15 — Stop and Understand the Architecture

Before pushing anything, let's understand the workflow.

This is the heart of the lab.

---

## Step 19 — Build Phase

We have:

```yaml
build-phase:
  uses: discover-devops/gha-lab-01-central/.github/workflows/reusable-build.yml@main
```

This is **LAB 1 coming back into the architecture**.

We are saying:

> Don't duplicate the build logic. Use the central reusable workflow.

The document describes exactly this pattern: the enterprise workflow calls the central reusable build and passes Node.js version 20. 03 Advanced Techniques and Best…

So:

```text
LAB 4
   |
   v
gha-lab-01-central
   |
   v
reusable-build.yml
   |
   +-- Checkout
   +-- Setup Node
   +-- npm ci
   +-- npm test
```

---

# PART 16 — Why Does `build-phase` Have No `runs-on`?

Students may ask this.

We have:

```yaml
build-phase:
  uses: ...
```

We do **not** have:

```yaml
runs-on:
```

Why?

Because this is a **reusable workflow call**.

The reusable workflow itself defines:

```yaml
runs-on: ubuntu-latest
```

The caller simply says:

```text
Use this workflow.
```

That is exactly the modularization principle from LAB 1.

---

# PART 17 — `needs: build-phase`

Next:

```yaml
deploy-phase:
  needs: build-phase
```

This means:

```text
BUILD
  |
  | must complete first
  v
DEPLOY
```

So our pipeline becomes:

```text
Push
  |
  v
Build
  |
  | SUCCESS
  v
Deploy
```

If build fails, deployment does not proceed.

---

# PART 18 — Security Hardening

At the top we have:

```yaml
permissions:
  contents: read
  deployments: write
```

This is **least privilege**.

The document specifically recommends restricting the `GITHUB_TOKEN` permissions instead of leaving broad permissions enabled. 03 Advanced Techniques and Best…

Think:

```text
Don't give the pipeline
        |
        v
"Master key to everything"
```

Instead:

```text
Pipeline
   |
   +-- Read repository
   |
   +-- Deployment permission
   |
   X
   No unnecessary permissions
```

---

# PART 19 — SHA Pinning

Look at:

```yaml
uses: actions/checkout@1d96c772d19495a3b5c517cd2bc0cb401ea0529f
```

Notice we are **not** using:

```yaml
uses: actions/checkout@v4
```

The document specifically uses this commit SHA as its example of SHA pinning. 03 Advanced Techniques and Best…

The reason is supply-chain security.

A version tag such as:

```text
@v4
```

can potentially move.

A specific commit SHA points to a specific commit.

The document's security MCQ explicitly identifies the immutability of the commit SHA as the reason it is more secure than a movable version tag. 03 Advanced Techniques and Best…

---

# PART 20 — Dynamic Release Tag

Now we use what we learned in LAB 2.

```yaml
- name: Generate Release Tag
  run: |
    echo "RELEASE_TAG=v1.0.${{ github.run_number }}" >> $GITHUB_ENV
```

Suppose this is workflow run:

```text
1
```

Then:

```text
RELEASE_TAG=v1.0.1
```

Run 2:

```text
RELEASE_TAG=v1.0.2
```

Run 3:

```text
RELEASE_TAG=v1.0.3
```

And because we write it to:

```text
$GITHUB_ENV
```

the later deployment step can access:

```bash
$RELEASE_TAG
```

This is exactly the dynamic release-tag pattern in Section 7. 03 Advanced Techniques and Best…

---

# PART 21 — Deployment

Now:

```yaml
- name: Attempt Deployment
  id: deploy
  run: |
    echo "Deploying $RELEASE_TAG to production servers..."
    ./scripts/deploy.sh
```

Notice:

```yaml
id: deploy
```

We give the deployment step an ID.

The script then intentionally fails:

```bash
exit 1
```

So:

```text
Deploy v1.0.1
     |
     v
Failure
```

---

# PART 22 — Self-Healing Rollback

Now the important part:

```yaml
- name: Execute Rollback
  if: failure()
```

If deployment fails:

```text
failure()
   |
   v
Execute Rollback
```

Then:

```bash
./scripts/safe-rollback.sh
```

The document uses this exact self-healing pattern. 03 Advanced Techniques and Best…

---

# PART 23 — Alert

Finally:

```yaml
- name: Alert Engineering Team
  if: failure()
```

So after failure:

```text
Deployment Failed
       |
       +----> Rollback
       |
       +----> Alert
```

The document describes this as the automated safety net that prevents Alex from having to manually wake up and roll back production. 03 Advanced Techniques and Best…

---

# PART 24 — Verify the Files

Run:

```bash
ls
```

Expected:

```text
package.json
package-lock.json
scripts
test.js
```

Now:

```bash
ls scripts
```

Expected:

```text
deploy.sh
safe-rollback.sh
```

Now:

```bash
ls .github/workflows
```

Expected:

```text
enterprise-deploy.yml
```

---

# PART 25 — Final Directory Structure

At this point you should have:

```text
gha-lab-04-enterprise-self-healing/
│
├── .github/
│   └── workflows/
│       └── enterprise-deploy.yml
│
├── scripts/
│   ├── deploy.sh
│   └── safe-rollback.sh
│
├── package.json
├── package-lock.json
└── test.js
```

---

# PART 26 — Check Git Status

Run:

```bash
git status
```

You should see:

```text
Untracked files:
  .github/
  package.json
  package-lock.json
  scripts/
  test.js
```

---

# PART 27 — Add Everything

Run:

```bash
git add .
```

---

# PART 28 — Verify

Run:

```bash
git status
```

You should now see files under:

```text
Changes to be committed:
```

---

# PART 29 — Commit

Run:

```bash
git commit -m "Build enterprise self-healing pipeline"
```

You should see something similar to:

```text
[main xxxxxxx] Build enterprise self-healing pipeline
```

---

# PART 30 — Push

Run:

```bash
git push -u origin main
```

This push is important because our workflow trigger is:

```yaml
on:
  push:
    branches:
      - main
```

So the push itself starts the pipeline.

---

# PART 31 — What Will Happen Now?

This is where the whole lab comes together.

When you execute:

```bash
git push
```

GitHub Actions starts.

The flow is:

```text
git push
    |
    v
GitHub
    |
    v
Enterprise Self-Healing Deployment
    |
    v
BUILD-PHASE
    |
    v
Central reusable-build.yml
    |
    +---- Checkout
    |
    +---- Node 20
    |
    +---- npm ci
    |
    +---- npm test
    |
    v
BUILD SUCCESS
    |
    v
DEPLOY-PHASE
    |
    v
Secure Checkout
    |
    v
Generate RELEASE_TAG
    |
    v
$GITHUB_ENV
    |
    v
Deploy v1.0.X
    |
    v
exit 1
    |
    v
FAILURE
    |
    +-------------------+
    |                   |
    v                   v
ROLLBACK              ALERT
    |                   |
    +-------------------+
            |
            v
      Production Stable
```

This is the complete enterprise self-healing architecture described in the document. 03 Advanced Techniques and Best…

---

# PART 32 — Go to GitHub Actions

Open:

```text
https://github.com/discover-devops/gha-lab-04-enterprise-self-healing
```

Click:

**Actions**

You should see:

```text
Enterprise Self-Healing Deployment
```

Click the workflow run.

---

# PART 33 — Observe the Build Phase

You should see something similar to:

```text
build-phase
```

Click it.

Because it is calling:

```text
gha-lab-01-central
```

you should see the reusable workflow execute.

The important steps are:

```text
Checkout Code
Setup Node.js
Install Dependencies
Run Tests
```

The test should produce:

```text
All tests passed!
```

---

# PART 34 — Observe the Deployment Phase

After build succeeds, the next job should start:

```text
deploy-phase
```

This happens because:

```yaml
needs: build-phase
```

---

# PART 35 — Observe Secure Checkout

You should see:

```text
Secure Checkout
```

The workflow is using the SHA-pinned version:

```text
actions/checkout@1d96c772d19495a3b5c517cd2bc0cb401ea0529f
```

This demonstrates the security-hardening concept from the document. 03 Advanced Techniques and Best…

---

# PART 36 — Observe Release Tag

Click:

**Generate Release Tag**

You should see something like:

```text
Generated release: v1.0.1
```

The exact number depends on the workflow run number.

---

# PART 37 — Observe Deployment Failure

Click:

**Attempt Deployment**

You should see:

```text
Deploying v1.0.1 to production servers...
```

followed by:

```text
ERROR: Production deployment failed!
```

The step fails intentionally.

---

# PART 38 — Observe Automatic Rollback

Now the important part.

Click:

**Execute Rollback**

You should see:

```text
WARNING: Deployment of v1.0.1 failed!
Initiating automatic rollback procedures...
=================================
AUTOMATIC SAFE ROLLBACK
=================================
Stopping failed release...
Restoring previous stable release...
Restoring production configuration...
Restarting application...
Running health checks...
Rollback completed successfully.
Production application is stable.
```

This is our self-healing mechanism.

---

# PART 39 — Observe the Alert

Click:

**Alert Engineering Team**

You should see:

```text
Sending critical failure alert to Slack...
Deployment failed.
Automatic rollback was triggered.
```

The document represents this as the Slack notification to engineering. 03 Advanced Techniques and Best…

Our lab prints the alert to the GitHub Actions log rather than connecting to a real Slack account.

---

# PART 40 — Very Important Teaching Point

The overall workflow may show a **failure** because:

```bash
./scripts/deploy.sh
```

returns:

```bash
exit 1
```

That is intentional.

The important result is:

```text
Deployment
    ❌
     |
     v
Rollback
    ✅
     |
     v
Alert
    ✅
```

We are demonstrating **failure recovery**, not pretending the deployment succeeded.

The document's own scenario describes the deployment script crashing with `exit 1`, followed by automatic rollback and alerting. 03 Advanced Techniques and Best…

---

# PART 41 — What Makes This "Self-Healing"?

Ask students:

> What did the pipeline do that a basic CI/CD pipeline would not do?

A basic pipeline:

```text
Deploy
  |
  X
FAIL
```

Human intervention required.

Our pipeline:

```text
Deploy
  |
  X
FAIL
  |
  v
Detect Failure
  |
  v
Rollback Automatically
  |
  v
Alert Engineering
```

The pipeline responds to its own failure.

That is why we call it:

**Self-Healing Pipeline**

---

# PART 42 — Compare Labs 1–4

Now we can connect the entire course section.

## LAB 1 — Reusable Workflow

Problem:

```text
20 repositories
20 copies of YAML
```

Solution:

```text
Central reusable workflow
        |
        +---- Repo A
        +---- Repo B
        +---- Repo C
        +---- ...
```

The document calls reusable workflows the "blueprints" for eliminating duplicated pipeline code. 03 Advanced Techniques and Best…

---

## LAB 2 — `$GITHUB_ENV`

Problem:

```text
Step 1 generates version
        |
        X
Step 2 cannot see it
```

Solution:

```text
Step 1
  |
  v
$GITHUB_ENV
  |
  v
Step 2
```

The document explicitly states that normal `export` does not survive between steps and that `$GITHUB_ENV` is used to persist state within a job. 03 Advanced Techniques and Best…

---

## LAB 3 — Failure Handling

Problem:

```text
Deployment fails
        |
        v
Pipeline stops
```

Solution:

```text
Deployment fails
       |
       v
failure()
       |
       v
Rollback
```

---

## LAB 4 — Enterprise Self-Healing

Now:

```text
Reusable Workflow
       +
$GITHUB_ENV
       +
Failure Handling
       +
Rollback
       +
Security
       |
       v
Enterprise Pipeline
```

This is the whole point of Section 7. 03 Advanced Techniques and Best…

---

# PART 43 — The Enterprise Architecture

This is the architecture I would draw on the board while teaching:

```text
                    DEVELOPER
                        |
                        | git push
                        v
                +---------------+
                | GitHub Repo    |
                +---------------+
                        |
                        v
          +---------------------------+
          | Enterprise Workflow       |
          | enterprise-deploy.yml     |
          +---------------------------+
                        |
                        v
             +-------------------+
             | SECURITY          |
             | Least Privilege   |
             | SHA Pinning       |
             +-------------------+
                        |
                        v
             +-------------------+
             | REUSABLE BUILD    |
             | gha-lab-01-central|
             +-------------------+
                        |
                        v
                BUILD + TEST
                        |
                        v
                  BUILD SUCCESS
                        |
                        v
             +-------------------+
             | RELEASE TAG       |
             | v1.0.X            |
             +-------------------+
                        |
                        v
                  $GITHUB_ENV
                        |
                        v
                PRODUCTION DEPLOY
                        |
                 +------+------+
                 |             |
              SUCCESS        FAILURE
                 |             |
                 v             v
              LIVE APP      failure()
                               |
                    +----------+----------+
                    |                     |
                    v                     v
                ROLLBACK                ALERT
                    |                     |
                    +----------+----------+
                               |
                               v
                       PRODUCTION STABLE
```

This matches the end-to-end lifecycle described in the document: developer push, reusable build, security, dynamic release state, deployment failure, rollback and alert. 03 Advanced Techniques and Best…

---

# PART 44 — Security Concepts Completed

There are two security practices from the document that this lab demonstrates.

## 1. Least Privilege

```yaml
permissions:
  contents: read
  deployments: write
```

Don't give the pipeline more access than it needs.

---

## 2. SHA Pinning

Instead of:

```yaml
uses: actions/checkout@v4
```

we use:

```yaml
uses: actions/checkout@1d96c772d19495a3b5c517cd2bc0cb401ea0529f
```

The document explicitly explains that tags can potentially be moved or overwritten, whereas a commit SHA identifies a specific immutable commit. 03 Advanced Techniques and Best…

---

# PART 45 — One Important Real-World Limitation

Our lab prints:

```text
Sending critical failure alert to Slack...
```

It does **not** actually send a Slack message.

That is intentional for this lab.

The document's conceptual flow uses Slack as the communication mechanism, but the Section 7 code demonstrates the alert using an `echo` command rather than a real Slack integration. 03 Advanced Techniques and Best…

So students should understand:

```text
LAB
 |
 +-- Demonstrates alert trigger
 |
 +-- Production
       |
       +-- Could connect to Slack
       +-- Email
       +-- PagerDuty
       +-- Incident system
```

We are teaching the **pipeline control flow**, not building a Slack integration.

---

# PART 46 — Final Copy/Paste Command Sequence

For your students, the complete command sequence is:

```bash
cd /p/Scaler/DevOpsTool_2/GitHubAction
```

```bash
git clone https://github.com/discover-devops/gha-lab-04-enterprise-self-healing.git
```

```bash
cd /p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-04-enterprise-self-healing
```

```bash
pwd
```

Expected:

```text
/p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-04-enterprise-self-healing
```

Create application files:

```bash
notepad package.json
```

```bash
notepad test.js
```

Generate lock file:

```bash
npm install
```

Test:

```bash
npm test
```

Create scripts:

```bash
mkdir scripts
```

Create deployment script:

```bash
notepad scripts/deploy.sh
```

Make executable:

```bash
chmod +x scripts/deploy.sh
```

Create rollback:

```bash
notepad scripts/safe-rollback.sh
```

Make executable:

```bash
chmod +x scripts/safe-rollback.sh
```

Create workflow:

```bash
mkdir -p .github/workflows
```

```bash
notepad .github/workflows/enterprise-deploy.yml
```

Then:

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
git commit -m "Build enterprise self-healing pipeline"
```

```bash
git push -u origin main
```

Then:

```text
GitHub
  ↓
Actions
  ↓
Enterprise Self-Healing Deployment
  ↓
Open workflow run
  ↓
Observe Build
  ↓
Observe Deployment
  ↓
Observe Failure
  ↓
Observe Rollback
  ↓
Observe Alert
```

---

# PART 47 — What Students Learned

At the end of LAB 4, students should be able to explain:

### 1. Why reusable workflows?

To avoid duplicating the same pipeline logic across repositories.

```text
Write Once
    ↓
Reuse Everywhere
```

### 2. Why `$GITHUB_ENV`?

To pass dynamic environment variables between subsequent steps in the same job.

```text
Generate Release
      ↓
$GITHUB_ENV
      ↓
Deploy
```

### 3. Why `needs`?

To establish job dependency:

```text
Build
  ↓
Deploy
```

### 4. Why `permissions`?

To implement least privilege.

```text
Only grant required permissions.
```

### 5. Why SHA pinning?

To pin third-party Actions to a specific commit rather than trusting a movable version tag. 03 Advanced Techniques and Best…

### 6. Why `failure()`?

To execute recovery logic when something fails.

```text
Failure
   ↓
Rollback
```

### 7. Why rollback?

To restore the application to a stable state automatically rather than waiting for a human.

### 8. What makes it self-healing?

The pipeline detects failure and automatically performs recovery and communication.

---

# FINAL TAKEAWAY

Your students should leave this entire section with this mental model:

```text
                 DEVOPS ENGINEER
                       |
                       | Push
                       v
                GITHUB ACTIONS
                       |
          +------------+------------+
          |                         |
          v                         v
     SECURITY                  REUSABILITY
   Least Privilege           Reusable Workflow
   SHA Pinning                     |
          |                         |
          +------------+------------+
                       |
                       v
                  BUILD / TEST
                       |
                       v
                DYNAMIC VERSION
                       |
                       v
                  $GITHUB_ENV
                       |
                       v
                   DEPLOY
                       |
              +--------+--------+
              |                 |
           SUCCESS            FAILURE
              |                 |
              v                 v
           LIVE APP          failure()
                                |
                         +------+------+
                         |             |
                         v             v
                     ROLLBACK        ALERT
                         |
                         v
                  STABLE SYSTEM
```

