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

# Lab 6 --- Generate a Version and Pass It to Deployment

## Goal

Generate a version once and use it:

1.  In a later step of the same job.
2.  In another job.
3.  In the reusable deployment workflow.

## What you learn

``` text
Between steps → $GITHUB_ENV
Between jobs  → $GITHUB_OUTPUT + job outputs
```

You also learn why a normal shell `export` does not survive into the
next GitHub Actions step.

------------------------------------------------------------------------

# Part 1 --- Demonstrate why `export` does not work

Make sure you are on `development`.

Create the workflow directory if necessary:

``` bash
mkdir -p .github/workflows
```

Create:

``` bash
cat > .github/workflows/env-demo.yml << 'EOF'
name: Env Demo

on:
  push:
    branches: [ development ]

jobs:
  demo:
    runs-on: ubuntu-latest

    steps:
      - name: Try export
        run: export MY_VERSION=1.0.0

      - name: Read it in the next step
        run: echo "Version is [$MY_VERSION]"

      - name: Use GITHUB_ENV instead
        run: echo "MY_VERSION=1.0.0" >> "$GITHUB_ENV"

      - name: Read it in the next step again
        run: echo "Version is [$MY_VERSION]"
EOF
```

Commit and push:

``` bash
git add .
git commit -m "Lab 6: env demo"
git push origin development
```

------------------------------------------------------------------------

# Part 2 --- Verify the result

Go to:

**Actions → Env Demo**

Open the job and expand the steps.

Expected:

``` text
Version is []
```

for the first read.

Then:

``` text
Version is [1.0.0]
```

for the second read.

## Why?

This:

``` bash
export MY_VERSION=1.0.0
```

only affects that shell.

The next `run:` step starts a new shell.

This:

``` bash
echo "MY_VERSION=1.0.0" >> "$GITHUB_ENV"
```

tells GitHub Actions to make the variable available to later steps in
the same job.

------------------------------------------------------------------------

# Part 3 --- Remove the demo workflow

Run:

``` bash
git rm .github/workflows/env-demo.yml
```

------------------------------------------------------------------------

# Part 4 --- Replace Main Pipeline

Create the final Day 1 pipeline:

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

    outputs:
      version: ${{ steps.ver.outputs.version }}

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Generate version
        id: ver
        run: |
          VERSION="1.0.${{ github.run_number }}-${GITHUB_SHA::7}"
          echo "APP_VERSION=$VERSION" >> "$GITHUB_ENV"
          echo "version=$VERSION" >> "$GITHUB_OUTPUT"

      - name: Use the version in a later step
        run: echo "Building version $APP_VERSION"

      - run: npm run build
      - run: npm test

  deploy-dev:
    needs: build
    if: github.ref == 'refs/heads/development'

    uses: ./.github/workflows/deploy-reusable.yml

    with:
      environment: development
      app_version: ${{ needs.build.outputs.version }}

    secrets:
      deploy_token: ${{ secrets.DEPLOY_TOKEN }}

  deploy-prod:
    needs: build
    if: github.ref == 'refs/heads/main'

    uses: ./.github/workflows/deploy-reusable.yml

    with:
      environment: production
      app_version: ${{ needs.build.outputs.version }}

    secrets:
      deploy_token: ${{ secrets.DEPLOY_TOKEN }}
EOF
```

------------------------------------------------------------------------

# Part 5 --- Commit and push

``` bash
git add .
git commit -m "Lab 6: generate version and pass it to deploy"
git push origin development
```

------------------------------------------------------------------------

# Part 6 --- Verify Development

Go to:

**Actions → Main Pipeline**

Open the latest run.

The build job should show something like:

``` text
Building version 1.0.5-a1b2c3d
```

The exact number and commit value will be different for your run.

Then `deploy-dev` should show the **same version**:

``` text
Deploying version 1.0.5-a1b2c3d
```

The important point is:

``` text
BUILD VERSION
     =
DEPLOY VERSION
```

------------------------------------------------------------------------

# Part 7 --- Verify Production

Create a PR:

``` text
base:    main
compare: development
```

Merge it.

Approve the production deployment when requested.

The production deployment should receive the version generated by the
build job.

------------------------------------------------------------------------

# Teaching explanation

There are two different mechanisms.

## Between steps

Use:

``` text
$GITHUB_ENV
```

Example:

``` bash
echo "APP_VERSION=$VERSION" >> "$GITHUB_ENV"
```

Later step:

``` bash
echo "$APP_VERSION"
```

## Between jobs

Use:

``` text
$GITHUB_OUTPUT
```

First give the step an ID:

``` yaml
id: ver
```

Write the output:

``` bash
echo "version=$VERSION" >> "$GITHUB_OUTPUT"
```

Expose it as a job output:

``` yaml
outputs:
  version: ${{ steps.ver.outputs.version }}
```

Then another job reads:

``` yaml
${{ needs.build.outputs.version }}
```

The mental model:

``` text
Same Job

Step 1
  |
  | GITHUB_ENV
  v
Step 2
  |
  v
Step 3
```

Across jobs:

``` text
BUILD JOB
    |
    | GITHUB_OUTPUT
    v
Job Output
    |
    | needs.build.outputs.version
    v
DEPLOY JOB
```

## Lab 6 learning checkpoint

You should be able to answer:

> Why doesn't `export` survive into the next step?

> When do we use `$GITHUB_ENV`?

> When do we use `$GITHUB_OUTPUT`?

> Why do we need `id: ver`?

> How does `needs.build.outputs.version` get the value?
