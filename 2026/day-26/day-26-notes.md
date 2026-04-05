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
```
<img width="1108" height="158" alt="image" src="https://github.com/user-attachments/assets/d0c86c9f-7520-43e5-9bc9-f59b2fb659c2" />

---

### Task 3: Issues
1. Create an issue on one of your repos from the terminal — give it a title, body, and a label
2. List all open issues on that repo
3. View a specific issue by its number
4. Close an issue from the terminal
5. Answer in your notes: How could you use `gh issue` in a script or automation?

---

### Task 4: Pull Requests
1. Create a branch, make a change, push it, and create a **pull request** entirely from the terminal
2. List all open PRs on a repo
3. View the details of your PR — check its status, reviewers, and checks
4. Merge your PR from the terminal
5. Answer in your notes:
   - What merge methods does `gh pr merge` support?
   - How would you review someone else's PR using `gh`?

---

### Task 5: GitHub Actions & Workflows (Preview)
1. List the workflow runs on any public repo that uses GitHub Actions
2. View the status of a specific workflow run
3. Answer in your notes: How could `gh run` and `gh workflow` be useful in a CI/CD pipeline?

(Don't worry if you haven't learned GitHub Actions yet — this is a preview for upcoming days)

---

### Task 6: Useful `gh` Tricks
Explore and try these — add the ones you find useful to your `git-commands.md`:
1. `gh api` — make raw GitHub API calls from the terminal
2. `gh gist` — create and manage GitHub Gists
3. `gh release` — create and manage releases
4. `gh alias` — create shortcuts for commands you use often
5. `gh search repos` — search GitHub repos from the terminal

---

## Hints
- `gh help` and `gh <command> --help` are your best friends
- Most `gh` commands work with `--repo owner/repo` to target a specific repo
- Use `--json` flag with most commands to get machine-readable output (useful for scripting)
- `gh pr create --fill` auto-fills the PR title and body from your commits

---

## Submission
1. Add your `day-26-notes.md` to `2026/day-26/`
2. Update `git-commands.md` with `gh` commands — this completes your Git & GitHub reference from Days 22–26
3. Push to your fork

---

## Learn in Public

Share your favorite `gh` commands or a screenshot of creating a PR from the terminal on LinkedIn.

`#90DaysOfDevOps` `#DevOpsKaJosh` `#TrainWithShubham`

Happy Learning!
**TrainWithShubham**
