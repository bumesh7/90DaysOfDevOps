### Task 1: Install and Authenticate
1. Install the GitHub CLI on your machine
```
sudo apt update
sudo apt install gh -y

gh --version
```
<img width="802" height="79" alt="image" src="https://github.com/user-attachments/assets/fc7a765b-f797-4a38-825d-e43c0cf24780" />

2. Authenticate with your GitHub account
```
gh auth login

#  create authentication token key in github and use that for login 
```
<img width="860" height="270" alt="image" src="https://github.com/user-attachments/assets/a9982cdd-647b-43ab-bef6-e0ea27d065bb" />

3. Verify you're logged in and check which account is active
```
gh auth status
```
<img width="1508" height="183" alt="image" src="https://github.com/user-attachments/assets/b93cd5ed-d1ac-4755-9649-348f6c6b7163" />

4. Answer in your notes: What authentication methods does `gh` support?
```
Browser-based OAuth (Recommended)
Personal Access Token (PAT)
SSH AUthentication
Environment Variable Token
```
---

### Task 2: Working with Repositories
1. Create a **new GitHub repo** directly from the terminal — make it public with a README
```
gh repo create my-test-repo \
  --public \
  --description "Test repo created from CLI" \
  --add-readme
```
2. Clone a repo using `gh` instead of `git clone`
```
syntax: gh repo clone <username>/<repo-name>

gh repo clone bumesh7/my-test-repo
```
<img width="1013" height="218" alt="image" src="https://github.com/user-attachments/assets/61ef6eea-6bc5-42ee-8a8d-f8cb37c3d0d1" />

3. View details of one of your repos from the terminal
```
syntax: gh repo view <username>/<repo-name>

gh repo view bumesh7/my-test-repo
```
<img width="1032" height="274" alt="image" src="https://github.com/user-attachments/assets/e2c05d7d-a386-4c0f-bbf0-53b83c90bfb0" />

4. List all your repositories
```
gh repo list

gh repo list --limit 10
```
<img width="1858" height="477" alt="image" src="https://github.com/user-attachments/assets/de07c84e-ced3-4b7f-8de4-516dfe718a58" />

5. Open a repo in your browser directly from the terminal
```
syntax: gh repo view <user-name>/<repo-name> --web

gh repo view bumesh7/my-test-repo --web
```
6. Delete the test repo you created (be careful!)
```
gh repo delete <user-name>/<repo-name>

gh repo delete bumesh7/my-test-repo
```
<img width="1108" height="158" alt="image" src="https://github.com/user-attachments/assets/d0c86c9f-7520-43e5-9bc9-f59b2fb659c2" />

---

### Task 3: Issues
1. Create an issue on one of your repos from the terminal — give it a title, body, and a label
```
syntax:
gh issue create \
  --repo username/repo-name \
  --title "Bug: Login not working" \
  --body "Users are unable to login after latest update" \
  --label "bug"

gh issue create \
  --repo bumesh7/my-test-repo \
  --title "Fix Terraform script error" \
  --body "Terraform apply fails due to missing variable" \
  --label "bug"
```
<img width="1813" height="447" alt="image" src="https://github.com/user-attachments/assets/9a53eb47-a527-4797-9a30-227741e950a5" />

2. List all open issues on that repo
```
syntax: gh issue list --repo username/repo-nam

gh issue list --repo bumesh7/my-test-repo
```
<img width="1170" height="364" alt="image" src="https://github.com/user-attachments/assets/e25489b9-160a-48b6-868b-e402ef27d95a" />

3. View a specific issue by its number
```
syntax: gh issue view ISSUE_NUMBER --repo username/repo-name

gh issue view 1 --repo bumesh7/my-test-repo

if that command do not work 

gh issue view 1 --repo bumesh7/my-test-repo --json title,body,state
```
<img width="1443" height="184" alt="image" src="https://github.com/user-attachments/assets/7f07b4d8-51be-4b62-9748-9f57d3920bee" />

4. Close an issue from the terminal
```
syntax: gh issue view 1 --repo <username>/<repo-name>

gh issue close 1 --repo bumesh7/my-test-repo
```
<img width="1453" height="77" alt="image" src="https://github.com/user-attachments/assets/51665b3d-69d5-4915-acc8-e301b933a674" />

5. Answer in your notes: How could you use `gh issue` in a script or automation?
```
gh issue create --title "Build Failed" --body "Check CI logs"
```
---

### Task 4: Pull Requests
1. Create a branch, make a change, push it, and create a **pull request** entirely from the terminal
```
git checkout -b feature-update-readme

echo "Update from CLI" >> README.md
git add README.md
git commit -m "Updated README from CLI"

git push origin feature-update-readme

gh pr create \
  --repo bumesh7/my-test-repo \
  --title "Update README via CLI" \
  --body "This PR updates README file from terminal" \
  --base main \
  --head feature-update-readme
```
<img width="1548" height="660" alt="image" src="https://github.com/user-attachments/assets/3066a605-cbcf-41f8-9cc9-619183ab8fed" />

2. List all open PRs on a repo
```
gh pr list --repo bumesh7/my-test-repo
```
<img width="1203" height="171" alt="image" src="https://github.com/user-attachments/assets/fb0c076d-6196-491e-913b-d4ed91cb43f1" />

3. View the details of your PR — check its status, reviewers, and checks
```
syntax: gh pr view PR_NUMBER --repo bumesh7/my-test-repo

gh pr view 1 --repo bumesh7/my-test-repo --json title,body,state
```
<img width="1442" height="173" alt="image" src="https://github.com/user-attachments/assets/f09f071b-1073-4e3b-a9c4-d1dc319a4519" />

4. Merge your PR from the terminal
```
syntax: gh pr merge PR_NUMBER --repo bumesh7/my-test-repo

gh pr merge 1 --repo bumesh7/my-test-repo
```
<img width="1230" height="188" alt="image" src="https://github.com/user-attachments/assets/eed39574-2a47-482a-9205-cf776437f926" />
```
Create a merge commit (default)

Keeps all commits + adds a merge commit
History looks like a tree
Good for tracking full development history
Slightly messy in big projects

Rebase and merge

Moves your commits on top of main
No merge commit
Clean linear history
Rewrites commit history

Squash and merge (Recommended)

Combines all commits into one single commit
Clean history
Easy to read
Most teams prefer this
```
5. Answer in your notes:
   - What merge methods does `gh pr merge` support?
```
gh pr merge --merge

gh pr merge --squash

gh pr merge --rebase
```
   - How would you review someone else's PR using `gh`?
```
checkout = gh pr checkout 1
view changes = gh pr diff 1
comment on PR = gh pr comment 1 --body "Looks good!"
Approve PR = gh pr review 1 --approve
Request Changes = gh pr review 1 --request-changes --body "Fix this issue"
```
---

### Task 5: GitHub Actions & Workflows (Preview)
1. List the workflow runs on any public repo that uses GitHub Actions
```
syntax: gh run list --repo <user-name>/<repo-name>

gh run list --repo bumesh7/GitHub_Actions
```
2. View the status of a specific workflow run
```
syntax: gh run view RUN_ID --repo <user-name>/<repo-name>

gh run view 23089738835 --repo bumesh7/GitHub_Actions

real time ci/cd monitor = gh run watch RUN_ID --repo <user-name>/<repo-name>
```
<img width="1511" height="496" alt="image" src="https://github.com/user-attachments/assets/45e9cd35-5934-4a1b-9e30-fc03553a9956" />

3. Answer in your notes: How could `gh run` and `gh workflow` be useful in a CI/CD pipeline?
```
monitor pieplines = gh run list
Debug Failuer = gh run view RUN_ID --log
Trigger workflow manually = gh workflow run workflow.yml
Re-run failed jobs = gh run rerun RUN_ID
Manage Workflow = gh workflow list

Automate Devops Workflow

if gh run list | grep failure; then
  echo "Pipeline failed!"
fi
```
(Don't worry if you haven't learned GitHub Actions yet — this is a preview for upcoming days)

---

### Task 6: Useful `gh` Tricks
Explore and try these — add the ones you find useful to your `git-commands.md`:
1. `gh api` — make raw GitHub API calls from the terminal
```
gh api user
gh api repos/bumesh7/my-test-repo
gh api repos/bumesh7/my-test-repo/issues
```
2. `gh gist` — create and manage GitHub Gists
```
gh gist create file.txt --public
echo "Hello DevOps" | gh gist create --public
gh gist list
```
3. `gh release` — create and manage releases
```
gh release create v1.0.0 \
  --title "First Release" \
  --notes "Initial release"

gh release list

gh release view v1.0.0
```
4. `gh alias` — create shortcuts for commands you use often
```
gh alias set prs "pr list"

gh prs

gh alias set co "pr checkout"
```
5. `gh search repos` — search GitHub repos from the terminal
```
gh search repos terraform

gh search repos "terraform aws" --language=HCL

gh search repos devops --limit 5
```
---
