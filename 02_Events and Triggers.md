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

# Lab 2 --- Events and Triggers

## Goal

Trigger one workflow in four different ways:

1.  `push`
2.  `workflow_dispatch`
3.  `pull_request`
4.  `schedule`

## What you learn

A workflow can listen to multiple events.

You also learn that branch filters decide which branches can trigger a
workflow.

------------------------------------------------------------------------

# Step 1 --- Remove Lab 1

Run:

``` bash
git rm .github/workflows/hello.yml
```

Make sure the workflow directory exists:

``` bash
mkdir -p .github/workflows
```

# Step 2 --- Create the Events workflow

Run:

``` bash
cat > .github/workflows/events-demo.yml << 'EOF'
name: Events Demo

on:
  push:
    branches: [ main ]

  pull_request:
    branches: [ main ]

  workflow_dispatch:
    inputs:
      reason:
        description: 'Why are you running this?'
        required: false
        default: 'learning'

  schedule:
    - cron: '30 3 * * 1-5'

jobs:
  show-event:
    runs-on: ubuntu-latest

    steps:
      - name: Print event details
        run: |
          echo "Event name : ${{ github.event_name }}"
          echo "Branch/ref : ${{ github.ref }}"
          echo "Actor      : ${{ github.actor }}"
          echo "Reason     : ${{ inputs.reason }}"
EOF
```

# Step 3 --- Commit and push

``` bash
git add .
git commit -m "Lab 2: events demo"
git push origin main
```

------------------------------------------------------------------------

# Trigger 1 --- Push

The push above should automatically trigger the workflow.

Go to:

**GitHub → Actions → Events Demo**

Open the run.

Expected:

``` text
Event name : push
```

------------------------------------------------------------------------

# Trigger 2 --- Manual

Go to:

**Actions → Events Demo → Run workflow**

Enter a reason such as:

``` text
my first manual run
```

Click **Run workflow**.

Open the run.

Expected:

``` text
Event name : workflow_dispatch
Reason     : my first manual run
```

------------------------------------------------------------------------

# Trigger 3 --- Pull Request

Create a feature branch:

``` bash
git checkout -b feature-1
```

Make a small change:

``` bash
echo "// small change" >> app.js
```

Commit:

``` bash
git add .
git commit -m "Small change on feature-1"
git push -u origin feature-1
```

On GitHub:

1.  Click **Compare & pull request**.
2.  Create the pull request into `main`.
3.  Open the Checks/Actions result.
4.  Confirm the event is `pull_request`.

### Important

Do **not** merge the PR yet.

Return to main:

``` bash
git checkout main
```

------------------------------------------------------------------------

# Trigger 4 --- Schedule

The workflow contains:

``` yaml
schedule:
  - cron: '30 3 * * 1-5'
```

This means:

``` text
03:30 UTC
Monday-Friday
```

For India:

``` text
09:00 AM IST
```

Scheduled runs may be delayed by GitHub.

You do not need to wait for the schedule during the lab.

------------------------------------------------------------------------

# What you learned

``` text
push              → code pushed
workflow_dispatch → manual Run workflow button
pull_request      → PR activity
schedule          → time-based trigger
```

The branch filter:

``` yaml
branches: [ main ]
```

means pushes to other branches do not trigger the `push` event for this
workflow.

## Important rule

The **Run workflow** button and schedule work from the default branch.

## Common problems

### No Run workflow button

Make sure the workflow exists on the default branch (`main`).

### Push to feature-1 does not trigger the push event

That is expected because the workflow says:

``` yaml
branches: [ main ]
```

### Schedule does not start exactly at 09:00 IST

GitHub can queue scheduled workflows. Allow some delay.

## Lab 2 learning checkpoint

You should now be able to answer:

> What is the difference between `push`, `pull_request`,
> `workflow_dispatch`, and `schedule`?
