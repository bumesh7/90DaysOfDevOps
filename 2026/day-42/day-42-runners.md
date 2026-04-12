### Task 1: GitHub-Hosted Runners
1. Create a workflow with 3 jobs, each on a different OS:
   - `ubuntu-latest`
   - `windows-latest`
   - `macos-latest`
2. In each job, print:
   - The OS name
   - The runner's hostname
   - The current user running the job
3. Watch all 3 run in parallel

```
name: Runner Info

on:
  push:
    branches: [main]

jobs:
  ubuntu-job:
    runs-on: ubuntu-latest
    steps:
      - name: Print system info (Ubuntu)
        run: |
          echo "OS: Ubuntu"
          echo "Hostname: $(hostname)"
          echo "User: $(whoami)"

  windows-job:
    runs-on: windows-latest
    steps:
      - name: Print system info (Windows)
        run: |
          echo "OS: Windows"
          hostname
          whoami

  macos-job:
    runs-on: macos-latest
    steps:
      - name: Print system info (macOS)
        run: |
          echo "OS: macOS"
          hostname
          whoami
```

Write in your notes: What is a GitHub-hosted runner? Who manages it?
```
A GitHub-hosted runner is a virtual machine provided by GitHub that runs your workflows.

It comes pre-installed with tools (Git, Node, Python, etc.)
It is created fresh for each job
It is automatically destroyed after the job finishes

GitHub manages everything:

Infrastructure (servers, scaling)
OS setup
Security updates
Cleanup after each run
```
<img width="1849" height="794" alt="image" src="https://github.com/user-attachments/assets/e10bb496-6c4c-4fa8-9df2-8f9fae30736b" />
<img width="472" height="385" alt="image" src="https://github.com/user-attachments/assets/a376122a-8da6-4c3f-aeec-f20be2b355fd" />
<img width="472" height="385" alt="image" src="https://github.com/user-attachments/assets/adfaf9b6-d3c2-4cf5-9023-3e9895315c1d" />
<img width="629" height="382" alt="image" src="https://github.com/user-attachments/assets/2a43d6d4-185b-42bf-b979-2838c276fd60" />

---

### Task 2: Explore What's Pre-installed
1. On the `ubuntu-latest` runner, run a step that prints:
   - Docker version
   - Python version
   - Node version
   - Git version
2. Look up the GitHub docs for the full list of pre-installed software on `ubuntu-latest`
```
$ vim pre-installed.yml

name: Pre-Installed Checks

on:
 push:
  branches:
   - main

jobs:
 explore-tools:
  runs-on: ubuntu-latest

  steps:
   - name: Print installed tool versions
     run: |
      echo: "Docker Version"
      docker --version

      echo: "Python Version"
      python3 --version

      echo: "Node Version"
      node --version

      echo: "Git Version"
      git --version

```
<img width="1850" height="682" alt="image" src="https://github.com/user-attachments/assets/b0eb9595-ae08-4105-ad11-89c55451fc8c" />

Write in your notes: Why does it matter that runners come with tools pre-installed?
```
1. Faster Workflows
No need to install Docker, Node, Python every time
Saves minutes on every run
2. Cost Efficient
GitHub Actions billing is time-based
Less setup time = lower cost
3. Consistency
Every run uses the same environment
Avoids “it works on my machine” problems
4. Simpler YAML
No long setup scripts
Cleaner, easier-to-maintain workflows
5. Ready for CI/CD
Most common tools are already available
You can focus on build, test, deploy
```

---

### Task 3: Set Up a Self-Hosted Runner
1. Go to your GitHub repo → Settings → Actions → Runners → **New self-hosted runner**
2. Choose Linux as the OS
3. Follow the instructions to download and configure the runner on:
   - Your local machine, OR
   - A cloud VM (EC2, Utho, or any VPS)
4. Start the runner — verify it shows as **Idle** in GitHub
```
Download

# Create a folder
$ mkdir actions-runner && cd actions-runner
# Download the latest runner package
$ curl -o actions-runner-linux-x64-2.333.1.tar.gz -L https://github.com/actions/runner/releases/download/v2.333.1/actions-runner-linux-x64-2.333.1.tar.gz
# Optional: Validate the hash
$ echo "18f8f68ed1892854ff2ab1bab4fcaa2f5abeedc98093b6cb13638991725cab74  actions-runner-linux-x64-2.333.1.tar.gz" | shasum -a 256 -c
# Extract the installer
$ tar xzf ./actions-runner-linux-x64-2.333.1.tar.gz
```
```
Configure

# Create the runner and start the configuration experience
$ ./config.sh --url https://github.com/bumesh7/github-actions-practice --token B5IXTEXE5OXB5RL6SMKFG4DJ3M73W
# Last step, run it!
$ ./run.sh
```
```
Use self-hosted runner

# Use this YAML in your workflow file for each job
runs-on: self-hosted
```
**Verify:** Your runner appears in the Runners list with a green dot.
<img width="1850" height="682" alt="image" src="https://github.com/user-attachments/assets/9016fa58-554b-4a07-a0bd-659ef9fcd0f8" />

---

### Task 4: Use Your Self-Hosted Runner
1. Create `.github/workflows/self-hosted.yml`
2. Set `runs-on: self-hosted`
3. Add steps that:
   - Print the hostname of the machine (it should be YOUR machine/VM)
   - Print the working directory
   - Create a file and verify it exists on your machine after the run
4. Trigger it and watch it run on your own hardware
```
$ vim self-hosted.yml

name: Self-Hosted Runner

on:
 push:
  branches:
   - main

jobs:
 self-host:
  runs-on: self-hosted

  steps:
   - name: Print host name
     run: |
      echo "The self-hosted runner name is: $(hostname)"

   - name: Print Present Working Directory
     run: pwd

   - name: Create new directory
     run: |
      mkdir -p /home/umesh/Documents/github_workflows/test
      cd /home/umesh/Documents/github_workflows/test
      pwd

```

**Verify:** Check your machine — is the file there?
<img width="1847" height="737" alt="image" src="https://github.com/user-attachments/assets/cc10effc-7146-480a-97f0-5841c0403dac" />
<img width="1538" height="253" alt="image" src="https://github.com/user-attachments/assets/e07888b8-6893-4235-9d3e-0b65640c64ca" />

---

### Task 5: Labels
1. Add a **label** to your self-hosted runner (e.g., `my-linux-runner`)
2. Update your workflow to use `runs-on: [self-hosted, my-linux-runner]`
3. Trigger it — does it still pick up the job?

```
GitHub Repo → Settings → Actions → Runners

Click your runner (e.g., umesh-Aspire-A515-57G)
Click Edit
Add a label:

or

GitHub does NOT allow changing the runner name after registration.

The name (like umesh-Aspire-A515-57G) is set during:

./config.sh

or

ust add a label:

Go to:
Settings → Actions → Runners
Click your runner
Click Edit
Add label:
```
```
modify this in ablove script

runs-on: [self-hosted, linux]
```
Write in your notes: Why are labels useful when you have multiple self-hosted runners?
```
1. Target Specific Machines
2. Different Environments
3. Resource Control
4> Scale with multiple runners
```
---

### Task 6: GitHub-Hosted vs Self-Hosted
Fill this in your notes:

| | GitHub-Hosted | Self-Hosted |
|---|---|---|
| Who manages it? | GitHub manages everything (setup, updates, maintenance) | You manage the machine (setup, updates, security) |
| Cost | Free (limited minutes) or paid for extra usage | You pay for your own infrastructure (VM, electricity, etc.) |
| Pre-installed tools | Comes with many tools (Docker, Node, Python, Git, etc.) | You must install and maintain tools yourself |
| Good for | Quick setup, CI/CD, general workloads, beginners | Custom environments, heavy workloads, special tools, private infra |
| Security concern | Safer (isolated, ephemeral runners reset after each job) | More responsibility (you must secure the machine and data) |

## Quick Summary
- GitHub-hosted = convenience + speed  
- Self-hosted = control + flexibility

---
