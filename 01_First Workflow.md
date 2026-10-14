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

# Lab 1 --- First Workflow

## Goal

Create your first GitHub Actions workflow, trigger it with a push, and
read the logs.

## What you learn

By the end of this lab you should understand:

-   A workflow lives in `.github/workflows/`.
-   A `push` can trigger a workflow.
-   A workflow contains jobs.
-   A job runs on a runner.
-   A job contains steps.
-   `run:` executes a shell command.
-   `uses:` calls a reusable GitHub Action.
-   A fresh runner does not automatically contain your repository files.
-   `actions/checkout@v4` brings the repository code onto the runner.

------------------------------------------------------------------------

## Prerequisite --- Create the sample project

If you already created the Day 1 project, skip this section.

### 1. Create a GitHub repository

Create a repository named:

``` text
gha-day1-app
```

For the original Day 1 lab, use a **Public** repository.

Add a README file.

### 2. Clone the repository

Run:

``` bash
git clone https://github.com/YOUR-USERNAME/gha-day1-app.git
cd gha-day1-app
```

### 3. Create the application

Run:

``` bash
cat > package.json << 'EOF'
{
  "name": "gha-day1-app",
  "version": "1.0.0",
  "scripts": {
    "build": "mkdir -p dist && cp app.js dist/app.js",
    "test": "node --test"
  }
}
EOF

cat > app.js << 'EOF'
function greet(name) {
  return `Hello, ${name}!`;
}

module.exports = { greet };

if (require.main === module) {
  console.log(greet("GitHub Actions"));
}
EOF

cat > app.test.js << 'EOF'
const test = require('node:test');
const assert = require('node:assert');
const { greet } = require('./app');

test('greet returns a greeting', () => {
  assert.strictEqual(greet('Asha'), 'Hello, Asha!');
});
EOF

mkdir -p .github/workflows
```

### 4. Push the project

``` bash
git add .
git commit -m "Add sample app"
git push origin main
```

------------------------------------------------------------------------

# Part 1 --- Create the workflow

Run:

``` bash
mkdir -p .github/workflows

cat > .github/workflows/hello.yml << 'EOF'
name: Hello GitHub Actions

on: push

jobs:
  hello:
    runs-on: ubuntu-latest

    steps:
      - name: Say hello
        run: echo "Hello from GitHub Actions"

      - name: Show where we are running
        run: |
          echo "Repository : ${{ github.repository }}"
          echo "Event      : ${{ github.event_name }}"
          echo "Triggered by: ${{ github.actor }}"
          echo "Runner OS  : ${{ runner.os }}"

      - name: List files BEFORE checkout
        run: ls -la

      - name: Checkout the code
        uses: actions/checkout@v4

      - name: List files AFTER checkout
        run: ls -la
EOF
```

## Part 2 --- Commit and push

``` bash
git add .
git commit -m "Lab 1: first workflow"
git push origin main
```

## Part 3 --- Verify in GitHub

Go to:

**Repository → Actions → Lab 1: first workflow**

Open the latest run.

Open the `hello` job.

Expand each step.

### Expected observation

Before checkout:

``` text
The runner folder is essentially empty.
```

After checkout:

``` text
app.js
app.test.js
package.json
README.md
...
```

## Teaching explanation

Think of the runner as a brand-new computer.

``` text
GitHub event
     ↓
Workflow
     ↓
Job
     ↓
Fresh runner
     ↓
Checkout code
     ↓
Run commands
```

`git push` is the event.

GitHub finds the workflow.

GitHub starts a fresh runner.

`actions/checkout@v4` downloads the repository into that runner.

## Common problems

### Nothing appears in Actions

Check that the file is exactly:

``` text
.github/workflows/hello.yml
```

### Invalid workflow file

Check YAML indentation. Use spaces, not tabs.

### Files are missing before checkout

That is expected. The checkout step has not run yet.

## Lab 1 learning checkpoint

You should now be able to explain:

> What is a workflow, job, runner, step, `run`, `uses`, and why checkout
> is required?
