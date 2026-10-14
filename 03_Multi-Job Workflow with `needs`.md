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

# Lab 3 --- Multi-Job Workflow with `needs`

## Goal

Build, test, and package using separate jobs.

Then control the execution order with `needs`.

## What you learn

-   Jobs run in parallel by default.
-   `needs` creates a dependency.
-   Each job gets its own fresh runner.
-   Each job that needs repository code must run checkout.
-   Artifacts are used to move files between jobs.

------------------------------------------------------------------------

# Part 1 --- Create the first version WITHOUT `needs`

Remove Lab 2:

``` bash
git rm .github/workflows/events-demo.yml
mkdir -p .github/workflows
```

Create the workflow:

``` bash
cat > .github/workflows/ci-multi-job.yml << 'EOF'
name: CI Multi Job

on:
  push:
    branches: [ main, development ]

  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Build the app
        run: npm run build

      - name: Upload the build output
        uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Run unit tests
        run: npm test

  package:
    runs-on: ubuntu-latest

    steps:
      - name: Download the build output
        uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist

      - name: Show what we packaged
        run: ls -R dist
EOF
```

Commit and push:

``` bash
git add .
git commit -m "Lab 3: multi job without needs"
git push origin main
```

------------------------------------------------------------------------

# Part 2 --- Observe the problem

Go to:

**GitHub → Actions → CI Multi Job**

Open the run.

Look at the workflow graph.

You should see:

``` text
build
test
package
```

starting independently.

The `package` job can fail with:

``` text
Artifact not found
```

## Why?

`build` creates and uploads the artifact:

``` text
dist
```

But `package` is trying to download it without waiting for `build`.

So the problem is not that the artifact definition is wrong.

The problem is:

``` text
package starts too early
```

------------------------------------------------------------------------

# Part 3 --- Fix the order using `needs`

Edit:

``` text
.github/workflows/ci-multi-job.yml
```

Change the `test` job to:

``` yaml
  test:
    needs: build
    runs-on: ubuntu-latest
```

Change the `package` job to:

``` yaml
  package:
    needs: [build, test]
    runs-on: ubuntu-latest
```

Commit and push:

``` bash
git add .
git commit -m "Lab 3: add needs"
git push origin main
```

## Expected graph

``` text
build
  |
  v
test
  |
  v
package
```

All three jobs should succeed.

------------------------------------------------------------------------

# Part 4 --- Intentionally break checkout

This is an important teaching experiment.

In the `test` job, temporarily remove:

``` yaml
- uses: actions/checkout@v4
```

Push the change:

``` bash
git add .
git commit -m "Lab 3: test missing checkout"
git push origin main
```

Open the failed test job.

You should see an error indicating that `package.json` cannot be found.

## Why?

Every job gets its own fresh runner.

The checkout performed in `build` does not automatically affect `test`.

------------------------------------------------------------------------

# Part 5 --- Fix checkout again

Put this back into the `test` job:

``` yaml
- uses: actions/checkout@v4
```

Then:

``` bash
git add .
git commit -m "Lab 3: restore checkout"
git push origin main
```

Verify the final run.

------------------------------------------------------------------------

# Teaching explanation

Think of jobs as separate computers:

``` text
BUILD RUNNER
    |
    +-- checkout
    +-- npm run build
    +-- upload dist
              |
              v
        GitHub Artifact
              |
              v
PACKAGE RUNNER
    |
    +-- download dist
    +-- package
```

`needs` controls **when** a job starts.

Artifacts control **how files move between jobs**.

Checkout gives each job access to repository code.

## Lab 3 learning checkpoint

You should be able to explain:

> Why did `package` fail without `needs`?

> Why does every job need its own checkout?

> What is the difference between `needs` and an artifact?
