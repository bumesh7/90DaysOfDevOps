### Task 1: Pull Request Event Types
Create `.github/workflows/pr-lifecycle.yml` that triggers on `pull_request` with **specific activity types**:
1. Trigger on: `opened`, `synchronize`, `reopened`, `closed`
2. Add steps that:
   - Print which event type fired: `${{ github.event.action }}`
   - Print the PR title: `${{ github.event.pull_request.title }}`
   - Print the PR author: `${{ github.event.pull_request.user.login }}`
   - Print the source branch and target branch
3. Add a conditional step that only runs when the PR is **merged** (closed + merged = true)
```
$ vim pr-lifecycle.yml

name: PR Lifecycle Workflow

on:
  pull_request:
    types: [opened, synchronize, reopened, closed]

jobs:
  pr-info:
    runs-on: ubuntu-latest

    steps:
      - name: Print PR Event Action
        run: |
          echo "Event Type: ${{ github.event.action }}"

      - name: Print PR Title
        run: |
          echo "PR Title: ${{ github.event.pull_request.title }}"

      - name: Print PR Author
        run: |
          echo "PR Author: ${{ github.event.pull_request.user.login }}"

      - name: Print Source and Target Branch
        run: |
          echo "Source Branch: ${{ github.head_ref }}"
          echo "Target Branch: ${{ github.base_ref }}"

      - name: Run only when PR is merged
        if: github.event.pull_request.merged == true
        run: |
          echo "PR has been merged!"

```
```
$ git add .
$ git commit -m "added pr-lifecycle.yml"
$ git push origin main
```
```
git checkout -b feature/test-pr
echo "test change" >> test.txt
git add .
git commit -m "Test PR lifecycle"
git push origin feature/test-pr
```
```
Create Pull Request (GitHub UI)

Go to your repo on GitHub
Click Compare & Pull Request
Create PR → triggers opened
```
<img width="1828" height="922" alt="image" src="https://github.com/user-attachments/assets/12eef04c-c70e-4005-ac7b-899426b8d6af" />

```
echo "another change" >> test.txt
git add .
git commit -m "Update PR"
git push origin feature/test-pr
```
Test it: create a PR, push an update to it, then merge it. Watch the workflow fire each time with a different event type.
```
Merge PR
```
<img width="1828" height="922" alt="image" src="https://github.com/user-attachments/assets/0791a3df-b9cc-4061-964d-32da2f627473" />
<img width="1828" height="922" alt="image" src="https://github.com/user-attachments/assets/f6ee63db-ef16-4b6d-98b2-a50972d2033d" />

---

### Task 2: PR Validation Workflow
Create `.github/workflows/pr-checks.yml` — a real-world PR gate:
1. Trigger on `pull_request` to `main`
2. Add a job `file-size-check` that:
   - Checks out the code
   - Fails if any file in the PR is larger than 1 MB
3. Add a job `branch-name-check` that:
   - Reads the branch name from `${{ github.head_ref }}`
   - Fails if it doesn't follow the pattern `feature/*`, `fix/*`, or `docs/*`
4. Add a job `pr-body-check` that:
   - Reads the PR body: `${{ github.event.pull_request.body }}`
   - Warns (but doesn't fail) if the PR description is empty
```
$ vim pr-checks.yml

name: PR Validation Checks

on:
  pull_request:
    branches:
      - main

jobs:
  file-size-check:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Check for large files (>1MB)
        run: |
          echo "Checking file sizes..."
          FAIL=0
          while IFS= read -r file; do
            if [ -f "$file" ]; then
              size=$(stat -c%s "$file")
              if [ "$size" -gt 1048576 ]; then
                echo "❌ File too large: $file ($(($size / 1024)) KB)"
                FAIL=1
              fi
            fi
          done < <(git diff --name-only origin/${{ github.base_ref }})
          
          if [ "$FAIL" -eq 1 ]; then
            echo "❌ Some files exceed 1MB limit"
            exit 1
          else
            echo "✅ All files are within size limit"
          fi

  branch-name-check:
    runs-on: ubuntu-latest

    steps:
      - name: Validate branch name
        run: |
          echo "Branch: ${{ github.head_ref }}"
          if [[ "${{ github.head_ref }}" =~ ^(feature|fix|docs)/.+ ]]; then
            echo "✅ Branch name is valid"
          else
            echo "❌ Invalid branch name. Use feature/*, fix/*, or docs/*"
            exit 1
          fi

  pr-body-check:
    runs-on: ubuntu-latest

    steps:
      - name: Check PR description
        run: |
          if [ -z "${{ github.event.pull_request.body }}" ]; then
            echo "⚠️ Warning: PR description is empty"
          else
            echo "✅ PR description is present"
          fi
```
```
Commit

git add .
git commit -m "Add PR validation workflow"
git push origin main
```
```
create bad branch the check will fail

git checkout -b wrong-branch-name
echo "test" > file.txt
git add .
git commit -m "Bad branch test"
git push origin wrong-branch-name
```
```
create valid branch as feature/*

git checkout -b feature/test-validation
echo "valid change" > valid.txt
git add .
git commit -m "Valid PR"
git push origin feature/test-validation
```
```
Create PR without description

Warning appears (but workflow passes)
```

**Verify:** Open a PR from a badly named branch — does the check fail?
```
The error was due to workflow code issue now its fixed so it will run
```
<img width="1828" height="735" alt="image" src="https://github.com/user-attachments/assets/eab72a6c-5814-43b0-a4ee-f439d58f3a27" />

---

### Task 3: Scheduled Workflows (Cron Deep Dive)
Create `.github/workflows/scheduled-tasks.yml`:
1. Add a `schedule` trigger with cron: `'30 2 * * 1'` (every Monday at 2:30 AM UTC)
2. Add **another** cron entry: `'0 */6 * * *'` (every 6 hours)
3. In the job, print which schedule triggered using `${{ github.event.schedule }}`
4. Add a step that acts as a **health check** — curl a URL and check the response code

Write in your notes:
- The cron expression for: every weekday at 9 AM IST
- The cron expression for: first day of every month at midnight
- Why GitHub says scheduled workflows may be delayed or skipped on inactive repos

**Important:** Also add `workflow_dispatch` so you can test it manually without waiting for the schedule.

---

### Task 4: Path & Branch Filters
Create `.github/workflows/smart-triggers.yml`:
1. Trigger on push but **only** when files in `src/` or `app/` change:
   ```yaml
   on:
     push:
       paths:
         - 'src/**'
         - 'app/**'
   ```
2. Add `paths-ignore` in a second workflow that skips runs when only docs change:
   ```yaml
   paths-ignore:
     - '*.md'
     - 'docs/**'
   ```
3. Add branch filters to only trigger on `main` and `release/*` branches
4. Test it: push a change to a `.md` file — does the workflow skip?

Write in your notes: When would you use `paths` vs `paths-ignore`?

---

### Task 5: `workflow_run` — Chain Workflows Together
Create two workflows:
1. `.github/workflows/tests.yml` — runs tests on every push
2. `.github/workflows/deploy-after-tests.yml` — triggers **only after** `tests.yml` completes successfully:
   ```yaml
   on:
     workflow_run:
       workflows: ["Run Tests"]
       types: [completed]
   ```
3. In the deploy workflow, add a conditional:
   - Only proceed if the triggering workflow **succeeded** (`${{ github.event.workflow_run.conclusion == 'success' }}`)
   - Print a warning and exit if it failed

**Verify:** Push a commit — does the test workflow run first, then trigger the deploy workflow?

---

### Task 6: `repository_dispatch` — External Event Triggers
1. Create `.github/workflows/external-trigger.yml` with trigger `repository_dispatch`
2. Set it to respond to event type: `deploy-request`
3. Print the client payload: `${{ github.event.client_payload.environment }}`
4. Trigger it using `curl` or `gh`:
   ```bash
   gh api repos/<owner>/<repo>/dispatches \
     -f event_type=deploy-request \
     -f client_payload='{"environment":"production"}'
   ```

Write in your notes: When would an external system (like a Slack bot or monitoring tool) trigger a pipeline?

---

## Hints
- PR merge check: `if: github.event.pull_request.merged == true`
- Cron syntax: `minute hour day-of-month month day-of-week`
- Scheduled workflows only run on the **default branch**
- `workflow_run` gives you access to the triggering workflow's conclusion and artifacts
- `repository_dispatch` requires a personal access token with `repo` scope
- Path filters use glob patterns — `**` matches nested directories

---

## Documentation
Create `day-47-advanced-triggers.md` with:
- Your workflow YAML files
- The cron expressions from Task 3
- Screenshot of the PR checks running on a pull request
- Explanation of `workflow_run` vs `workflow_call` in your own words

---

## Submission
1. Add `day-47-advanced-triggers.md` to `2026/day-47/`
2. Commit and push to your fork

---

## Learn in Public
Share your PR validation workflow on LinkedIn — automated PR gates are a real DevOps flex.

`#90DaysOfDevOps` `#DevOpsKaJosh` `#TrainWithShubham`

Happy Learning!
**TrainWithShubham**
