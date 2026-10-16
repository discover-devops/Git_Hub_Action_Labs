# GitHub Actions Advanced Labs

Secrets, Triggers, Caching, Matrix, Needs and Artifacts

You will build one small Node.js project and then add one workflow file per lab. Every lab is separate. Copy, paste, run, and check the "You should see" box before moving on.

---

## How This Runbook Works

| Lab | Topic | Workflow file you create | Starts by |
|---|---|---|---|
| 0 | Project and repository setup | none | not applicable |
| 1 | Secrets and Variables (the Vault) | `lab1-secrets.yml` | Run workflow button |
| 2 | Event triggers: push, schedule, workflow_dispatch | `lab2-triggers.yml` | push, button, clock |
| 2B | Manual inputs: dropdown, text, checkbox | `lab2b-manual-inputs.yml` | Run workflow button |
| 3 | Matrix strategy | `lab3-matrix.yml` | Run workflow button |
| 4 | Caching | `lab4-cache.yml` | Run workflow button |
| 5 | Job dependencies with needs | `lab5-needs.yml` | Run workflow button |
| 6 | Artifacts | `lab6-artifacts.yml` | Run workflow button |
| 7 | Final project: Ultimate Production Pipeline | `lab7-ultimate-pipeline.yml` | push, button, clock |

Labs 1, 3, 4, 5 and 6 start only from the button. This keeps your Actions tab clean, because a push will not start every lab at once.

Do the labs in order. Lab 0 is required for all of them. Lab 1 is required for Lab 7.

---

## Rules That Prevent 90 Percent of Problems

- Copy the whole code block, from the first line to the last line. Never copy half.
- YAML uses spaces for indentation. Tabs break it. Never type YAML by hand, always paste it.
- Run every command inside the project folder `github-actions-advanced-lab`.
- A workflow file must be pushed to the `main` branch before the Run workflow button appears.
- If the Actions tab shows a red "Invalid workflow file" message, the file has an indentation or paste error. Paste it again.

### How to start a workflow with the button

1. Open your repository on GitHub and click the Actions tab.
2. In the left sidebar click the workflow name, for example "Lab 3 - Matrix Strategy".
3. Click the Run workflow dropdown on the right.
4. Click the green Run workflow button.
5. Refresh the page after a few seconds, then click the new run at the top.

---

## Understanding the Basics First

**Key setting: a workflow is a file, and a run is one execution of that file.**

**Default Behavior**
- Each YAML file in `.github/workflows/` is one workflow.
- Every time its trigger fires, GitHub creates one new run of that workflow.

**What changes as you add labs**
- One push can start several workflows, because each file with a matching `on: push` starts its own run.
- A trigger written in `on:` is only permission to start. Nothing starts until that event really happens.

**Why it matters**
- 3 runs in the Actions list does not mean 3 workflows. It can be 1 workflow that ran 3 times.
- Under every run title GitHub shows what started it. "pushed by" means a push, and "Manually run by" means the button.

---

# Lab 0: Project and Repository Setup

**Goal:** create the Node.js project and the GitHub repository that every other lab uses.
**Time:** 15 minutes

### Prerequisites

- Git Bash (Windows) or a terminal (Mac or Linux)
- Node.js 22 installed. Check with `node -v`
- A GitHub account with SSH access set up. If you use HTTPS instead, replace the remote URL in Step 3.

Replace `YOUR-GITHUB-USERNAME` with your own username wherever you see it.

### Step 1: Create an empty repository on GitHub

1. On GitHub click New repository.
2. Name: `github-actions-advanced-lab`
3. Choose Public. Do NOT add a README, .gitignore or license.
4. Click Create repository.

### Step 2: Create the project folder

```bash
mkdir github-actions-advanced-lab
cd github-actions-advanced-lab
git init
git branch -M main
mkdir -p src test .github/workflows
```

### Step 3: Create the project files

package.json:

```bash
cat > package.json << 'EOF'
{
  "name": "robo-kitchen",
  "version": "1.0.0",
  "scripts": {
    "test": "node --test",
    "build": "node build.js"
  },
  "dependencies": {
    "lodash": "^4.17.21"
  }
}
EOF
```

The app code:

```bash
cat > src/kitchen.js << 'EOF'
const _ = require('lodash');

function totalBill(items) {
  return _.sumBy(items, (i) => i.price * i.qty);
}

function mostOrdered(items) {
  return _.maxBy(items, 'qty').name;
}

module.exports = { totalBill, mostOrdered };
EOF
```

The tests:

```bash
cat > test/kitchen.test.js << 'EOF'
const test = require('node:test');
const assert = require('node:assert');
const { totalBill, mostOrdered } = require('../src/kitchen');

const orders = [
  { name: 'pasta', price: 10, qty: 3 },
  { name: 'pizza', price: 12, qty: 5 },
];

test('totalBill adds up all items', () => {
  assert.strictEqual(totalBill(orders), 90);
});

test('mostOrdered returns the popular dish', () => {
  assert.strictEqual(mostOrdered(orders), 'pizza');
});
EOF
```

The build script (it makes the "finished dish" used in the Artifacts lab):

```bash
cat > build.js << 'EOF'
const fs = require('fs');
fs.mkdirSync('dist', { recursive: true });
fs.writeFileSync(
  'dist/index.html',
  `<h1>Robo Kitchen</h1><p>Built from commit: ${process.env.GITHUB_SHA || 'local'}</p>`
);
console.log('Build complete: dist/index.html created');
EOF
```

The ignore file:

```bash
printf "node_modules\ndist\n" > .gitignore
```

### Step 4: Install and test on your own machine

```bash
npm install
npm test
npm run build
ls package-lock.json dist
```

**You should see:** 2 passing tests, the line `Build complete`, and both `package-lock.json` and `dist` listed.

The file `package-lock.json` is important. The cache key and `npm ci` both depend on it, so it must be committed.

### Step 5: Push to GitHub

```bash
git remote add origin git@github.com:YOUR-GITHUB-USERNAME/github-actions-advanced-lab.git
git add .
git commit -m "Add Robo Kitchen app"
git push -u origin main
```

HTTPS alternative for the remote line: `git remote add origin https://github.com/YOUR-GITHUB-USERNAME/github-actions-advanced-lab.git`

**You should see:** your files on the repository page, including `package-lock.json`. The `dist` and `node_modules` folders must NOT be there.

---

# Lab 1: Secrets and Variables (The Vault)

**Goal:** store a password safely and see how GitHub hides it.
**Time:** 20 minutes

**Key setting: `${{ secrets.NAME }}` reads an encrypted value stored in GitHub, so the real value never appears in your code.**

**Default Behavior**
- Anything written in a YAML file is plain text.
- Git keeps history forever, so a deleted password still sits in old commits.
- Anyone with read access to the repository can read it.

**What changes with Secrets**
- You save the value in the GitHub UI. GitHub encrypts it with Libsodium sealed boxes.
- Nobody can read it again, not even you. You can only update or delete it.
- If a step prints it, the log shows `***`.

**Why it matters**
- It prevents the "junior developer saw the AWS password in the repo" incident.
- You change a password in one place without editing code.

**Secret or Variable?**

| | Secret | Variable |
|---|---|---|
| Use for | Passwords, tokens, SSH keys | App name, region, feature flags |
| Shown in logs | No, masked as `***` | Yes, plain text |
| Readable in the UI later | No | Yes |
| Syntax | `${{ secrets.NAME }}` | `${{ vars.NAME }}` |

Rule: if a leak would cause damage, use a Secret.

### Step 1: Create the secret and the variable

1. Open your repository, then Settings.
2. In the left sidebar open Secrets and variables, then Actions.
3. On the Secrets tab click New repository secret.
   - Name: `SUPER_SECRET_TOKEN`
   - Secret: `12345-my-hidden-password`
   - Click Add secret.
4. Open the Variables tab and click New repository variable.
   - Name: `APP_NAME`
   - Value: `robo-kitchen`
   - Click Add variable.

Names are case sensitive. A typo in the name gives an empty value with no error.

### Step 2: Create the workflow

```bash
cat > .github/workflows/lab1-secrets.yml << 'EOF'
name: Lab 1 - Secrets and Variables

on:
  workflow_dispatch:

jobs:
  vault-demo:
    runs-on: ubuntu-latest
    env:
      APP_NAME: ${{ vars.APP_NAME }}

    steps:
      - name: Read the variable (not sensitive)
        run: echo "App name is $APP_NAME"

      - name: Use the secret safely
        env:
          API_TOKEN: ${{ secrets.SUPER_SECRET_TOKEN }}
        run: |
          if [ -z "$API_TOKEN" ]; then
            echo "SECRET IS EMPTY - check the secret name"
          else
            echo "Secret loaded successfully"
          fi
          echo "Token length is ${#API_TOKEN} characters"
          echo "Trying to print the token: $API_TOKEN"

      - name: Masking is only text matching (dummy token only)
        env:
          API_TOKEN: ${{ secrets.SUPER_SECRET_TOKEN }}
        run: |
          echo "Reversed token: $(echo "$API_TOKEN" | rev)"
EOF
```

### Step 3: Push and run

```bash
git add .
git commit -m "Lab 1: secrets and variables"
git push
```

Start it with the button (see "How to start a workflow with the button" above).

**You should see:**
- `App name is robo-kitchen` in plain text.
- `Secret loaded successfully` and `Token length is 24 characters`.
- `Trying to print the token: ***` (GitHub masked it).
- In the last step, the reversed token appears in plain text. GitHub only hides the exact text it knows. A changed version of the secret is not hidden.

**Teaching point:** masking is a safety net, not protection. Never print secrets on purpose, and never do the reverse trick with a real secret.

### Experiments

1. Go to Settings, Secrets and variables, Actions, update `SUPER_SECRET_TOKEN` to a longer value, and run again. The length changes.
2. Rename the secret in the YAML to `WRONG_NAME`, push, and run. The log says `SECRET IS EMPTY`. A wrong name does not fail, it gives an empty value.
3. Bad practice talk: imagine `run: echo "password 12345-my-hidden-password"` in the YAML. Ask: who can see this, and can we ever remove it from Git history?

### Good habits

- Pass secrets through `env:` (as in this lab) instead of writing `${{ secrets.X }}` directly inside a `run:` command.
- Set the secret on the smallest scope possible, which is the single step that needs it.
- Secrets are not passed to workflows triggered from forked repositories. This stops strangers stealing them with a pull request.
- Base64 is encoding, not encryption. It does not protect a password.

---

# Lab 2: Event Triggers (When Does the Factory Wake Up?)

**Goal:** see three different events start the same workflow.
**Time:** 25 minutes

**Key setting: the `on:` block lists the events that may start the workflow, and any one event is enough (they work as OR).**

**Default Behavior**
- `on: push` starts the workflow only when someone pushes code.

**What changes with more triggers**
- `workflow_dispatch` adds a Run workflow button, and it can ask for inputs.
- `schedule` starts the workflow on a timer using cron, with nobody pushing.
- `paths:` makes a push trigger fire only when certain files change.

**Why it matters**
- One file can serve automatic runs, manual runs and nightly jobs.
- Manual deploys, nightly backups and weekly security scans all need these triggers.

**Reading a cron expression**

```
minute  hour  day-of-month  month  day-of-week
  30     20        *          *         *
```

This means 20:30 every day.

- Cron time is always UTC. 20:30 UTC is 02:00 AM in India (IST, UTC plus 5:30).
- Scheduled workflows run only from the default branch, and they can start a few minutes late.
- The shortest allowed interval is 5 minutes.
- In public repositories, GitHub disables scheduled workflows after 60 days without repository activity.

### Step 1: Create the workflow

```bash
cat > .github/workflows/lab2-triggers.yml << 'EOF'
name: Lab 2 - Event Triggers

on:
  push:
    branches:
      - main
    paths:
      - '.github/workflows/lab2-triggers.yml'
      - 'trigger-test.txt'
  workflow_dispatch:
    inputs:
      environment:
        description: 'Which environment to deploy to?'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
  schedule:
    - cron: '30 20 * * *'

jobs:
  show-trigger:
    runs-on: ubuntu-latest
    steps:
      - name: Show what started this run
        run: |
          echo "Event name       : ${{ github.event_name }}"
          echo "Started by       : ${{ github.actor }}"
          echo "Branch           : ${{ github.ref_name }}"
          echo "Environment input: ${{ inputs.environment || 'not a manual run' }}"
          echo "Cron schedule    : ${{ github.event.schedule || 'not a scheduled run' }}"
EOF
```

### Step 2: Push (this is trigger 1: push)

```bash
git add .
git commit -m "Lab 2: event triggers"
git push
```

**You should see:** a new run of "Lab 2 - Event Triggers" with "pushed by" under the title. Open the job log. It prints `Event name : push` and `Environment input: not a manual run`.

### Step 3: Run with the button (trigger 2: workflow_dispatch)

Start "Lab 2 - Event Triggers" with the button. Choose `production` in the dropdown.

**You should see:** the title says "Manually run by", and the log prints `Event name : workflow_dispatch` and `Environment input: production`.

### Step 4: Prove the paths filter works

Change a file that is NOT in the paths list:

```bash
echo "hello" > README.md
git add .
git commit -m "Add README only"
git push
```

**You should see:** no new run of Lab 2. The files changed do not match `paths:`.

Now change a file that IS in the list:

```bash
echo "trigger test" >> trigger-test.txt
git add .
git commit -m "Touch trigger-test.txt"
git push
```

**You should see:** a new push run of Lab 2.

### Step 5: Watch the schedule fire (trigger 3: schedule)

1. Check the current UTC time:

```bash
date -u
```

2. Open `.github/workflows/lab2-triggers.yml` in an editor. Change the cron line to a time about 10 minutes ahead. Example: if `date -u` shows 14:07, use `'20 14 * * *'`.
3. Save, then push:

```bash
git add .
git commit -m "Test schedule"
git push
```

4. Wait. A scheduled run can start 5 to 15 minutes late. Open the Actions tab and look for a run that is not "pushed by" and not "Manually run by".

**You should see:** the log prints `Event name : schedule` and the cron text you set.

5. Put the nightly time back so it does not keep running:

```bash
sed -i "s/- cron: .*/- cron: '30 20 * * *'/" .github/workflows/lab2-triggers.yml
git add .
git commit -m "Restore nightly cron"
git push
```

If `sed -i` does not work on your machine, edit the line by hand.

### Check yourself

1. You push once and 2 workflows start. Why?
2. Cron says `0 9 * * *`. What time is that in IST?
3. Why did the README push not start Lab 2?

Answers: 1) Two workflow files both listen to push, and each starts its own run. 2) 09:00 UTC is 2:30 PM IST. 3) README.md is not in the `paths:` list.

---

# Lab 2B: Manual Inputs (Dropdown, Text Box, Checkbox)

**Goal:** build a workflow that only a human can start, with three kinds of input.
**Time:** 15 minutes

**Key setting: a workflow with only `on: workflow_dispatch` never starts by itself, it runs only when someone clicks Run workflow.**

**Default Behavior**
- Pushing code does not start it, and the clock does not start it.

**What changes with inputs**
- Input types are `choice` (dropdown), `string` (text box) and `boolean` (checkbox).
- You read each value with `${{ inputs.NAME }}`.

**Why it matters**
- It is the right pattern for actions a human must choose on purpose: production deploys, database restores, cleanups.

### Step 1: Create the workflow

```bash
cat > .github/workflows/lab2b-manual-inputs.yml << 'EOF'
name: Lab 2B - Manual Deploy Demo

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Which environment to deploy to?'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
      version:
        description: 'Version number to deploy'
        required: true
        default: '1.0.0'
        type: string
      dry_run:
        description: 'Dry run only (no real deployment)?'
        required: false
        default: true
        type: boolean

jobs:
  manual-demo:
    runs-on: ubuntu-latest
    steps:
      - name: Show the inputs
        run: |
          echo "Trigger event : ${{ github.event_name }}"
          echo "Environment   : ${{ inputs.environment }}"
          echo "Version       : ${{ inputs.version }}"
          echo "Dry run       : ${{ inputs.dry_run }}"

      - name: Decide what to do
        run: |
          if [ "${{ inputs.dry_run }}" = "true" ]; then
            echo "Dry run selected, nothing will be deployed"
          else
            echo "Deploying version ${{ inputs.version }} to ${{ inputs.environment }}"
          fi
EOF
```

### Step 2: Push

```bash
git add .
git commit -m "Lab 2B: manual inputs"
git push
```

**You should see:** NO new run after this push. This workflow ignores push.

### Step 3: Run it twice with the button

1. First run: keep the checkbox ticked, choose `staging`, version `1.0.0`.
2. Second run: untick the checkbox, choose `production`, type version `2.5.1`.

**You should see:** the first log says "Dry run selected". The second says "Deploying version 2.5.1 to production".

---

# Lab 3: Matrix Strategy (Many Stoves at Once)

**Goal:** test the app on many operating systems and Node versions with a few lines of YAML.
**Time:** 20 minutes

**Key setting: `strategy.matrix` multiplies your lists of values and starts one parallel runner for every combination.**

**Default Behavior**
- One job runs on one machine with one setup.
- To test two setups you would copy the whole job twice.

**What changes with Matrix**
- You list values once: 2 operating systems and 3 Node versions.
- GitHub makes 2 x 3 = 6 combinations and starts them all at the same time.
- Inside the job, `${{ matrix.os }}` and `${{ matrix.node }}` hold the values for that runner.
- `exclude` removes one combination. We remove Windows with Node 24, so 5 jobs run.

**Why it matters**
- A few lines replace hundreds of copied lines.
- Total time equals the slowest job, not the sum of all jobs.
- It catches "works on my machine" problems before customers see them.

**Good to know**
- By default, if one job fails, GitHub cancels the rest (`fail-fast: true`). We set `fail-fast: false` so every job finishes.
- Parallel jobs use more runner minutes in total. In private repositories Windows counts 2 times and macOS 10 times.
- One run can have at most 256 matrix jobs.

### Step 1: Create the workflow

```bash
cat > .github/workflows/lab3-matrix.yml << 'EOF'
name: Lab 3 - Matrix Strategy

on:
  workflow_dispatch:
    inputs:
      break_node22:
        description: 'Make every Node 22 job fail? (fail-fast demo)'
        required: false
        default: false
        type: boolean

jobs:
  test:
    name: Test on ${{ matrix.os }} with Node ${{ matrix.node }}
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest]
        node: [20, 22, 24]
        exclude:
          - os: windows-latest
            node: 24

    steps:
      - name: Show the matrix values of this runner
        shell: bash
        run: |
          echo "OS from matrix   : ${{ matrix.os }}"
          echo "Node from matrix : ${{ matrix.node }}"
          echo "Runner OS        : $RUNNER_OS"

      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}

      - name: Check versions
        shell: bash
        run: |
          node -v
          npm -v

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Pretend there is a bug in Node 22
        shell: bash
        run: |
          if [ "${{ inputs.break_node22 }}" = "true" ] && [ "${{ matrix.node }}" = "22" ]; then
            echo "Simulated failure on Node 22"
            exit 1
          fi
          echo "No bug found"

      - name: Wait a little so you can see parallel runs
        shell: bash
        run: sleep 15
EOF
```

### Step 2: Push and run

```bash
git add .
git commit -m "Lab 3: matrix"
git push
```

Start "Lab 3 - Matrix Strategy" with the button. Leave the checkbox unticked.

**You should see:**
1. On the left of the run page, 5 jobs with names like `Test on ubuntu-latest with Node 20`.
2. All 5 show the yellow running circle at the same time. This is parallel execution.
3. Each job prints different matrix values in the first step.
4. The Node version printed by `node -v` matches `matrix.node` for that job.

Ask students: "We listed 2 operating systems and 3 Node versions. Why 5 jobs and not 6?"

### Experiments

**Experiment 1: remove exclude.** Delete these 3 lines from the file, then commit, push and run:

```yaml
        exclude:
          - os: windows-latest
            node: 24
```

You get 6 jobs.

**Experiment 2: add a third operating system.** Change the os line to:

```yaml
        os: [ubuntu-latest, windows-latest, macos-latest]
```

With exclude still removed you get 3 x 3 = 9 jobs. One added word created 3 new runners.

**Experiment 3: fail-fast.**
1. Put the exclude block and the old os line back.
2. Run the workflow with the checkbox ticked. The two Node 22 jobs fail and the other 3 still finish, because `fail-fast: false`.
3. Change `fail-fast: false` to `fail-fast: true`, push, and run again with the box ticked. The other jobs are cancelled as soon as one fails.

### Check yourself

1. A matrix has 3 operating systems and 4 Node versions, and 2 combinations are excluded. How many jobs run?
2. Why is the total time of a matrix equal to the slowest job?

Answers: 1) 3 x 4 - 2 = 10 jobs. 2) All jobs run in parallel, so you wait only for the last one.

---

# Lab 4: Caching (The Hot Water Tank)

**Goal:** understand how a brand-new empty runner can still reuse files from an earlier run.
**Time:** 25 minutes

**Key setting: the cache is not stored on the runner. It lives on GitHub's servers, and every new runner downloads it at the start of the job.**

**Default Behavior**
- Every runner is a brand-new empty machine, destroyed when the job ends.
- So every run downloads all packages from the internet again.

**What changes with the cache step**
- At the start (restore): the cache step asks GitHub if a saved folder exists for your key. If yes, it copies the folder onto the new runner.
- At the end (save): a hidden "Post Cache" step copies the folder back to GitHub, but only if the key was not found at the start.
- **Cache miss:** the key is not found. Packages are downloaded, then saved.
- **Cache hit:** the key is found. The folder is restored in seconds.
- **Lock file changes:** the hash changes, so the key changes, and old packages are never reused by mistake.
- **restore-keys:** if the exact key is missing, GitHub restores the closest older cache that starts with this prefix.

**Why it matters**
- Feedback time drops from minutes to seconds.
- Less network use and fewer failures from slow downloads.

**Analogy.** The runner is a kitchen that is demolished after every order. The cache is a hot water tank in a warehouse that is never demolished. Each new kitchen gets water delivered from the tank instead of boiling it from scratch.

**How the key is built**

```
npm-Linux-8f7d6a5c...
 |     |      |
tool   OS    hash of package-lock.json
```

**Good to know**
- Caches not used for 7 days are deleted. Each repository can hold 10 GB.
- A cache is a speed boost only. The workflow must still work with an empty cache.

### Step 1: Create the workflow

```bash
cat > .github/workflows/lab4-cache.yml << 'EOF'
name: Lab 4 - Caching

on:
  workflow_dispatch:

jobs:
  cache-demo:
    runs-on: ubuntu-latest
    steps:
      - name: Which runner am I?
        run: |
          echo "Runner name : $RUNNER_NAME"
          echo "Run number  : ${{ github.run_number }}"
          echo "Attempt     : ${{ github.run_attempt }}"

      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Look at the npm folder BEFORE the cache step
        run: du -sh ~/.npm 2>/dev/null || echo "~/.npm does not exist yet (empty runner)"

      - name: Cache npm packages
        id: cache
        uses: actions/cache@v4
        with:
          path: ~/.npm
          key: npm-${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            npm-${{ runner.os }}-

      - name: What did the cache step do?
        run: |
          echo "Exact cache hit : ${{ steps.cache.outputs.cache-hit || 'false' }}"
          echo "Key we asked for: npm-${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}"

      - name: Look at the npm folder AFTER the cache step
        run: du -sh ~/.npm 2>/dev/null || echo "~/.npm still does not exist (cache miss)"

      - name: Install dependencies (timed)
        run: |
          start=$(date +%s)
          npm ci
          end=$(date +%s)
          echo "npm ci took $((end - start)) seconds"

      - name: Run tests
        run: npm test
EOF
```

### Step 2: Push

```bash
git add .
git commit -m "Lab 4: caching"
git push
```

### Step 3: Run 1 - the cache miss

Start "Lab 4 - Caching" with the button and open the run.

**You should see:**
- "Which runner am I?" prints a runner name. Write it down.
- "BEFORE the cache step" says `~/.npm does not exist yet (empty runner)`.
- In "Cache npm packages", a line says the cache was not found.
- `Exact cache hit : false`.
- "AFTER the cache step" still says the folder does not exist.
- At the bottom, "Post Cache npm packages" says the cache was saved. This is the moment GitHub copies the folder to its storage.

### Step 4: Run 2 - the cache hit

Start the workflow with the button again. Use a new run, not Re-run, so you also get a fresh runner.

**You should see:**
- A different runner name than in run 1. This is a NEW machine.
- "BEFORE the cache step" says the folder does not exist, because the machine is brand new and empty.
- "Cache npm packages" says the cache was restored from your key.
- `Exact cache hit : true`.
- "AFTER the cache step" shows a folder size, such as `1.2M`. The files came from GitHub's storage, not from the internet.
- "Post Cache npm packages" says it skipped saving, because the exact key already exists.

**This is the answer to "how does the cache work if every runner is new":** the files travel from GitHub's cache storage to the new runner at the start of the job.

### Step 5: See the stored cache

1. In the Actions tab, click Caches in the left sidebar.
2. You will see the saved cache with your key, its size and when it was last used.

### Experiments

**Experiment 1: invalidate the cache by changing the lock file.**

```bash
npm install dayjs
git add .
git commit -m "Add dayjs, lock file changes"
git push
```

Run Lab 4 again. The hash changed, so the key changed. You get `Exact cache hit : false`, but the log may say it restored a cache using `restore-keys` (a partial match). Run it once more and you get `true`.

**Experiment 2: make the saving visible with heavier packages.** Our app has only one small package, so the saving is a few seconds. In a real project the same cache saves minutes. To see a bigger difference:

```bash
npm install --save-dev eslint typescript
git add .
git commit -m "Heavier dependencies"
git push
```

Run Lab 4 twice. Compare the `npm ci took ... seconds` line of the miss run and the hit run.

**Experiment 3: delete the cache and run again.** In the Caches page, delete the cache with the bin icon, then run the workflow. It is a miss again.

### Check yourself

1. Where is the cache stored, on the runner or on GitHub?
2. What makes GitHub decide hit or miss?
3. What happens to the key when you add a package?
4. Why does run 2 skip saving the cache?

Answers: 1) On GitHub's servers. 2) Whether a saved cache with the same key exists. 3) The lock file changes, so the hash and the key change. 4) The exact key already exists, so there is nothing new to save.

---

# Lab 5: Job Dependencies with needs (The Quality Gate)

**Goal:** make sure a deploy job runs only when the test job passes.
**Time:** 15 minutes

**Key setting: `needs: job_name` makes a job wait for another job, and skips it if that job fails.**

**Default Behavior**
- All jobs in a workflow start at the same time, in parallel.
- Without a rule, a deploy job could start while tests are still running.

**What changes with needs**
- It creates an order: test, then build, then deploy.
- If a job fails, every job that needs it is skipped.
- `if: always()` lets a job run even after a failure, which is useful for reports.

**Why it matters**
- Broken code never reaches production.
- It models a real delivery pipeline: Continuous Integration first, then Continuous Delivery.

### Step 1: Create the workflow

```bash
cat > .github/workflows/lab5-needs.yml << 'EOF'
name: Lab 5 - Job Dependencies (needs)

on:
  workflow_dispatch:
    inputs:
      break_tests:
        description: 'Make the test job fail?'
        required: false
        default: false
        type: boolean

jobs:
  lint:
    name: lint (no needs, starts immediately)
    runs-on: ubuntu-latest
    steps:
      - run: echo "Lint check runs in parallel because this job has no needs"

  test:
    runs-on: ubuntu-latest
    steps:
      - name: Run the quality check
        run: |
          echo "Checking the quality of the dish..."
          sleep 5
          if [ "${{ inputs.break_tests }}" = "true" ]; then
            echo "Quality check FAILED"
            exit 1
          fi
          echo "Quality check passed"

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building the app..."

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to production..."

  report:
    needs: [test, build, deploy]
    if: always()
    runs-on: ubuntu-latest
    steps:
      - name: Show the result of every job
        run: |
          echo "test   : ${{ needs.test.result }}"
          echo "build  : ${{ needs.build.result }}"
          echo "deploy : ${{ needs.deploy.result }}"
EOF
```

### Step 2: Push

```bash
git add .
git commit -m "Lab 5: needs"
git push
```

### Step 3: Happy path

Start "Lab 5 - Job Dependencies (needs)" with the checkbox UNTICKED.

**You should see:**
- In the run summary graph, `lint` and `test` start together on the left. Then test, build, deploy and report follow in a chain.
- The report job prints `success` for all three jobs.

### Step 4: Failure path

Start it again with the checkbox TICKED.

**You should see:**
- `test` fails with a red cross.
- `build` and `deploy` show as skipped (grey). The broken code never reached "production".
- `lint` still passes, because it does not depend on test.
- The `report` job still runs because of `if: always()`. It prints `test : failure`, `build : skipped`, `deploy : skipped`.

### Check yourself

1. What happens to a job whose needs-job failed?
2. Why did lint still run in the failure test?
3. What does `if: always()` do?

Answers: 1) It is skipped. 2) It has no needs, so it does not depend on test. 3) The job runs whether earlier jobs passed, failed or were skipped.

---

# Lab 6: Artifacts (Packing the Finished Dish)

**Goal:** pass build output from one job to another and download it yourself.
**Time:** 20 minutes

**Key setting: `upload-artifact` saves files from a job to GitHub, and `download-artifact` brings them into a later job.**

**Default Behavior**
- Every job runs on its own machine, and that machine is deleted when the job ends.
- Files made in the build job are gone, and the next job cannot see them.

**What changes with Artifacts**
- The build job uploads a folder, such as `dist`, under a name.
- Another job downloads that name, so it uses exactly what was built.
- You can also download the artifact by hand from the run page.

**Why it matters**
- You build once and deploy the same files everywhere.
- You keep proof of what was released: reports, logs and installers.

**Cache or Artifact?**

| | Cache | Artifact |
|---|---|---|
| Purpose | Speed up future runs | Pass or keep results of this run |
| Content | Dependencies | Build outputs, reports |
| Used by | The next workflow runs | Later jobs and humans |
| Key idea | Reuse | Deliver |

Artifacts are kept for 90 days by default. You can shorten it with `retention-days`.

### Step 1: Create the workflow

```bash
cat > .github/workflows/lab6-artifacts.yml << 'EOF'
name: Lab 6 - Artifacts

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Install dependencies
        run: npm ci

      - name: Build the app
        run: npm run build

      - name: Show what we built
        run: ls -la dist

      - name: Upload production artifact
        uses: actions/upload-artifact@v4
        with:
          name: site-dist
          path: dist
          retention-days: 7

  without-artifact:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Look for dist (we did NOT download the artifact)
        run: ls dist || echo "RESULT - dist is NOT here. This job is a different machine."

  with-artifact:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download production artifact
        uses: actions/download-artifact@v4
        with:
          name: site-dist
          path: dist

      - name: Look for dist (we DID download the artifact)
        run: |
          ls -la dist
          cat dist/index.html
EOF
```

### Step 2: Push and run

```bash
git add .
git commit -m "Lab 6: artifacts"
git push
```

Start "Lab 6 - Artifacts" with the button.

**You should see:**
1. Job `build` creates `dist/index.html` and uploads it.
2. Job `without-artifact` prints `RESULT - dist is NOT here`. A new machine never has the files of another job.
3. Job `with-artifact` lists `index.html` and prints its content, including the commit id.
4. At the bottom of the run page, an Artifacts section lists `site-dist`.

### Step 3: Download it yourself

1. Click `site-dist` in the Artifacts section. A zip file downloads.
2. Unzip it and open `index.html` in your browser.

### Check yourself

1. Why could the without-artifact job not see `dist`?
2. Cache or artifact: which one is for passing the build result to the deploy job?

Answers: 1) Each job runs on a separate machine that is deleted after the job. 2) Artifact.

---

# Lab 7: Final Project - Ultimate Production Pipeline

**Goal:** combine every concept in one workflow: triggers, matrix, cache, secrets, variables, artifacts and needs.
**Time:** 30 minutes

**Prerequisite:** Lab 1 (the secret `SUPER_SECRET_TOKEN` and the variable `APP_NAME` must exist).

**Key setting: one workflow, three triggers, and a chain of test, build and deploy.**

```
Trigger: push to main, or manual button, or nightly schedule
   |
Test job (matrix: 5 runners, each restoring the cache)
   |   needs: test
Build job (creates dist and uploads it as an artifact)
   |   needs: build
Deploy job (downloads the artifact, uses the secret)
```

| Concept | Where it appears in this lab |
|---|---|
| Variables | `vars.APP_NAME` printed in the deploy log |
| Secrets | `secrets.SUPER_SECRET_TOKEN` masked in the deploy log |
| Triggers | push, workflow_dispatch with dropdown, schedule |
| Matrix | test job, 5 runners |
| Cache | npm cache in the test job |
| needs | test, then build, then deploy |
| Artifacts | `site-dist` from build to deploy |

### Step 1: Create the workflow

```bash
cat > .github/workflows/lab7-ultimate-pipeline.yml << 'EOF'
name: Ultimate Production Pipeline

on:
  push:
    branches:
      - main
  workflow_dispatch:
    inputs:
      environment:
        description: 'Which environment to deploy to?'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
  schedule:
    - cron: '30 20 * * *'

env:
  APP_NAME: ${{ vars.APP_NAME }}

jobs:
  test:
    name: Test on ${{ matrix.os }} with Node ${{ matrix.node }}
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest]
        node: [20, 22, 24]
        exclude:
          - os: windows-latest
            node: 24

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}

      - name: Find npm cache folder
        id: npm-cache-dir
        shell: bash
        run: echo "dir=$(npm config get cache)" >> "$GITHUB_OUTPUT"

      - name: Cache npm packages
        id: cache
        uses: actions/cache@v4
        with:
          path: ${{ steps.npm-cache-dir.outputs.dir }}
          key: ${{ runner.os }}-node${{ matrix.node }}-npm-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node${{ matrix.node }}-npm-

      - name: Was the cache used?
        shell: bash
        run: echo "Cache hit = ${{ steps.cache.outputs.cache-hit || 'false' }}"

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Install dependencies
        run: npm ci

      - name: Build the app
        run: npm run build

      - name: Upload production artifact
        uses: actions/upload-artifact@v4
        with:
          name: site-dist
          path: dist

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment || 'staging' }}
    steps:
      - name: Download production artifact
        uses: actions/download-artifact@v4
        with:
          name: site-dist
          path: dist

      - name: Show what we are deploying
        run: |
          ls -la dist
          cat dist/index.html

      - name: Deploy app
        env:
          API_TOKEN: ${{ secrets.SUPER_SECRET_TOKEN }}
          TARGET: ${{ inputs.environment || 'staging' }}
        run: |
          echo "Deploying $APP_NAME to $TARGET"
          echo "Token length is ${#API_TOKEN} characters"
          echo "Trying to print the token: $API_TOKEN"
EOF
```

### Step 2: Push

```bash
git add .
git commit -m "Lab 7: ultimate production pipeline"
git push
```

From now on, every push to `main` starts this workflow.

### Step 3: Proof steps

1. **Trigger:** the push starts a run. The title says "pushed by".
2. **Matrix and cache:** open the run. The test stage shows 5 parallel jobs. Open any one and read "Was the cache used?". The first run prints `false`.
3. **Cache hit:** start the workflow with the button, choose `production`, and open a test job again. Now it prints `true`.
4. **needs:** the graph shows test, then build, then deploy.
5. **Artifact:** the run page lists `site-dist` at the bottom.
6. **Secret and variable:** in the "Deploy app" step, `APP_NAME` prints as `robo-kitchen` in plain text, and the token prints as `***`.
7. **Dropdown:** on the manual run, the deploy log shows `Deploying robo-kitchen to production`.

### Experiment: break a test and watch needs protect production

1. In `test/kitchen.test.js`, change `90` to `91`.
2. Commit and push:

```bash
git add .
git commit -m "Break a test on purpose"
git push
```

3. The test jobs fail, and build and deploy are skipped.
4. Fix it:

```bash
git revert HEAD --no-edit
git push
```

### Bonus: protect production with an approval

Because the deploy job uses `environment:`, GitHub created two environments for you.

1. Go to Settings, then Environments, and open `production`.
2. Tick Required reviewers and add yourself, then save.
3. Start the workflow with the button and choose `production`.
4. The deploy job now pauses with "Waiting for review". Click Review deployments, tick production, and click Approve and deploy.

Required reviewers on environments may need a public repository or a paid plan for private repositories.

---

# Lab 8: Cleanup

Do this when the class ends.

1. Delete the secret: Settings, Secrets and variables, Actions, delete `SUPER_SECRET_TOKEN`.
2. Delete the variable `APP_NAME` on the Variables tab.
3. Stop the nightly schedule by deleting the schedule block or the workflow file:

```bash
git rm .github/workflows/lab2-triggers.yml .github/workflows/lab7-ultimate-pipeline.yml
git commit -m "Remove scheduled workflows"
git push
```

4. Delete old caches: Actions, Caches.
5. To delete the whole repository: Settings, scroll to the Danger Zone, Delete this repository.

---

# Troubleshooting

| Problem | Likely cause | Fix |
|---|---|---|
| Run workflow button is missing | The file is not on `main`, or it has no `workflow_dispatch` | Push the file to `main`, refresh the page |
| "Invalid workflow file" in red | Indentation error or tab characters | Delete the file and paste the whole block again |
| `npm ci` fails: no package-lock.json | The lock file was not committed | Run `npm install`, then `git add package-lock.json`, commit and push |
| `npm ci` fails: lock file out of sync | `package.json` changed without updating the lock file | Run `npm install`, commit both files |
| Secret prints as empty | Name typo, or the secret was not created | Check the exact name in Settings, Secrets and variables |
| Cache never hits | The lock file changes on every push, or the cache was deleted | Check `Key we asked for` in the log, then run twice with no code change in between |
| Schedule does not run | Not on the default branch, wrong UTC time, or GitHub delay | Check `date -u`, wait 15 minutes, confirm the cron line |
| Two workflows start on one push | Two files listen to push | This is normal. Add `paths:` or `paths-ignore:` to limit one of them |
| `git push` is rejected | Wrong remote URL or SSH not set up | Run `git remote -v` and compare it with your repo, or use the HTTPS URL |
| `sed -i` does not work | Different sed version on your system | Edit the cron line by hand |

---

# Command Cheat Sheet

```bash
# Every lab ends with these three commands
git add .
git commit -m "Describe your change"
git push

# See which files are changed
git status

# Check the remote address
git remote -v

# Show current time in UTC (for cron)
date -u

# Check your Node.js version
node -v
```

---

# Key Takeaways

- **Secrets** keep passwords out of code. Use `${{ secrets.NAME }}` through `env:`, and Variables for non-sensitive settings.
- **Triggers** give control. `workflow_dispatch` adds a manual button with inputs, `schedule` runs on a UTC cron timer, and `paths:` limits push triggers.
- **Caching** stores a folder on GitHub's servers so brand-new runners can restore it. The `hashFiles` key refreshes it when dependencies change.
- **Matrix** tests many combinations in parallel without copied YAML, and `exclude` removes the ones you do not need.
- **needs** guarantees that deploy runs only after tests pass.
- **Artifacts** carry build results between jobs and let you download them later.
- **One push can start many workflows**, and one workflow can run many times. A run always records which event started it.
