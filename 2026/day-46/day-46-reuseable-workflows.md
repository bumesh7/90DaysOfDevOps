### Task 1: Understand `workflow_call`
Before writing any code, research and answer in your notes:
1. What is a **reusable workflow**?
```
A reusable workflow in GitHub Actions is a workflow that can be called by other workflows.
Instead of duplicating the same jobs (like build, test, deploy) across multiple workflows, you define them once and reuse them.

Think of it like a function for CI/CD pipelines.
```
2. What is the `workflow_call` trigger?
```
workflow_call is a special trigger that allows a workflow to be invoked by another workflow.

Example idea:

on:
  workflow_call:

It also allows:

Passing inputs
Passing secrets
Returning outputs
```
3. How is calling a reusable workflow different from using a regular action (`uses:`)?
```
Reusable Workflow

Calls an entire workflow (multiple jobs + steps)
Defined using workflow_call
Can include multiple jobs
ex: uses: owner/repo/.github/workflows/workflow.yml@main

Regular Action (uses:)

Calls a single reusable step
Usually from GitHub Marketplace or custom action
Works inside a step, not at job level

ex: uses: actions/checkout@v4
```
4. Where must a reusable workflow file live?
```
A reusable workflow must be stored in:

.github/workflows/
```
---

### Task 2: Create Your First Reusable Workflow
Create `.github/workflows/reusable-build.yml`:
1. Set the trigger to `workflow_call`
2. Add an `inputs:` section with:
   - `app_name` (string, required)
   - `environment` (string, required, default: `staging`)
3. Add a `secrets:` section with:
   - `docker_token` (required)
4. Create a job that:
   - Checks out the code
   - Prints `Building <app_name> for <environment>`
   - Prints `Docker token is set: true` (never print the actual secret)
```
$ vim reusable-build.yml

name: Reusable Build Workflow

on:
  workflow_call:
    inputs:
      app_name:
        description: "Name of the application"
        required: true
        type: string
      environment:
        description: "Deployment environment"
        required: true
        type: string
        default: staging
    secrets:
      docker_token:
        description: "Docker authentication token"
        required: true

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Print build info
        run: echo "Building ${{ inputs.app_name }} for ${{ inputs.environment }}"

      - name: Verify Docker token
        run: echo "Docker token is set: ${{ secrets.docker_token != '' }}"
```
**Verify:** This file alone won't run — it needs a caller. That's next.
```
The flow wont run it needs caller
```
---

### Task 3: Create a Caller Workflow
Create `.github/workflows/call-build.yml`:
1. Trigger on push to `main`
2. Add a job that uses your reusable workflow:
   ```yaml
   jobs:
     build:
       uses: ./.github/workflows/reusable-build.yml
       with:
         app_name: "my-web-app"
         environment: "production"
       secrets:
         docker_token: ${{ secrets.DOCKER_TOKEN }}
   ```
3. Push to `main` and watch it run
```
$ vim call-build.yml

name: Call Build Workflow

on:
  push:
    branches:
      - main

jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml

    with:
      app_name: "my-web-app"
      environment: "production"

    secrets:
      docker_token: ${{ secrets.DOCKER_TOKEN }}
```
**Verify:** In the Actions tab, do you see the caller triggering the reusable workflow? Click into the job — can you see the inputs printed?
<img width="1839" height="906" alt="image" src="https://github.com/user-attachments/assets/cc2af02a-dbf3-4eea-a4bd-e9670837161b" />

---

### Task 4: Add Outputs to the Reusable Workflow
Extend `reusable-build.yml`:
1. Add an `outputs:` section that exposes a `build_version` value
2. Inside the job, generate a version string (e.g., `v1.0-<short-sha>`) and set it as output
```
$ vim reusable-build.yml

name: Reusable Build Workflow

on:
  workflow_call:
    inputs:
      app_name:
        required: true
        type: string
      environment:
        required: true
        type: string
        default: staging
    secrets:
      docker_token:
        required: true
    outputs:
      build_version:
        description: "Generated build version"
        value: ${{ jobs.build.outputs.build_version }}

jobs:
  build:
    runs-on: ubuntu-latest

    outputs:
      build_version: ${{ steps.set_version.outputs.build_version }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Generate version
        id: set_version
        run: |
          VERSION="v1.0-${GITHUB_SHA::7}"
          echo "build_version=$VERSION" >> $GITHUB_OUTPUT

      - name: Print build info
        run: echo "Building ${{ inputs.app_name }} for ${{ inputs.environment }}"

      - name: Verify Docker token
        run: echo "Docker token is set: ${{ secrets.docker_token != '' }}"
```
3. In your caller workflow, add a second job that:
   - Depends on the build job (`needs:`)
   - Reads and prints the `build_version` output
```
$ vim call-build.yml

name: Call Build Workflow

on:
  push:
    branches:
      - main

jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml

    with:
      app_name: "my-web-app"
      environment: "production"

    secrets:
      docker_token: ${{ secrets.DOCKER_TOKEN }}

  print-version:
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Print build version
        run: echo "Build version is ${{ needs.build.outputs.build_version }}"
```
**Verify:** Does the second job print the version from the reusable workflow?
<img width="1833" height="652" alt="image" src="https://github.com/user-attachments/assets/87fd6b35-b082-4251-8ada-8cb504b2069c" />

<img width="1833" height="652" alt="image" src="https://github.com/user-attachments/assets/6d3f394b-a0d4-4e41-900c-9d85e327624f" />

---

### Task 5: Create a Composite Action
Create a **custom composite action** in your repo at `.github/actions/setup-and-greet/action.yml`:
1. Define inputs: `name` and `language` (default: `en`)
2. Add steps that:
   - Print a greeting in the specified language
   - Print the current date and runner OS
   - Set an output called `greeted` with value `true`
3. Use the composite action in a new workflow with `uses: ./.github/actions/setup-and-greet`
```
$ pwd => /home/umesh/Documents/github_workflows/github-actions-practice/.github/actions/setup-and-greet

name: "Greet Action"

inputs:
  name:
    required: true
  language:
    required: false
    default: "en"

outputs:
  greeted:
    value: ${{ steps.out.outputs.greeted }}

runs:
  using: "composite"
  steps:
    - name: Greet
      run: echo "Hello ${{ inputs.name }}"
      shell: bash

    - name: Print info
      run: |
        date
        echo $RUNNER_OS
      shell: bash

    - name: Set output
      id: out
      run: echo "greeted=true" >> $GITHUB_OUTPUT
      shell: bash
```
```
pwd => /home/umesh/Documents/github_workflows/github-actions-practice/.github/workflows/use-action.yml

name: Use Action

on:
  push:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Run action
        id: greet
        uses: ./.github/actions/setup-and-greet
        with:
          name: Umesh

      - name: Check output
        run: echo "Greeted: ${{ steps.greet.outputs.greeted }}" 
```
**Verify:** Does your custom action run and print the greeting?
<img width="1841" height="919" alt="image" src="https://github.com/user-attachments/assets/6a323c2e-8ca4-4e25-9ba9-4405d61d0331" />

---

### Task 6: Reusable Workflow vs Composite Action
Fill this in your notes:

| | Reusable Workflow | Composite Action |
|---|---|---|
| Triggered by | `workflow_call` | `uses:` in a step |
| Can contain jobs? | Yes (multiple jobs) | No (only steps) |
| Can contain multiple steps? | Yes | Yes |
| Lives where? | `.github/workflows/` | `.github/actions/<action-name>/` |
| Can accept secrets directly? | Yes | No (must be passed as inputs) |
| Best for | Full CI/CD pipelines (build, test, deploy) | Reusable step logic (scripts, setup tasks) |
---

## Hints
- Reusable workflows must be in `.github/workflows/` directory
- Caller syntax: `uses: ./.github/workflows/file.yml` (same repo) or `uses: org/repo/.github/workflows/file.yml@main` (cross-repo)
- Composite action: `action.yml` with `runs: using: "composite"`
- Reusable workflow outputs: `on: workflow_call: outputs: name: value: ${{ jobs.job-id.outputs.name }}`
- A reusable workflow can be called by at most 20 unique caller workflows in a single run

---
