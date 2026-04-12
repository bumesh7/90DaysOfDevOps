
### Task 1: Multi-Job Workflow
Create `.github/workflows/multi-job.yml` with 3 jobs:
- `build` — prints "Building the app"
- `test` — prints "Running tests"
- `deploy` — prints "Deploying"

Make `test` run only **after** `build` succeeds.
Make `deploy` run only **after** `test` succeeds.

```
$ vim multi-level.yml

name: multi-level job

on:
 push:
  branches:
   - main

jobs:
 build:
  runs-on: ubuntu-latest
  steps:
   - name: Build app
     run: |
      echo "Building the app"

 test:
  runs-on: ubuntu-latest
  needs: build
  steps:
   - name: Test app
     run: |
      echo "Testing the app"

 deploy:
  runs-on: ubuntu-latest
  needs: test
  steps:
   - name: Deploy app
     run: |
      echo "Deploying the app"

```
**Verify:** Check the workflow graph in the Actions tab — does it show the dependency chain?
<img width="1848" height="618" alt="image" src="https://github.com/user-attachments/assets/1278fd16-e1b2-4dcb-9bb4-aa3e6ee9b564" />


---

### Task 2: Environment Variables
In a new workflow, use environment variables at 3 levels:
1. **Workflow level** — `APP_NAME: myapp`
2. **Job level** — `ENVIRONMENT: staging`
3. **Step level** — `VERSION: 1.0.0`

Print all three in a single step and verify each is accessible.

```
$ vim env-demo.yml

name: Environment Variables Demo

on:
  push:
    branches:
      - main

env:
  APP_NAME: myapp   # Workflow level

jobs:
  env-demo:
    runs-on: ubuntu-latest
    env:
      ENVIRONMENT: staging   # Job level

    steps:
      - name: Print all variables
        env:
          VERSION: 1.0.0   # Step level
        run: |
          echo "App Name: $APP_NAME"
          echo "Environment: $ENVIRONMENT"
          echo "Version: $VERSION"

          echo "Commit SHA: ${{ github.sha }}"
          echo "Triggered by: ${{ github.actor }}"
```
Then use a **GitHub context variable** — print the commit SHA and the actor (who triggered the run).

<img width="1848" height="618" alt="image" src="https://github.com/user-attachments/assets/6e533658-e366-4763-9482-5d7660890ae9" />

---

### Task 3: Job Outputs
1. Create a job that **sets an output** — e.g., today's date as a string
2. Create a second job that **reads that output** and prints it
3. Pass the value using `outputs:` and `needs.<job>.outputs.<name>`
```
$ vim job-outputs.yml

name: Job Outputs Demo

on:
  push:
    branches:
      - main

jobs:
  generate-date:
    runs-on: ubuntu-latest
    outputs:
      today: ${{ steps.set-date.outputs.today }}

    steps:
      - name: Set today's date
        id: set-date
        run: echo "today=$(date +'%Y-%m-%d')" >> $GITHUB_OUTPUT

  use-date:
    runs-on: ubuntu-latest
    needs: generate-date

    steps:
      - name: Print date from previous job
        run: echo "Today's date is ${{ needs.generate-date.outputs.today }}"
```
Write in your notes: Why would you pass outputs between jobs?
```
1. Share Data Across Jobs
Jobs run in separate environments
Outputs are the way to pass data between them

2. Dynamic Workflows
One job decides something (version, tag, date)
Other jobs use that value

3. Real CI/CD Use Cases
Pass build version → deploy job
Pass artifact name → release job
Pass test results → notification job
```
<img width="1848" height="618" alt="image" src="https://github.com/user-attachments/assets/ded2c8bc-b53a-4678-a801-99c3751450f9" />
<img width="1848" height="618" alt="image" src="https://github.com/user-attachments/assets/9bb0d921-3aac-454e-ae1a-052e95af3728" />

---

### Task 4: Conditionals
In a workflow, add:
1. A step that only runs when the branch is `main`
2. A step that only runs when the previous step **failed**
3. A job that only runs on **push** events, not on pull requests
4. A step with `continue-on-error: true` — what does this do?
```
Normally:

If a step fails → job stops

With this:

continue-on-error: true
Step fails 
BUT workflow continues
```

```
name: Conditionals Demo

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  conditional-job:
    runs-on: ubuntu-latest
    if: github.event_name == 'push'   # Runs only on push (not PR)

    steps:
      - name: Step runs only on main branch
        if: github.ref == 'refs/heads/main'
        run: echo "This runs only on main branch"

      - name: Step that fails intentionally
        #continue-on-error: true
        run: exit 1

      - name: Run only if previous step failed
        if: failure()
        run: echo "Previous step failed, so this runs"

      - name: Continue even if error occurs
        continue-on-error: true
        run: |
          echo "This step will fail but workflow continues"
          exit 1

      - name: This still runs because of continue-on-error
        run: echo "Workflow did not stop"

```
<img width="1853" height="905" alt="image" src="https://github.com/user-attachments/assets/cae23745-fd92-46d8-b5a8-fa675f2982e5" />

---

### Task 5: Putting It Together
Create `.github/workflows/smart-pipeline.yml` that:
1. Triggers on push to any branch
2. Has a `lint` job and a `test` job running in parallel
3. Has a `summary` job that runs after both, prints whether it's a `main` branch push or a feature branch push, and prints the commit message

```
$ vim smart-pipeline

name: Smart Pipeline

on:
  push:   # triggers on push to any branch

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - name: Lint step
        run: echo "Running lint checks"

  test:
    runs-on: ubuntu-latest
    steps:
      - name: Test step
        run: echo "Running tests"

  summary:
    runs-on: ubuntu-latest
    needs: [lint, test]   # waits for both jobs

    steps:
      - name: Print branch type
        run: |
          if [[ "${GITHUB_REF}" == "refs/heads/main" ]]; then
            echo "This is a MAIN branch push"
          else
            echo "This is a FEATURE branch push"
          fi

      - name: Print commit message
        run: |
         echo "Commit message: ${{ github.event.head_commit.message }}"
~                                                                      
```
<img width="1852" height="706" alt="image" src="https://github.com/user-attachments/assets/82a7d52c-3c71-4e03-92b6-d17b7c2833f4" />

<img width="1852" height="706" alt="image" src="https://github.com/user-attachments/assets/ac1587b9-6f6e-41f0-9288-9de7c94c5218" />

---

