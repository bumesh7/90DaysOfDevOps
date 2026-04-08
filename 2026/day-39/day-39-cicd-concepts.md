Challenge Tasks
Task 1: The Problem

Think about a team of 5 developers all pushing code to the same repo manually deploying to production.

Write in your notes:
```
What can go wrong?
-> Merger issues
-> Code conflicts
-> Untracked commits

What does "it works on my machine" mean and why is it a real problem?
-> Different OS version
-> Different dependency version
-> Different environment variables

How many times a day can a team safely deploy manually?
-> 2 to 3 times, a person can manually deploy.
```

Task 2: CI vs CD

Research and write short definitions (2-3 lines each):
```
Continuous Integration — what happens, how often, what it catches

CI - It is the process of fetching the code and test the code whenever the new is code is pushed to the repository.
It can happen multiple times a day.
Code can have bugs, Code may not reach quality standard

Continuous Delivery — how it's different from CI, what "delivery" means

Once the code is passed in the testing phase then code is deployed to the server in manual approval step.
CI ensure code works and code can be released at any time
The code is tested in staging area once code is passed, the manager click deploy for production.

Continuous Deployment — how it differs from Delivery, when teams use it

When the code is build and the code is tested automatic test cases and deployed directly without manual approval.
Delivery = manual release trigger.
Deployment = fully automated release to production.
Use when you have strong test cases that can cover all cases.

Netflix uses this proceess that cover wide range of testcases and deploy automatically.
```

Write one real-world example for each.

Task 3: Pipeline Anatomy

A pipeline has these parts — write what each one does:

    Trigger — what starts the pipeline
    Stage — a logical phase (build, test, deploy)
    Job — a unit of work inside a stage
    Step — a single command or action inside a job
    Runner — the machine that executes the job
    Artifact — output produced by a job

Task 4: Draw a Pipeline

Draw a CI/CD pipeline for this scenario:

    A developer pushes code to GitHub. The app is tested, built into a Docker image, and deployed to a staging server.

Include at least 3 stages. Hand-drawn and photographed is perfectly fine.

```
[ Developer ]
      |
      v
[ GitHub Repo ]
 (Code Push)
      |
      v
-------------------------
|   CI Pipeline         |
|-----------------------|
| 1. Build Stage        |
| - Install deps        |
| - Compile app         |
|                       |
| 2. Test Stage         |
| - Run unit tests      |
| - Run lint checks     |
|                       |
| 3. Docker Build       |
| - Build Docker image  |
| - Push to registry    |
-------------------------
      |
      v
-------------------------
|   CD Pipeline         |
|-----------------------|
| 4. Deploy Stage       |
| - Pull Docker image   |
| - Deploy to Staging   |
-------------------------
      |
      v
[ Staging Server ]
```

Task 5: Explore in the Wild

    Open any popular open-source repo on GitHub (Kubernetes, React, FastAPI — pick one you know)
    Find their .github/workflows/ folder
    Open one workflow YAML file
    Write in your notes:
What triggers it?
```
Runs on:
push (when code is pushed)
pull_request (when a PR is created or updated)

This ensures every change is tested before merging.
```
How many jobs does it have?
```
1 main job (e.g., test)
Inside the job, multiple steps run sequentially
```
What does it do? (best guess)
```
Sets up Python environment
Installs dependencies
Runs tests (pytest)
checks formatting/linting

Basically, it ensures the code is working and clean before deployment
```

Hints

    CI/CD is a practice, not just a tool
    GitHub Actions, Jenkins, GitLab CI, CircleCI — all are tools that implement CI/CD
    A pipeline failing is not a problem — it's CI/CD doing its job

