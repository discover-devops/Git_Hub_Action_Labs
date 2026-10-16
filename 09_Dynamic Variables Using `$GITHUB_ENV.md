
# LAB 2 — Dynamic Variables Using `$GITHUB_ENV`

## What are we going to learn?

In this lab, students will learn how to:

1. Create a GitHub repository from scratch.
2. Clone it using Git Bash.
3. Create a GitHub Actions workflow.
4. Understand why normal Linux `export` does **not** persist between GitHub Actions steps.
5. Use `$GITHUB_ENV` to pass a dynamically generated variable from one step to the next.
6. Use GitHub's `${{ github.run_number }}` to generate a dynamic version.
7. Commit and push the workflow using Git Bash.
8. Run the workflow manually.
9. Verify the result in GitHub Actions.

The core concept from your document is:

```text
Step 1
   |
   | export MY_VERSION=...
   |
   X
   |
Step 2
```

The variable disappears because the next step runs in a separate shell.

With `$GITHUB_ENV`:

```text
Step 1
   |
   | echo "MY_VERSION=..." >> $GITHUB_ENV
   |
   v
$GITHUB_ENV
   |
   v
Step 2
   |
   | echo $MY_VERSION
   |
   v
Variable available
```

This is exactly the problem and solution described in the source material. 03 Advanced Techniques and Best…

---

# PART 1 — Lab Architecture

We will create **one GitHub repository**.

Repository name:

```text
gha-lab-02-github-env
```

Our local directory will be:

```text
/p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-02-github-env
```

The final structure will be:

```text
/p/Scaler/DevOpsTool_2/GitHubAction/
│
├── gha-lab-01-central/
│
├── gha-lab-01-microservice/
│
└── gha-lab-02-github-env/
    │
    └── .github/
        └── workflows/
            └── github-env.yml
```

**Important:** We are NOT creating another parent directory.

Your existing directory:

```text
/p/Scaler/DevOpsTool_2/GitHubAction
```

is our working directory.

---

# PART 2 — Create the GitHub Repository

## Step 1 — Open GitHub

Open GitHub in your browser and sign in using your account:

```text
discover-devops
```

---

## Step 2 — Create a new repository

Click:

**+ → New repository**

Enter:

### Repository name

```text
gha-lab-02-github-env
```

### Description

```text
GitHub Actions GITHUB_ENV demonstration
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

We want an empty repository.

Click:

**Create repository**

---

# PART 3 — Open Git Bash

Open:

**Git Bash**

---

# Step 3 — Check your current directory

Run:

```bash
pwd
```

Your expected directory is:

```text
/p/Scaler/DevOpsTool_2/GitHubAction
```

If you are somewhere else, run:

```bash
cd /p/Scaler/DevOpsTool_2/GitHubAction
```

Now verify:

```bash
pwd
```

Expected:

```text
/p/Scaler/DevOpsTool_2/GitHubAction
```

---

# PART 4 — Clone the Repository

## Step 4 — Clone the GitHub repository

Run:

```bash
git clone https://github.com/discover-devops/gha-lab-02-github-env.git
```

You should see something similar to:

```text
Cloning into 'gha-lab-02-github-env'...
```

---

# Step 5 — Verify the directory

Run:

```bash
ls
```

You should now see:

```text
gha-lab-01-central
gha-lab-01-microservice
gha-lab-02-github-env
```

The first two directories are from LAB 1.

The new directory is:

```text
gha-lab-02-github-env
```

---

# PART 5 — Enter the Lab 2 Directory

## Step 6 — Go into the repository

Run:

```bash
cd /p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-02-github-env
```

Verify:

```bash
pwd
```

Expected:

```text
/p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-02-github-env
```

---

# Step 7 — Check the repository

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

The repository is currently empty apart from Git's internal `.git` directory.

---

# PART 6 — Create the Workflow Directory

We now need to create:

```text
.github/workflows
```

## Step 8 — Create the directories

Run:

```bash
mkdir -p .github/workflows
```

---

# Step 9 — Verify the directory

Run:

```bash
ls -la
```

You should now see:

```text
.github
.git
```

Now run:

```bash
ls .github
```

Expected:

```text
workflows
```

Now run:

```bash
ls .github/workflows
```

There should be no output because the directory is currently empty.

---

# PART 7 — First Understand the Problem

Before creating the final workflow, let's understand **why `$GITHUB_ENV` is required**.

The document gives this example:

```yaml
- name: Set Version
  run: export MY_VERSION="v1.5.0"

- name: Print Version
  run: echo "The version is $MY_VERSION"
```

The problem is that the second step does not receive the variable from the first step because the steps run in separate shell processes. 03 Advanced Techniques and Best…

We will actually demonstrate this.

---

# PART 8 — Create the Demonstration Workflow

## Step 10 — Create `github-env.yml`

Make sure you are still here:

```text
/p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-02-github-env
```

Run:

```bash
notepad .github/workflows/github-env.yml
```

Windows Notepad will open.

If Windows asks:

> Do you want to create a new file?

Click:

**Yes**

---

# Step 11 — Paste this workflow

Paste the following **exactly**:

```yaml
name: GITHUB_ENV Lab

on:
  workflow_dispatch:

jobs:
  environment-demo:
    runs-on: ubuntu-latest

    steps:

      - name: Demonstrate the Problem
        run: |
          export MY_VERSION="v1.5.0"
          echo "Step 1: MY_VERSION is $MY_VERSION"

      - name: Show Variable Does Not Persist
        run: |
          echo "Step 2: MY_VERSION is '$MY_VERSION'"

      - name: Generate Dynamic Version
        run: |
          DYNAMIC_VER="v2.0.${{ github.run_number }}"
          echo "MY_VERSION=$DYNAMIC_VER" >> $GITHUB_ENV
          echo "Generated version: $DYNAMIC_VER"

      - name: Use Version
        run: |
          echo "Deploying version $MY_VERSION"

      - name: Verify Version
        run: |
          echo "Final version is: $MY_VERSION"
```

Save:

**Ctrl + S**

Then close Notepad.

---

# PART 9 — Understand the Workflow

Let's break this into two parts.

## Part A — The wrong approach

We have:

```yaml
- name: Demonstrate the Problem
  run: |
    export MY_VERSION="v1.5.0"
    echo "Step 1: MY_VERSION is $MY_VERSION"
```

Here we create:

```text
MY_VERSION=v1.5.0
```

The first step can see it.

Then the next step runs:

```yaml
- name: Show Variable Does Not Persist
  run: |
    echo "Step 2: MY_VERSION is '$MY_VERSION'"
```

The variable is gone.

Why?

Because:

```text
Step 1
   |
   | Shell Process 1
   |
   | MY_VERSION exists
   |
   X
   |
   | Shell Process 1 destroyed
   |
   v
Step 2
   |
   | New Shell Process
   |
   | MY_VERSION does not exist
```

This is the exact concept explained in your document. 03 Advanced Techniques and Best…

---

# PART 10 — The Correct Approach

Now we use:

```yaml
echo "MY_VERSION=$DYNAMIC_VER" >> $GITHUB_ENV
```

This is the important line of the lab.

We first generate a dynamic version:

```yaml
DYNAMIC_VER="v2.0.${{ github.run_number }}"
```

Suppose this is GitHub Actions run number:

```text
1
```

Then:

```text
DYNAMIC_VER
```

becomes:

```text
v2.0.1
```

If the next run is number 2:

```text
v2.0.2
```

If the next run is number 3:

```text
v2.0.3
```

So GitHub is helping us generate a unique version for each workflow run.

---

# PART 11 — What Does `$GITHUB_ENV` Do?

This line:

```bash
echo "MY_VERSION=$DYNAMIC_VER" >> $GITHUB_ENV
```

writes:

```text
MY_VERSION=v2.0.1
```

into GitHub's special environment file.

Then subsequent steps can access:

```bash
$MY_VERSION
```

So:

```text
Step 1
   |
   | Generate version
   |
   v
DYNAMIC_VER=v2.0.1
   |
   |
   v
$GITHUB_ENV
   |
   |
   v
MY_VERSION=v2.0.1
   |
   |
   v
Step 2
   |
   | $MY_VERSION
   |
   v
v2.0.1
```

The source document explicitly describes `$GITHUB_ENV` as the mechanism for passing variables between steps. 03 Advanced Techniques and Best…

---

# PART 12 — Verify the Workflow File

Go back to Git Bash.

Run:

```bash
ls .github/workflows
```

Expected:

```text
github-env.yml
```

Now run:

```bash
cat .github/workflows/github-env.yml
```

You should see the complete YAML you just created.

---

# PART 13 — Check Git Status

Run:

```bash
git status
```

You should see:

```text
Untracked files:
  .github/
```

---

# PART 14 — Add the Workflow

Run:

```bash
git add .
```

There should be no error.

---

# Step 15 — Check status

Run:

```bash
git status
```

You should now see something similar to:

```text
Changes to be committed:
  new file: .github/workflows/github-env.yml
```

---

# PART 15 — Commit the Workflow

## Step 16 — Commit

Run:

```bash
git commit -m "Add GITHUB_ENV demonstration"
```

You should see something similar to:

```text
[main xxxxxxx] Add GITHUB_ENV demonstration
 1 file changed
```

---

# PART 16 — Push to GitHub

## Step 17 — Push

Run:

```bash
git push -u origin main
```

You should see something similar to:

```text
[new branch] main -> main
```

---

# PART 17 — Verify on GitHub

Go to your GitHub repository:

```text
gha-lab-02-github-env
```

You should see:

```text
.github
```

Open:

```text
.github
```

Then:

```text
workflows
```

You should see:

```text
github-env.yml
```

---

# PART 18 — Run the GitHub Actions Workflow

Now we will actually execute the lab.

On GitHub, open:

**gha-lab-02-github-env**

Then click:

**Actions**

You should see:

```text
GITHUB_ENV Lab
```

Click:

**GITHUB_ENV Lab**

You should see a button:

**Run workflow**

Click:

**Run workflow**

Then click the green:

**Run workflow**

button.

---

# PART 19 — Watch the Workflow

A workflow run will appear.

Click the workflow run.

Then click:

```text
environment-demo
```

You will see the individual steps.

They should look approximately like:

```text
✓ Demonstrate the Problem
✓ Show Variable Does Not Persist
✓ Generate Dynamic Version
✓ Use Version
✓ Verify Version
```

---

# PART 20 — Examine the First Step

Click:

**Demonstrate the Problem**

You should see something similar to:

```text
Step 1: MY_VERSION is v1.5.0
```

This proves that the variable exists inside that step.

---

# PART 21 — Examine the Second Step

Click:

**Show Variable Does Not Persist**

You should see:

```text
Step 2: MY_VERSION is ''
```

or effectively an empty value.

This is the important demonstration.

The variable was created using:

```bash
export MY_VERSION="v1.5.0"
```

but it did not survive into the next step.

---

# PART 22 — Examine the Third Step

Click:

**Generate Dynamic Version**

You should see something similar to:

```text
Generated version: v2.0.1
```

The exact number depends on the GitHub Actions run number.

For example:

```text
v2.0.1
```

or:

```text
v2.0.2
```

or:

```text
v2.0.3
```

---

# PART 23 — Examine the Fourth Step

Click:

**Use Version**

You should see:

```text
Deploying version v2.0.1
```

This proves that:

```text
Step 3
```

successfully passed the variable to:

```text
Step 4
```

using:

```text
$GITHUB_ENV
```

---

# PART 24 — Examine the Fifth Step

Click:

**Verify Version**

You should see:

```text
Final version is: v2.0.1
```

Again, the exact number will depend on the workflow run number.

---

# PART 25 — The Complete Flow

Now explain this to the students:

```text
GitHub Actions Workflow
        |
        v
Step 1
export MY_VERSION=v1.5.0
        |
        X
        |
        | Variable lost
        v
Step 2
MY_VERSION = empty
```

Then:

```text
GitHub Actions Workflow
        |
        v
Generate Dynamic Version
        |
        v
DYNAMIC_VER=v2.0.1
        |
        v
$GITHUB_ENV
        |
        v
MY_VERSION=v2.0.1
        |
        v
Use Version
        |
        v
Deploying version v2.0.1
        |
        v
Verify Version
        |
        v
Final version is v2.0.1
```

---

# PART 26 — Why Is This Useful in Real DevOps?

Give students this scenario.

Imagine we have a pipeline:

```text
Build
  |
  v
Generate Version
  |
  v
Deploy
```

The Build step generates:

```text
v2.0.25
```

The Deploy step needs to know:

```text
Which version should I deploy?
```

We need to pass:

```text
v2.0.25
```

from one step to another.

That is where `$GITHUB_ENV` helps.

Real-world flow:

```text
Build Application
       |
       v
Generate Release Version
       |
       | APP_VERSION=v2.0.25
       v
$GITHUB_ENV
       |
       v
Deployment Step
       |
       v
Deploy v2.0.25
```

The source material uses this same pattern for dynamic version information moving between build/deploy steps. 03 Advanced Techniques and Best…

---

# PART 27 — Important Distinction

Students should remember:

### `export`

```bash
export MY_VERSION="v1.5.0"
```

means:

> Make the variable available in the current shell/process.

It does **not** automatically make the variable available to the next GitHub Actions step.

---

### `$GITHUB_ENV`

```bash
echo "MY_VERSION=v1.5.0" >> $GITHUB_ENV
```

means:

> Tell GitHub Actions to make this environment variable available to subsequent steps in the job.

---

# PART 28 — Final Directory Structure

Your complete local structure should now be:

```text
/p/Scaler/DevOpsTool_2/GitHubAction/
│
├── gha-lab-01-central/
│   ├── .github/
│   │   └── workflows/
│   │       └── reusable-build.yml
│   ├── package.json
│   ├── package-lock.json
│   └── test.js
│
├── gha-lab-01-microservice/
│   ├── .github/
│   │   └── workflows/
│   │       └── microservice-a.yml
│   ├── package.json
│   ├── package-lock.json
│   └── test.js
│
└── gha-lab-02-github-env/
    │
    └── .github/
        └── workflows/
            └── github-env.yml
```

For **Lab 2**, there is only one new directory:

```text
gha-lab-02-github-env
```

And only one file:

```text
.github/workflows/github-env.yml
```

---

# PART 29 — Complete Copy/Paste Command Sequence

For students who just want the commands in order:

### Start

```bash
cd /p/Scaler/DevOpsTool_2/GitHubAction
```

### Clone repository

```bash
git clone https://github.com/discover-devops/gha-lab-02-github-env.git
```

### Enter repository

```bash
cd /p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-02-github-env
```

### Verify

```bash
pwd
```

Expected:

```text
/p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-02-github-env
```

### Create workflow directory

```bash
mkdir -p .github/workflows
```

### Create workflow

```bash
notepad .github/workflows/github-env.yml
```

Paste the complete YAML from above.

### Verify

```bash
ls .github/workflows
```

Expected:

```text
github-env.yml
```

### Git

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
git commit -m "Add GITHUB_ENV demonstration"
```

```bash
git push -u origin main
```

### Run

GitHub:

```text
Repository
   ↓
Actions
   ↓
GITHUB_ENV Lab
   ↓
Run workflow
   ↓
Run workflow
```

---

# PART 30 — What Students Learned

At the end of the lab, students should be able to answer:

### Question 1
Why doesn't this work across two steps?

```bash
export MY_VERSION="v1.5.0"
```

Because each GitHub Actions step runs in a separate shell process. 03 Advanced Techniques and Best…

### Question 2
How do we persist an environment variable for subsequent steps?

```bash
echo "MY_VERSION=value" >> $GITHUB_ENV
```

### Question 3
How did we create a dynamic version?

```bash
DYNAMIC_VER="v2.0.${{ github.run_number }}"
```

### Question 4
How did the next step access it?

```bash
echo "$MY_VERSION"
```

### Question 5
What is the key difference?

```text
export
  ↓
Current shell

$GITHUB_ENV
  ↓
Subsequent steps in the job
```

---

# The one-line takeaway for students

> **Use `$GITHUB_ENV` when one GitHub Actions step needs to pass an environment variable to a later step in the same job.**

This completes **LAB 2** and stays aligned with the source document's `$GITHUB_ENV` hands-on exercise. 03 Advanced Techniques and Best…
