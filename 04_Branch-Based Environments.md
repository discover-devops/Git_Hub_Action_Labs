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

# Lab 4 --- Branch-Based Environments

## Goal

Deploy to a different GitHub Environment depending on the branch.

``` text
development branch → development environment
main branch        → production environment
```

## What you learn

-   GitHub Environments
-   Environment variables
-   Branch-based deployment
-   `if:` conditions
-   Production approval
-   Environment protection rules

------------------------------------------------------------------------

# Part 1 --- Create the development environment

Go to:

**Repository → Settings → Environments → New environment**

Create:

``` text
development
```

Configure it.

Under **Environment variables**, add:

``` text
Name:  APP_ENV
Value: DEVELOPMENT
```

Save.

------------------------------------------------------------------------

# Part 2 --- Create the production environment

Create another environment:

``` text
production
```

Under **Deployment protection rules**:

-   Enable **Required reviewers**
-   Add yourself as reviewer
-   Save the protection rules

Under **Environment variables**, add:

``` text
Name:  APP_ENV
Value: PRODUCTION
```

Save.

------------------------------------------------------------------------

# Part 3 --- Create the development branch

Run:

``` bash
git checkout main
git pull origin main
git checkout -b development
git push -u origin development
```

------------------------------------------------------------------------

# Part 4 --- Create the branch-based deployment workflow

Make sure the workflow directory exists:

``` bash
mkdir -p .github/workflows
```

Create:

``` bash
cat > .github/workflows/branch-deploy.yml << 'EOF'
name: Branch Based Deploy

on:
  push:
    branches: [ development, main ]

jobs:
  deploy-dev:
    if: github.ref == 'refs/heads/development'
    runs-on: ubuntu-latest
    environment: development

    steps:
      - name: Deploy to development
        run: |
          echo "Branch      : ${{ github.ref_name }}"
          echo "Environment : ${{ vars.APP_ENV }}"
          echo "Deploying to the DEVELOPMENT environment"

  deploy-prod:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production

    steps:
      - name: Deploy to production
        run: |
          echo "Branch      : ${{ github.ref_name }}"
          echo "Environment : ${{ vars.APP_ENV }}"
          echo "Deploying to the PRODUCTION environment"
EOF
```

Commit and push:

``` bash
git add .
git commit -m "Lab 4: branch based deploy"
git push origin development
```

------------------------------------------------------------------------

# Part 5 --- Verify development

Go to:

**GitHub → Actions → Branch Based Deploy**

Open the development run.

Expected:

``` text
deploy-dev     ✓
deploy-prod    skipped
```

The log should show:

``` text
Branch      : development
Environment : DEVELOPMENT
Deploying to the DEVELOPMENT environment
```

## Why was production skipped?

Because:

``` yaml
if: github.ref == 'refs/heads/main'
```

is false when the branch is `development`.

------------------------------------------------------------------------

# Part 6 --- Promote to production

Create a pull request:

``` text
base:    main
compare: development
```

Create and merge the PR.

After the merge, open:

**Actions → Branch Based Deploy**

The production job should wait for approval.

Click:

**Review deployments**

Select:

``` text
production
```

Then:

**Approve and deploy**

Expected production output:

``` text
Branch      : main
Environment : PRODUCTION
Deploying to the PRODUCTION environment
```

------------------------------------------------------------------------

# Teaching explanation

The workflow contains two jobs.

``` text
                    Branch Based Deploy
                           |
             +-------------+-------------+
             |                           |
       development                     main
             |                           |
       deploy-dev                   deploy-prod
             |                           |
     development env             production env
```

The `if:` condition decides which job is eligible.

The `environment:` connects the job to the GitHub Environment.

The Environment provides:

-   variables
-   secrets
-   protection rules
-   approval

## Lab 4 learning checkpoint

You should be able to explain:

> How does the same workflow deploy differently from development and
> main?

> What does `if:` do?

> What is the purpose of a GitHub Environment?

> How does production approval happen?
