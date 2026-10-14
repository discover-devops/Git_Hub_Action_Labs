# Git_Hub_Action_Labs
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

# Day 1 --- Student Runbook Pack

This folder contains one independent runbook for each Day 1 hands-on
lab.

  File                                     Lab
  ---------------------------------------- ---------------------------------
  `01_Lab1_First_Workflow.md`              First Workflow
  `02_Lab2_Events_and_Triggers.md`         Events and Triggers
  `03_Lab3_Multi_Job_and_Needs.md`         Multi-Job + `needs` + Artifacts
  `04_Lab4_Branch_Based_Environments.md`   Branch-Based Environments
  `05_Lab5_Reusable_Deployment.md`         Reusable Deployment Workflow
  `06_Lab6_Dynamic_Version_Passing.md`     Dynamic Version Passing

## Final Day 1 mental model

``` text
Git
  ↓
GitHub
  ↓
Event
  ↓
Workflow
  ↓
Job
  ↓
Runner
  ↓
Step
  ↓
Action / Command
```

Final deployment flow:

``` text
development branch → Development environment
main branch        → Production environment + approval
```

Final pipeline:

``` text
Main Pipeline
     |
   build
     |
     +----------------------+
     |                      |
 deploy-dev            deploy-prod
     |                      |
     +----------+-----------+
                |
       Reusable Deploy
                |
       Development / Production
```

## Day 1 core learning

-   Events trigger workflows.
-   Jobs are independent unless `needs` creates an order.
-   Each job gets a fresh runner.
-   Checkout brings repository code to a runner.
-   Artifacts move files between jobs.
-   Environments provide environment-specific configuration and
    protection.
-   Reusable workflows prevent deployment logic from being duplicated.
-   `$GITHUB_ENV` passes values between steps in one job.
-   `$GITHUB_OUTPUT` plus job outputs passes values between jobs.
