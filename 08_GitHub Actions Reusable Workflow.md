What you need :

- GitHub account
- Git Bash
- GitHub repository creation
- Basic copy/paste


The lab will create exactly **two directories** under this location, corresponding to the two GitHub repositories.

The lab follows the reusable-workflow concept from your uploaded document: create one central workflow and have another repository call it using `workflow_call`. 03 Advanced Techniques and Best…

---

# LAB 1 — GitHub Actions Reusable Workflow

## 1. What are we going to build?

We are going to create **two GitHub repositories**.

```text
GitHub
│
├── gha-lab-01-central
│   └── Reusable Build Workflow
│
└── gha-lab-01-microservice
    └── Calls the Reusable Workflow
```

The architecture is:

```text
Student Code
     |
     v
gha-lab-01-microservice
     |
     | calls
     v
gha-lab-01-central
     |
     v
reusable-build.yml
     |
     +---- Checkout code
     |
     +---- Setup Node.js
     |
     +---- npm ci
     |
     +---- npm test
```

The important concept from the document is:

> Instead of copying the same build logic into many repositories, create the workflow once in a central repository and let other repositories call it. 03 Advanced Techniques and Best…

---

# PART 1 — Before Starting

## 2. What you need

You need:

1. GitHub account
2. Git Bash
3. Internet connection
4. Git configured on Git Bash

Your GitHub username is:

```text
discover-devops
```

We will therefore use these repository names:

```text
gha-lab-01-central
gha-lab-01-microservice
```

---

# PART 2 — Create the GitHub Repositories

We will first create the two repositories on GitHub.

Do **not** create any files locally yet.

---

## 3. Create Repository 1

Open GitHub in your browser.

Go to:

```text
https://github.com
```

Log in using your GitHub account.

Click:

**+ → New repository**

Enter:

### Repository name

```text
gha-lab-01-central
```

### Description

You can enter:

```text
Central reusable GitHub Actions workflow
```

Choose:

```text
Public
```

For this training lab, Public is easier because another repository needs to call the workflow.

### IMPORTANT

Do **not** select:

```text
Add a README file
```

Do **not** add:

```text
.gitignore
```

Do **not** add:

```text
License
```

We want an empty repository.

Click:

**Create repository**

---

# 4. Create Repository 2

Again click:

**+ → New repository**

Create:

```text
gha-lab-01-microservice
```

Description:

```text
Microservice calling a reusable GitHub Actions workflow
```

Choose:

```text
Public
```

Again, leave these unchecked:

```text
☐ Add a README file
☐ Add .gitignore
☐ Choose a license
```

Click:

**Create repository**

---

# PART 3 — Verify the Two Repositories

You should now have:

```text
https://github.com/discover-devops/gha-lab-01-central
```

and

```text
https://github.com/discover-devops/gha-lab-01-microservice
```

At this point **both repositories should be empty**.

---

# PART 4 — Open Git Bash

Now open:

**Git Bash**

We will work from your existing directory:

```text
/p/Scaler/DevOpsTool_2/GitHubAction
```

---

# 5. Check your current directory

Run:

```bash
pwd
```

You should see:

```text
/p/Scaler/DevOpsTool_2/GitHubAction
```

If you see something else, **do not continue**.

Run:

```bash
cd /p/Scaler/DevOpsTool_2/GitHubAction
```

Now verify again:

```bash
pwd
```

Expected:

```text
/p/Scaler/DevOpsTool_2/GitHubAction
```

---

# PART 5 — Create the Two Local Directories

We are going to clone the two GitHub repositories.

The `git clone` command will automatically create the directories.

---

# 6. Clone Repository 1

Make sure you are here:

```text
/p/Scaler/DevOpsTool_2/GitHubAction
```

Run:

```bash
git clone https://github.com/discover-devops/gha-lab-01-central.git
```

You should see something similar to:

```text
Cloning into 'gha-lab-01-central'...
```

After cloning, run:

```bash
ls
```

You should see:

```text
gha-lab-01-central
```

---

# 7. Clone Repository 2

You should still be in:

```text
/p/Scaler/DevOpsTool_2/GitHubAction
```

Run:

```bash
git clone https://github.com/discover-devops/gha-lab-01-microservice.git
```

You should see:

```text
Cloning into 'gha-lab-01-microservice'...
```

Now run:

```bash
ls
```

You should see:

```text
gha-lab-01-central
gha-lab-01-microservice
```

---

# 8. Verify the directories

Run:

```bash
pwd
```

Expected:

```text
/p/Scaler/DevOpsTool_2/GitHubAction
```

Run:

```bash
ls
```

Expected:

```text
gha-lab-01-central
gha-lab-01-microservice
```

So our local structure is now:

```text
/p/Scaler/DevOpsTool_2/GitHubAction/
│
├── gha-lab-01-central/
│
└── gha-lab-01-microservice/
```

**Do not create another parent directory.**

We are working directly under your existing:

```text
GitHubAction
```

directory.

---

# PART 6 — Build the CENTRAL Repository

Now we will work only inside:

```text
gha-lab-01-central
```

This repository contains the **master reusable workflow**.

---

# 9. Enter the central repository

Run:

```bash
cd /p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-01-central
```

Verify:

```bash
pwd
```

Expected:

```text
/p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-01-central
```

Run:

```bash
ls -la
```

You should see the Git files, such as:

```text
.
..
.git
```

---

# 10. Create the workflow directory

Run:

```bash
mkdir -p .github/workflows
```

Now verify:

```bash
ls -la
```

You should see:

```text
.github
.git
```

---

# 11. Create `reusable-build.yml`

We need to create this file:

```text
.github/workflows/reusable-build.yml
```

There are several ways to create files in Git Bash.

For students, I recommend using **Notepad** because they can clearly see the file contents.

Run:

```bash
notepad .github/workflows/reusable-build.yml
```

If Windows asks whether you want to create the file, click:

**Yes**

---

# 12. Paste the following YAML

Paste this **exactly** into Notepad:

```yaml
name: Reusable Node.js Build

on:
  workflow_call:
    inputs:
      node_version:
        required: true
        type: string
        description: "The version of Node.js to use"

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node_version }}

      - name: Install Dependencies
        run: npm ci

      - name: Run Tests
        run: npm test
```

Save the file:

**Ctrl + S**

Close Notepad.

---

# 13. Verify the workflow file

Back in Git Bash, run:

```bash
ls .github/workflows
```

Expected:

```text
reusable-build.yml
```

---

# PART 7 — Create the Node.js Application

Our reusable workflow will execute:

```text
npm ci
```

and:

```text
npm test
```

Therefore, we need a Node.js project.

---

# 14. Create `package.json`

Make sure you are still inside:

```text
/p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-01-central
```

Verify:

```bash
pwd
```

Now run:

```bash
notepad package.json
```

Paste:

```json
{
  "name": "gha-reusable-build",
  "version": "1.0.0",
  "scripts": {
    "test": "node test.js"
  }
}
```

Save:

**Ctrl + S**

Close Notepad.

---

# 15. Create `test.js`

Run:

```bash
notepad test.js
```

Paste:

```javascript
console.log("Running application tests...");
console.log("All tests passed!");
```

Save:

**Ctrl + S**

Close Notepad.

---

# PART 8 — VERY IMPORTANT: Create package-lock.json

This is the step that caused the error in your previous run.

Our workflow uses:

```bash
npm ci
```

`npm ci` requires a `package-lock.json`.

Therefore, **before pushing the repository**, we must generate the lock file.

---

# 16. Check whether Node.js and npm are installed

Run:

```bash
node --version
```

You should get something similar to:

```text
v20.x.x
```

Now run:

```bash
npm --version
```

You should get a version number.

If both commands return versions, continue.

---

# 17. Generate package-lock.json

Make sure you are in:

```text
/p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-01-central
```

Run:

```bash
npm install
```

This will create:

```text
package-lock.json
```

---

# 18. Verify the files

Run:

```bash
ls
```

You should now see:

```text
package-lock.json
package.json
test.js
```

Also run:

```bash
ls .github/workflows
```

You should see:

```text
reusable-build.yml
```

---

# 19. Verify the complete directory structure

Run:

```bash
find . -maxdepth 3 -type f
```

You should see something similar to:

```text
./.git/...
./.github/workflows/reusable-build.yml
./package.json
./package-lock.json
./test.js
```

The important structure is:

```text
gha-lab-01-central/
│
├── .github/
│   └── workflows/
│       └── reusable-build.yml
│
├── package.json
├── package-lock.json
└── test.js
```

---

# PART 9 — Test the Application Locally

Before pushing anything to GitHub, let's make sure our Node application works.

Run:

```bash
npm test
```

Expected output:

```text
> gha-reusable-build@1.0.0 test
> node test.js

Running application tests...
All tests passed!
```

If you see this, our application is working.

---

# PART 10 — Commit the CENTRAL Repository

Now we need to push everything to GitHub.

---

# 20. Check Git status

Run:

```bash
git status
```

You should see files such as:

```text
.github/workflows/reusable-build.yml
package.json
package-lock.json
test.js
```

---

# 21. Add all files

Run:

```bash
git add .
```

There should be no error.

---

# 22. Check Git status again

Run:

```bash
git status
```

You should now see the files under:

```text
Changes to be committed
```

---

# 23. Commit the files

Run:

```bash
git commit -m "Create reusable Node.js workflow"
```

You should get a message similar to:

```text
[main xxxxxxx] Create reusable Node.js workflow
```

---

# 24. Push to GitHub

Run:

```bash
git push -u origin main
```

If GitHub asks you to authenticate, complete the GitHub authentication.

After successful push, you should see something similar to:

```text
[new branch] main -> main
```

---

# PART 11 — Verify CENTRAL Repository on GitHub

Open:

```text
https://github.com/discover-devops/gha-lab-01-central
```

You should now see:

```text
.github
package-lock.json
package.json
test.js
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
reusable-build.yml
```

---

# PART 12 — Understand What We Have Built

Before creating the second repository, stop here and understand the architecture.

Our central repository is:

```text
gha-lab-01-central
```

It contains:

```text
reusable-build.yml
```

The important part is:

```yaml
on:
  workflow_call:
```

This means:

> "This workflow is not intended to be triggered directly by a normal push. Other workflows can call this workflow."

The document specifically identifies `workflow_call` as the trigger used to create a workflow that can be called from other workflows. 03 Advanced Techniques and Best…

We also defined an input:

```yaml
inputs:
  node_version:
    required: true
    type: string
```

This means the calling repository can tell the reusable workflow which Node.js version to use.

And this:

```yaml
node-version: ${{ inputs.node_version }}
```

uses the value supplied by the caller. 03 Advanced Techniques and Best…

---

# PART 13 — Build the MICROSERVICE Repository

Now we will create the second repository.

The second repository will **call** the reusable workflow.

---

# 25. Go to the microservice directory

Run:

```bash
cd /p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-01-microservice
```

Verify:

```bash
pwd
```

Expected:

```text
/p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-01-microservice
```

---

# 26. Verify repository is empty

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

There should not yet be a workflow.

---

# 27. Create the workflow directory

Run:

```bash
mkdir -p .github/workflows
```

---

# 28. Create the caller workflow

Run:

```bash
notepad .github/workflows/microservice-a.yml
```

Paste exactly:

```yaml
name: Microservice A Pipeline

on:
  push:
    branches:
      - main

jobs:
  call-master-build:
    uses: discover-devops/gha-lab-01-central/.github/workflows/reusable-build.yml@main
    with:
      node_version: '20'
```

Save:

**Ctrl + S**

Close Notepad.

---

# PART 14 — Understand This YAML

The important section is:

```yaml
jobs:
  call-master-build:
    uses: discover-devops/gha-lab-01-central/.github/workflows/reusable-build.yml@main
```

We are saying:

> "Do not define the build logic here. Go to the central repository and use its reusable workflow."

The source document describes exactly this pattern: the microservice workflow points to the central reusable workflow and passes the required input. 03 Advanced Techniques and Best…

Then:

```yaml
with:
  node_version: '20'
```

passes:

```text
20
```

to:

```yaml
inputs.node_version
```

in the central workflow.

---

# PART 15 — Add Application Files to Microservice

For this lab, the caller repository should also contain the Node.js application files because the reusable workflow checks out **the caller repository's code**.

This is an important point.

The central workflow is stored in:

```text
gha-lab-01-central
```

but when the reusable workflow runs:

```yaml
- name: Checkout Code
  uses: actions/checkout@v4
```

the code checked out is the repository that called the reusable workflow.

Therefore, we need:

```text
package.json
package-lock.json
test.js
```

in the microservice repository as well.

---

# 29. Create `package.json`

Run:

```bash
notepad package.json
```

Paste:

```json
{
  "name": "gha-microservice-a",
  "version": "1.0.0",
  "scripts": {
    "test": "node test.js"
  }
}
```

Save and close Notepad.

---

# 30. Create `test.js`

Run:

```bash
notepad test.js
```

Paste:

```javascript
console.log("Microservice A tests are running...");
console.log("Microservice A tests passed!");
```

Save and close Notepad.

---

# 31. Generate package-lock.json

This is **mandatory** because the reusable workflow executes:

```bash
npm ci
```

Run:

```bash
npm install
```

This creates:

```text
package-lock.json
```

---

# 32. Verify the files

Run:

```bash
ls
```

Expected:

```text
package-lock.json
package.json
test.js
```

Now:

```bash
ls .github/workflows
```

Expected:

```text
microservice-a.yml
```

---

# PART 16 — Test the Microservice Locally

Run:

```bash
npm test
```

Expected:

```text
> gha-microservice-a@1.0.0 test
> node test.js

Microservice A tests are running...
Microservice A tests passed!
```

---

# PART 17 — Commit the Microservice

## 33. Check Git status

Run:

```bash
git status
```

You should see:

```text
.github/workflows/microservice-a.yml
package.json
package-lock.json
test.js
```

---

# 34. Add the files

Run:

```bash
git add .
```

---

# 35. Check status

Run:

```bash
git status
```

The files should now appear under:

```text
Changes to be committed
```

---

# 36. Commit

Run:

```bash
git commit -m "Call reusable build workflow"
```

---

# 37. Push

Run:

```bash
git push -u origin main
```

---

# PART 18 — Watch the GitHub Actions Pipeline

Now open:

```text
https://github.com/discover-devops/gha-lab-01-microservice
```

Click:

**Actions**

You should see:

```text
Microservice A Pipeline
```

Click the workflow run.

You should see a job similar to:

```text
call-master-build
```

Click it.

You should see the reusable workflow execute:

```text
Checkout Code
Setup Node.js
Install Dependencies
Run Tests
```

---

# 38. Expected result

The workflow should complete successfully.

You should see:

```text
✓ Checkout Code
✓ Setup Node.js
✓ Install Dependencies
✓ Run Tests
```

And the test output should contain:

```text
Microservice A tests are running...
Microservice A tests passed!
```

---

# PART 19 — Understand What Actually Happened

This is the most important part of the lab.

We have:

```text
Repository 1
gha-lab-01-central
        |
        | reusable-build.yml
        |
        | workflow_call
        |
        v
Repository 2
gha-lab-01-microservice
        |
        | microservice-a.yml
        |
        | uses:
        v
Central Workflow
```

The microservice repository only contains:

```yaml
jobs:
  call-master-build:
    uses: discover-devops/gha-lab-01-central/.github/workflows/reusable-build.yml@main

    with:
      node_version: '20'
```

It doesn't contain:

```text
Checkout
Setup Node
npm ci
npm test
```

Those instructions are centralized.

---

# PART 20 — Demonstrate the Real Benefit

Now let's demonstrate **why reusable workflows are useful**.

Imagine we have:

```text
Microservice A
Microservice B
Microservice C
Microservice D
...
Microservice T
```

Without reusable workflows:

```text
Repo A → 100 lines YAML
Repo B → 100 lines YAML
Repo C → 100 lines YAML
Repo D → 100 lines YAML
...
Repo T → 100 lines YAML
```

If you need to change the build process, you potentially have to change many repositories.

With reusable workflows:

```text
                    ┌── Microservice A
                    |
                    ├── Microservice B
                    |
Central Workflow ───┼── Microservice C
                    |
                    ├── Microservice D
                    |
                    └── Microservice T
```

The central workflow contains the common build logic.

The uploaded document uses this exact "20 repositories / central workflow" problem to introduce reusable workflows. 03 Advanced Techniques and Best…

---

# PART 21 — Demonstrate the Input

Our central workflow has:

```yaml
inputs:
  node_version:
    required: true
    type: string
```

The caller sends:

```yaml
with:
  node_version: '20'
```

The central workflow receives it here:

```yaml
node-version: ${{ inputs.node_version }}
```

So the flow is:

```text
Microservice
     |
     | node_version = 20
     v
Reusable Workflow
     |
     v
actions/setup-node
     |
     v
Node.js 20
```

This is the beginning of **parameterized reusable workflows**.

---

# PART 22 — Final Directory Structure

At the end of the lab, your local directory should look like this:

```text
/p/Scaler/DevOpsTool_2/GitHubAction/
│
├── gha-lab-01-central/
│   │
│   ├── .github/
│   │   └── workflows/
│   │       └── reusable-build.yml
│   │
│   ├── package.json
│   ├── package-lock.json
│   └── test.js
│
└── gha-lab-01-microservice/
    │
    ├── .github/
    │   └── workflows/
    │       └── microservice-a.yml
    │
    ├── package.json
    ├── package-lock.json
    └── test.js
```

---

# PART 23 — All Git Bash Commands in One Place

For a student who wants to copy/paste, here is the complete command sequence.

## Create the two local repositories

Starting location:

```bash
cd /p/Scaler/DevOpsTool_2/GitHubAction
```

Then:

```bash
git clone https://github.com/discover-devops/gha-lab-01-central.git
git clone https://github.com/discover-devops/gha-lab-01-microservice.git
```

---

## Central repository

```bash
cd /p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-01-central
```

```bash
mkdir -p .github/workflows
```

Create the three files:

```bash
notepad .github/workflows/reusable-build.yml
```

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

Commit:

```bash
git add .
git commit -m "Create reusable Node.js workflow"
git push -u origin main
```

---

## Microservice repository

```bash
cd /p/Scaler/DevOpsTool_2/GitHubAction/gha-lab-01-microservice
```

```bash
mkdir -p .github/workflows
```

Create:

```bash
notepad .github/workflows/microservice-a.yml
```

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

Commit:

```bash
git add .
git commit -m "Call reusable build workflow"
git push -u origin main
```

Then:

**GitHub → `gha-lab-01-microservice` → Actions → Microservice A Pipeline**

---

# PART 24 — What Students Learned

At the end of this lab, students should be able to answer these questions.

### 1. What problem does a reusable workflow solve?

**Copy-paste and maintenance of the same workflow across multiple repositories.**

### 2. What creates a reusable workflow?

```yaml
on:
  workflow_call:
```

### 3. How does another workflow call it?

Using:

```yaml
uses:
```

For example:

```yaml
uses: discover-devops/gha-lab-01-central/.github/workflows/reusable-build.yml@main
```

### 4. How do we pass a value to the reusable workflow?

Using:

```yaml
with:
```

Example:

```yaml
with:
  node_version: '20'
```

### 5. How does the reusable workflow receive that value?

Using:

```yaml
${{ inputs.node_version }}
```

### 6. Why did we create `package-lock.json`?

Because our reusable workflow uses:

```bash
npm ci
```

and `npm ci` requires a lock file.

### 7. What is the architecture?

```text
                GitHub Actions
                     |
                     |
       ┌─────────────┴─────────────┐
       |                           |
       v                           v
gha-lab-01-central       gha-lab-01-microservice
       |                           |
       | reusable-build.yml        |
       |                           |
       └────────── workflow_call ◄──┘
                     |
                     v
              Node.js Build
                     |
               ┌─────┴─────┐
               |           |
             npm ci      npm test
```

## One sentence for students

> **Reusable Workflow = Write the CI/CD logic once in a central repository and let multiple repositories call it with their own inputs.**

That is the core learning objective of Lab 1. 03 Advanced Techniques and Best…
