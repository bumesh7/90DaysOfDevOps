### Task 1: Git Reset — Hands-On
1. Make 3 commits in your practice repo (commit A, B, C)
<img width="1123" height="609" alt="image" src="https://github.com/user-attachments/assets/48e2ffa8-7aec-4f6a-89dd-77eb3dde5adb" />

2. Use `git reset --soft` to go back one commit — what happens to the changes?
```
$ git reset --soft HEAD~1

What happens?
Moves HEAD from C → B
Changes from commit C are NOT lost
They stay staged (in index)

A → B (HEAD)
Changes from C → staged
```
<img width="1053" height="214" alt="image" src="https://github.com/user-attachments/assets/94283d46-fdbd-4573-a220-4072a1da04b0" />

3. Re-commit, then use `git reset --mixed` to go back one commit — what happens now?
```
$ git reset --mixed HEAD~1

What happens?
Moves HEAD from C → B
Changes from C are kept
But now unstaged (working directory)

A → B (HEAD)
Changes from C → unstaged
```
<img width="1384" height="490" alt="image" src="https://github.com/user-attachments/assets/62ac8df6-d424-45cf-96fc-95c4123523f2" />

4. Re-commit, then use `git reset --hard` to go back one commit — what happens this time?
```
$ git reset --hard HEAD~1

What happens?
Moves HEAD from C → B
Changes from C are completely deleted

A → B (HEAD)
Changes from C → gone forever

It deletes changes from:
staging area
working directory
No easy recovery (unless reflog helps)

```
<img width="1350" height="429" alt="image" src="https://github.com/user-attachments/assets/41d05d2a-8c10-4956-a49d-ac454f25aa1e" />

5. Answer in your notes:
   - What is the difference between `--soft`, `--mixed`, and `--hard`?
```
Command 	HEAD Move	    Staging Area	Working Directory
--soft	  Yes	               ✅ Kept	✅ Kept
--mixed	  Yes	             ❌ Cleared	✅ Kept
--hard	  Yes	             ❌ Cleared	❌ Deleted
```
   - Which one is destructive and why?
```
git reset --hard

Why?

It deletes changes from:
staging area
working directory
No easy recovery (unless reflog helps)
```
   - When would you use each one?
```
--soft
Fix last commit message
Combine commits (squash manually)
Keep everything staged

--mixed (most commonly used)
Undo commit but keep changes
Reorganize files before committing again

--hard
Throw away unwanted work completely
Reset repo to a clean state
```
   - Should you ever use `git reset` on commits that are already pushed?
```
Generally NO

Why?
Rewrites commit history
Causes conflicts for others
```
---

### Task 2: Git Revert — Hands-On
1. Make 3 commits (commit X, Y, Z)
<img width="1373" height="495" alt="image" src="https://github.com/user-attachments/assets/381a238c-168b-492d-b560-b87263c00e67" />

2. Revert commit Y (the middle one) — what happens?
```
git log --oneline

git revert <id>
```
3. Check `git log` — is commit Y still in the history?
```
Git does NOT delete commit Y
Instead, it creates a new commit that undoes Y

X → Y → Z → Y' (HEAD)
```
<img width="1240" height="1003" alt="image" src="https://github.com/user-attachments/assets/a04e95b4-4d35-4435-a4b6-b9eb042f57a7" />

4. Answer in your notes:
   - How is `git revert` different from `git reset`?
```
Feature	                  git revert	                      git reset
History	                  Preserved	                        Rewritten
Creates new commit	      Yes (revert commit)	              No
Safe for shared?	        Yes	                              No (dangerous)
Deletes commits?	        No	                              Yes (depending on mode)
```
   - Why is revert considered **safer** than reset for shared branches?
```
Why is git revert safer?

Because:

It does NOT change history
It only adds a new commit

Result:
No conflicts for teammates
No broken history
No need for force push
```
   - When would you use revert vs reset?
```
When to use revert vs reset?

Use git revert when:
Working on shared branches (like main)
Code is already pushed
You want a safe undo

Use git reset when:
Working locally
Commits are not pushed
You want to edit/clean history
```
```
Real-World Insight

👉 Imagine:

You pushed a bad commit (Y)
Team already pulled it

If you use reset:
History breaks
Team gets conflicts

If you use revert:
Clean fix
Everyone stays in sync
```
---

### Task 3: Reset vs Revert — Summary
Create a comparison in your notes:

| | `git reset` | `git revert` |
|---|---|---|
| What it does | Moves HEAD to a previous commit (rewrites history) | Creates a new commit that undoes a previous commit |
| Removes commit from history? | ✅ Yes (history is rewritten) | ❌ No (history is preserved) |
| Safe for shared/pushed branches? | ❌ No (can break others' history) | ✅ Yes (safe, no history rewrite) |
| When to use | Local changes, before pushing, cleaning up commits | Undo changes in shared branches or after pushing |

---

### Task 4: Branching Strategies
Research the following branching strategies and document each in your notes with:
- How it works (short description)
- A simple diagram or flow (text-based is fine)
- When/where it's used
- Pros and cons

1. **GitFlow** — develop, feature, release, hotfix branches
```
Uses multiple long-lived branches:

main → production
develop → integration branch
feature/* → new features
release/* → prepare releases
hotfix/* → urgent fixes

main ────────────────●──────────────
          ↑         ↑
       hotfix     release
          ↑         ↑
develop ──●───●───●──────────────
           ↑   ↑
        feature branches

Where it's used
Large teams
Enterprise projects
Projects with planned/scheduled releases

Pros
Clear structure
Good for versioning & releases
Stable production branch

Cons
Complex
Slower development
Too heavy for small teams
```
2. **GitHub Flow** — simple, single main branch + feature branches
```
How it works
Only one main branch: main
Create short-lived feature branches
Open PR → review → merge → deploy

main ─────●────●────●────●────
            ↑    ↑
        feature branches
            ↓    ↓
          PR → merge

Where it's used
Startups
Web apps
Continuous deployment environments

Pros
Simple & fast
Easy to understand
Works well with CI/CD

Cons
No strict release management
Can get messy in large teams
```
3. **Trunk-Based Development** — everyone commits to main, short-lived branches
```
How it works
Everyone works on main (trunk)
Very short-lived branches (or none)
Frequent small commits
Uses feature flags for incomplete work

main ──●─●─●─●─●─●─●──
        ↑ ↑ ↑ ↑
     small commits / short branches

Where it's used
High-performing DevOps teams
Companies like Google, Netflix

Pros
Very fast development
Fewer merge conflicts
Ideal for CI/CD

Cons
Requires strong testing
Risky without discipline
Not beginner-friendly
```
4. Answer:
   - Which strategy would you use for a startup shipping fast?
```
GitHub Flow

Simple
Fast
Easy deployment
```
   - Which strategy would you use for a large team with scheduled releases?
```
GitFlow

Structured
Release management
Stable production cycles
```
   - Which one does your favorite open-source project use? (check any repo on GitHub)
```
GitHub Flow

Example:
Kubernetes

Use main/master
Feature branches + Pull Requests
```
```
GitFlow → structured & slow
GitHub Flow → simple & fast
Trunk-Based → fastest but needs discipline
```
---

### Task 5: Git Commands Reference Update
Update your `git-commands.md` to cover everything from Days 22–25:
- Setup & Config
- Basic Workflow (add, commit, status, log, diff)
- Branching (branch, checkout, switch)
- Remote (push, pull, fetch, clone, fork)
- Merging & Rebasing
- Stash & Cherry Pick
- Reset & Revert

```
Setup and Config

git config --global user.name "Your Name"
git config --global user.email "your@email.com
"
git config --list

git init
git clone <repo-url>

Basic Workflow

git status
git add <file>
git add .
git commit -m "message"

git log
git log --oneline

git diff
git diff --staged

Branching

git branch
git branch <branch-name>

git checkout <branch>
git switch <branch>

git switch -c <branch-name>

git branch -d <branch-name>

Remote Operations

git remote -v

git clone <repo-url>

git push origin <branch>
git pull origin <branch>
git fetch origin

Merging and Rebasing

git merge <branch>

git rebase <branch>

git rebase -i HEAD~3

Stash and Cherry Pick

git stash
git stash list
git stash apply

git cherry-pick <commit-hash>

Reset and Revert

Reset
git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1

Revert
git revert <commit-hash>

git revert --continue
git revert --abort

Pro Tips

Use --soft to keep changes staged
Use --mixed to keep changes unstaged
Use --hard carefully because it deletes changes
Use revert for shared branches
Prefer switch over checkout for branch operations

One-Line Summary

add stages changes
commit saves snapshot
branch creates new line of development
merge and rebase combine work
reset rewrites history
revert safely undoes changes
```
---
