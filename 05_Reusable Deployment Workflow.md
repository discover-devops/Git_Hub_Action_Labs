# GitHub Actions Day 1 --- Student Runbook

## Lab sequence

1.  Lab 1 --- First Workflow
2.  Lab 2 --- Events and Triggers
3.  Lab 3 --- Multi-Job Workflow with `needs`
4.  Lab 4 --- Branch-Based Environments
5.  Lab 5 --- Reusable Deployment Workflow
6.  Lab 6 --- Dynamic Version Passing

## Working environment

These labs are designed to be run from a terminal such as WSL Ubuntu or
Git Bash.

Use your own GitHub repository and username wherever the runbook says
`YOUR-USERNAME`.

Important: - Run terminal commands from the repository root. - Workflow
files must be under `.github/workflows/`. - YAML indentation uses
spaces, not tabs. - After changing a workflow, always `git add`,
`git commit`, and `git push`.

# Lab 5 --- Reusable Deployment Workflow

## Goal

Write the deployment logic once, then reuse it for Development and
Production.

## What you learn

-   `workflow_call`
-   Reusable workflows
-   Inputs
-   Secrets
-   Calling a reusable workflow with `uses:`
-   Reusing the same deployment logic for multiple environments

------------------------------------------------------------------------

# Part 1 --- Create the repository secret

Go to:

**Repository → Settings → Secrets and variables → Actions**

Under **Repository secrets**, click:

**New repository secret**

Enter:

``` text
Name:  DEPLOY_TOKEN
Value: demo-token-12345
```

Click **Add secret**.

This is a dummy learning token.

------------------------------------------------------------------------

# Part 2 --- Clean up Lab 3 and Lab 4

Run:

``` bash
git checkout development
git pull origin development
```

Remove the old workflows:

``` bash
git rm .github/workflows/branch-deploy.yml .github/workflows/ci-multi-job.yml
```

Important: after removing the last workflow files, Git may no longer
have a local `.github/workflows` directory.

Therefore recreate it before using `cat`:

``` bash
mkdir -p .github/workflows
```

------------------------------------------------------------------------

# Part 3 --- Create the reusable deployment workflow

Run:

``` bash
cat > .github/workflows/deploy-reusable.yml << 'EOF'
name: Reusable Deploy

on:
  workflow_call:
    inputs:
      environment:
        description: 'Target environment name'
        required: true
        type: string

      app_version:
        description: 'Version being deployed'
        required: false
        type: string
        default: '0.0.0'

    secrets:
      deploy_token:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}

    steps:
      - name: Deploy
        env:
          DEPLOY_TOKEN: ${{ secrets.deploy_token }}

        run: |
          echo "Deploying version ${{ inputs.app_version }}"
          echo "Target environment: ${{ inputs.environment }}"
          echo "Token length: ${#DEPLOY_TOKEN} characters"
          echo "Printing the token directly: $DEPLOY_TOKEN"
EOF
```

------------------------------------------------------------------------

# Part 4 --- Create the Main Pipeline

Run:

``` bash
cat > .github/workflows/main-pipeline.yml << 'EOF'
name: Main Pipeline

on:
  push:
    branches: [ development, main ]

  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - run: npm run build
      - run: npm test

  deploy-dev:
    needs: build
    if: github.ref == 'refs/heads/development'

    uses: ./.github/workflows/deploy-reusable.yml

    with:
      environment: development

    secrets:
      deploy_token: ${{ secrets.DEPLOY_TOKEN }}

  deploy-prod:
    needs: build
    if: github.ref == 'refs/heads/main'

    uses: ./.github/workflows/deploy-reusable.yml

    with:
      environment: production

    secrets:
      deploy_token: ${{ secrets.DEPLOY_TOKEN }}
EOF
```

------------------------------------------------------------------------

# Part 5 --- Commit and push

``` bash
git add .
git commit -m "Lab 5: reusable deploy workflow"
git push origin development
```

------------------------------------------------------------------------

# Part 6 --- Verify Development

Go to:

**GitHub → Actions → Main Pipeline**

Open the latest run.

Expected:

``` text
build          ✓
deploy-dev     ✓
deploy-prod    skipped
```

The deployment log should show:

``` text
Deploying version 0.0.0
Target environment: development
Token length: 16 characters
Printing the token directly: ***
```

GitHub masks the secret when it is printed.

------------------------------------------------------------------------

# Part 7 --- Promote to Production

Create a PR:

``` text
base:    main
compare: development
```

Merge it.

Go to:

**Actions → Main Pipeline**

Production should wait for approval because the `production` environment
has required reviewers.

Click:

**Review deployments → production → Approve and deploy**

Expected:

``` text
Deploying version 0.0.0
Target environment: production
Token length: 16 characters
Printing the token directly: ***
```

------------------------------------------------------------------------

# Teaching explanation

The reusable workflow is:

``` text
deploy-reusable.yml
```

It behaves like a function.

``` text
Caller
   |
   | environment = development
   | deploy_token = secret
   v
Reusable Deploy
```

or:

``` text
Caller
   |
   | environment = production
   | deploy_token = secret
   v
Reusable Deploy
```

### `workflow_call`

Turns the workflow into a reusable workflow.

### `inputs`

Parameters passed by the caller.

### `secrets`

Protected values passed by the caller.

### `uses`

Calls the reusable workflow.

Important:

A calling job uses:

``` yaml
uses: ./.github/workflows/deploy-reusable.yml
```

It does not define its own `runs-on` or `steps`.

## Lab 5 learning checkpoint

You should be able to explain:

> Why should deployment logic be reused instead of copied?

> What is `workflow_call`?

> What is the difference between an input and a secret?

> Why is `uses:` at the job level here?
