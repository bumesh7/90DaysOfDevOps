### Task 1: Trigger on Pull Request
1. Create `.github/workflows/pr-check.yml`
```
$ cd .github/workflows
$ vim pr-check.yml
```
2. Trigger it only when a pull request is **opened or updated** against `main`
```
name: PR Check

on:
  pull_request:
    branches:
      - main
    types: [opened, synchronize]

jobs:
  pr-check:
    runs-on: ubuntu-latest

    steps:
      - name: Print PR info
        run: | 
         echo "The branch that triggered this run is $GITHUB_REF_NAME"
         echo "PR check running for branch: ${{ github.head_ref }}"
```
```
pull_request → triggers on PR events
branches: main → only PRs targeting main
opened → when PR is created
synchronize → when new commits are pushed to PR
GITHUB_REF_NAME → gives your branch name
```
3. Add a step that prints: `PR check running for branch: <branch name>`
4. Create a new branch, push a commit, and open a PR
```
$ git checkout -b feature/test-pr

echo "test change" >> test.txt
git add .
git commit -m "test PR workflow"
git push origin feature/test-pr
```
<img width="1847" height="387" alt="image" src="https://github.com/user-attachments/assets/dddf5da2-876a-44e8-ba29-f1aa6c4b7bd6" />

```
-> when everything is messed in you local by multiple commits the fetch the original data from the github

git fetch origin
git reset --hard origin/main
```

5. Watch the workflow run automatically
<img width="1848" height="548" alt="image" src="https://github.com/user-attachments/assets/4dcf66c4-b19b-4f41-82f6-e99535f20af9" />

Does it show up on the PR page?
<img width="1849" height="450" alt="image" src="https://github.com/user-attachments/assets/aaba55ab-bf6c-4a32-b5ee-4fa26ab2cdfc" />

---

### Task 2: Scheduled Trigger
1. Add a `schedule:` trigger to any workflow using cron syntax
2. Set it to run every day at midnight UTC
```
name: PR Check + Schedule

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
    types: [opened, synchronize]

  schedule:
    - cron: "0 0 * * *"   # runs every day at midnight UTC

jobs:
  pr-check:
    runs-on: ubuntu-latest

    steps:
      - name: Scheduled or PR run
        run: |
          echo "Workflow triggered"
          echo "Event: ${{ github.event_name }}"

```
3. Write in your notes: What is the cron expression for every Monday at 9 AM?
```
* * * * *
│ │ │ │ │
│ │ │ │ └── Day of week (0-7)
│ │ │ └──── Month
│ │ └────── Day of month
│ └──────── Hour
└────────── Minute

corn every monday = 0 9 * * 1
```
<img width="1849" height="548" alt="image" src="https://github.com/user-attachments/assets/dfe3cccf-08a0-43da-b9db-2291b81b3bf2" />

---

### Task 3: Manual Trigger
1. Create `.github/workflows/manual.yml` with a `workflow_dispatch:` trigger
2. Add an **input** that asks for an `environment` name (staging/production)
```
name: Manual Workflow

on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Select environment"
        required: true
        default: "staging"
        type: choice
        options:
          - staging
          - production

jobs:
  manual-job:
    runs-on: ubuntu-latest

    steps:
      - name: Print environment
        run: echo "Selected environment is ${{ github.event.inputs.environment }}"
```
3. Print the input value in a step
<img width="1849" height="648" alt="image" src="https://github.com/user-attachments/assets/8378e8eb-3af7-4171-bea5-67f47cdbf037" />

4. Go to the **Actions** tab → find the workflow → click **Run workflow**
<img width="1849" height="648" alt="image" src="https://github.com/user-attachments/assets/2f9bb93b-9f96-4889-953d-e9b77847dcec" />

**Verify:** Can you trigger it manually and see your input printed?
```
Yes manually we need to trigger the input and workflow will run

workflow_dispatch → enables manual trigger 
inputs.environment → dropdown input (staging/production)
${{ github.event.inputs.environment }} → reads your input
```

---

### Task 4: Matrix Builds
Create `.github/workflows/matrix.yml` that:
1. Uses a matrix strategy to run the same job across:
   - Python versions: `3.10`, `3.11`, `3.12`
```
name: Matrix Build

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        python-version: [3.11, 3.12, 3.13]

    steps:
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}

      - name: Print Python version
        run: python --version
```

<img width="1849" height="648" alt="image" src="https://github.com/user-attachments/assets/265cbc0e-1559-4a39-875a-438f3967a0f1" />

2. Each job installs Python and prints the version
<img width="1849" height="741" alt="image" src="https://github.com/user-attachments/assets/d79dbd7c-6c85-4d6d-ba38-9ee6bc62480f" />

3. Watch all 3 run in parallel

Then extend the matrix to also include 2 operating systems — how many total jobs run now?

```
Total 6 jons will run
3 python version for windows
3 python versions for ubuntu
```
```
name: Matrix Build

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ${{ matrix.os }}

    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        python-version: [3.11, 3.12, 3.13]

    steps:
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}

      - name: Print Python version
        run: python --version
```
<img width="1849" height="794" alt="image" src="https://github.com/user-attachments/assets/3b075db7-46df-43d0-8dea-4b4fc9b11eea" />

---

### Task 5: Exclude & Fail-Fast
1. In your matrix, **exclude** one specific combination (e.g., Python 3.10 on Windows)
2. Set `fail-fast: false` — trigger a failure in one job and observe what happens to the rest
```
name: Matrix Build (Exclude + Fail Fast)

on:
  push:
    branches: [main]

jobs:
  build:
    name: Python ${{ matrix.python-version }} on ${{ matrix.os }}
    runs-on: ${{ matrix.os }}

    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest]
        python-version: [3.11, 3.12, 3.13]

        exclude:
          - os: windows-latest
            python-version: 3.11

    steps:
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}

      - name: Print Python version
        run: python --version

      # Force a failure for testing
      - name: Force failure on Python 3.12
        if: matrix.python-version == '3.12'
        run: exit 1

```
<img width="1849" height="794" alt="image" src="https://github.com/user-attachments/assets/1bad5752-0818-40e1-a782-88cc7a13aa56" />

3. Write in your notes: What does `fail-fast: true` (the default) do vs `false`?
```
If fail-fast: true => one job fails automatically all jobs will be stop the execution.
```
<img width="1849" height="794" alt="image" src="https://github.com/user-attachments/assets/a1e7584f-25f5-4e92-a3d8-ea6aa5b9d25e" />

---
