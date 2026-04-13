### Task 1: GitHub Secrets
1. Go to your repo → Settings → Secrets and Variables → Actions
2. Create a secret called `MY_SECRET_MESSAGE`
3. Create a workflow that reads it and prints: `The secret is set: true` (never print the actual value)
4. Try to print `${{ secrets.MY_SECRET_MESSAGE }}` directly — what does GitHub show?
```
$ vim secrets.yml

name: Secret Check

on: push

jobs:
  check-secret:
    runs-on: ubuntu-latest

    steps:
      - name: Check if secret is set
        run: |
          if [ -n "${{ secrets.MY_SECRET_MESSAGE }}" ]; then
            echo "The secret is set: true"
          else
            echo "The secret is set: false"
          fi

      - name: Try printing secret (not recommended)
        run: |
         echo "${{ secrets.MY_SECRET_MESSAGE }}"
```
<img width="1848" height="692" alt="image" src="https://github.com/user-attachments/assets/cdab4db4-5201-49be-b76d-ada43dc5ced8" />

```
If you try:

echo "${{ secrets.MY_SECRET_MESSAGE }}"

👉 GitHub masks the value in logs, so you will see:

***

✔️ This is a built-in security feature of GitHub Actions.
```
Write in your notes: Why should you never print secrets in CI logs?

```
Logs are visible to anyone with repo access (or even public in open repos)
Secrets can leak if masking fails or logs are exported
Attackers can misuse exposed credentials (API keys, tokens, passwords)
Logs may be stored for a long time → increases risk of exposure
Violates security best practices and compliance standards
```
---

### Task 2: Use Secrets as Environment Variables
1. Pass a secret to a step as an environment variable
2. Use it in a shell command without ever hardcoding it
3. Add `DOCKER_USERNAME` and `DOCKER_TOKEN` as secrets (you'll need these on Day 45)
```
$ vim secrets-env.yml

name: Use Secrets as Env Vars

on: push

jobs:
  use-secrets:
    runs-on: ubuntu-latest

    steps:
      - name: Use secret as environment variable
        env:
          MY_SECRET: ${{ secrets.MY_SECRET_MESSAGE }}
        run: |
          echo "Checking secret..."
          if [ -n "$MY_SECRET" ]; then
            echo "Secret is available!"
          else
            echo "Secret is missing!"
          fi

      - name: Simulate Docker login (safe usage)
        env:
          DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
          DOCKER_TOKEN: ${{ secrets.DOCKER_TOKEN }}
        run: |
          echo "Logging into Docker..."
          echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USERNAME" --password-stdin
```
<img width="1848" height="804" alt="image" src="https://github.com/user-attachments/assets/f245fd0d-3cf7-4c83-91e5-d0c6fb346fee" />

---

### Task 3: Upload Artifacts
1. Create a step that generates a file — e.g., a test report or a log file
2. Use `actions/upload-artifact` to save it
3. After the workflow runs, download the artifact from the Actions tab
```
$ vim artifact.yml

name: Upload Artifact Demo

on: push

jobs:
  artifact-job:
    runs-on: ubuntu-latest

    steps:
      - name: Generate a report file
        run: |
          echo "Test Report - $(date)" > report.txt
          echo "All tests passed" >> report.txt

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: test-report
          path: report.txt
```
**Verify:** Can you see and download it from GitHub?
<img width="1853" height="890" alt="image" src="https://github.com/user-attachments/assets/bfb97879-33f8-49ed-8af1-a01aaf723e50" />
```
How to Download the Artifact

Go to your repo → Actions tab
Click on the latest workflow run
Scroll down to Artifacts
Click test-report
Download the .zip file

Verification

✔️ Yes — you should be able to:

See the artifact listed in the workflow run
Download it as a .zip
Open it locally and view report.txt

Why Artifacts Matter

Store build outputs (logs, reports, binaries)
Share files between jobs
Debug failures using saved logs
Useful in CI/CD pipelines (testing, deployments)
```
---

### Task 4: Download Artifacts Between Jobs
1. Job 1: generate a file and upload it as an artifact
2. Job 2: download the artifact from Job 1 and use it (print its contents)

```
$ vim artifact-bw-jobs.yml

name: Artifact Sharing Between Jobs

on: push

jobs:
  job1:
    runs-on: ubuntu-latest

    steps:
      - name: Generate file
        run: |
          echo "Hello from Job 1" > message.txt

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: shared-file
          path: message.txt

  job2:
    runs-on: ubuntu-latest
    needs: job1   # ensures job1 runs first

    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: shared-file

      - name: Read file
        run: |
          echo "Contents of file:"
          cat message.txt
```
Write in your notes: When would you use artifacts in a real pipeline?
```
📦 Build → Deploy
Build app in one job, deploy in another
🧪 Test Reports
Generate test results, then analyze or publish them
🔍 Debugging
Save logs from failed jobs for investigation
🏗️ Multi-stage pipelines
Share compiled binaries, Docker images metadata, etc.
📊 Security scans / reports
Pass scan results between jobs
```
<img width="1853" height="966" alt="image" src="https://github.com/user-attachments/assets/ac5696a8-beda-46f9-a10c-3e61690a89f8" />

---

### Task 5: Run Real Tests in CI
Take any script from your earlier days (Python or Shell) and run it in CI:
1. Add your script to the `github-actions-practice` repo
2. Write a workflow that:
   - Checks out the code
   - Installs any dependencies needed
   - Runs the script
   - Fails the pipeline if the script exits with a non-zero code
3. Intentionally break the script — verify the pipeline goes red
4. Fix it — verify it goes green again

```
$ vim script.sh

#!/bin/bash

echo "Running the scripts"

exit 0
```
```

$ vim script-workflow.yml

name: Run Shell Script

on: push

jobs:
  test-script:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Make script executable
        run: chmod +x .github/workflows/script.sh

      - name: Run script
        run: .github/workflows/script.sh

```
<img width="1836" height="812" alt="image" src="https://github.com/user-attachments/assets/97686ce5-1baf-4509-86c6-2dcd74d6e963" />
```
use exit 1 => to fail the pipeline and fix again
```
<img width="1836" height="812" alt="image" src="https://github.com/user-attachments/assets/f13d627c-a0f2-4fa8-8600-6d5e9a3f84ab" />

---

### Task 6: Caching
1. Add `actions/cache` to a workflow that installs dependencies
2. Run it twice — observe the time difference
```
$ pwd => /home/umesh/Documents/github_workflows/github-actions-practice/.github/workflows

$ cd ../../

$ vim requirements.txt

requests==2.31.0
pandas>=2.0.0
flask
```
```
$ vim cache.yml

name: Cache Demo

on: push

jobs:
  cache-job:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Debug files
        run: ls -la

      - name: Cache dependencies
        uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: cache-${{ hashFiles('requirements.txt') }}

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run script
        run: echo "Caching works!"
```

<img width="1839" height="962" alt="image" src="https://github.com/user-attachments/assets/af117559-d35d-44c4-8cae-c9c5f0933e92" />

3. Write in your notes: What is being cached and where is it stored?
```
GitHub saves downloaded dependencies in its own storage and reuses them in future runs to make workflows faster.
stores in path
~/.cache/pip
```
---

