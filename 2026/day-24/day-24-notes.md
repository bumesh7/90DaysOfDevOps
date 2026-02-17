Task 1: Git Merge — Hands-On
  
Create a new branch feature-login from main, add a couple of commits to it
<img width="1117" height="485" alt="image" src="https://github.com/user-attachments/assets/ee20029c-4602-4611-a6ae-3f8bb4f4b8f9" />

Switch back to main and merge feature-login into main
<img width="727" height="197" alt="image" src="https://github.com/user-attachments/assets/118e18f1-8909-4ba2-a5e6-52620e0105f8" />

Observe the merge — did Git do a fast-forward merge or a merge commit?
-> Fast-Forward

Now create another branch feature-signup, add commits to it — but also add a commit to main before merging
<img width="911" height="285" alt="image" src="https://github.com/user-attachments/assets/c2da134e-2eff-4a19-86a5-0028bf394dbc" />

<img width="1092" height="332" alt="image" src="https://github.com/user-attachments/assets/94fc889b-7c6c-4815-95f3-4465b9ac0efa" />


Merge feature-signup into main — what happens this time?
<img width="982" height="552" alt="image" src="https://github.com/user-attachments/assets/e6c346f0-eb39-4e11-b0fd-fc1e56c2d2f7" />

Answer in your notes:
What is a fast-forward merge?
-> A fast-forward merge moves the branch pointer forward to the latest commit when there is no divergent history between branches.
When does Git create a merge commit instead?
-> Git creates a merge commit when the branches have diverged and both contain unique commits.
What is a merge conflict? (try creating one intentionally by editing the same line in both branches)
-> A merge conflict occurs when Git cannot automatically merge changes because the same line in a file was modified differently in both branches.

Task 2: Git Rebase — Hands-On

Create a branch feature-dashboard from main, add 2-3 commits
While on main, add a new commit (so main moves ahead)
<img width="963" height="696" alt="image" src="https://github.com/user-attachments/assets/4f68fbdc-d12e-4b89-a30a-12ef77b4f628" />
<img width="932" height="683" alt="image" src="https://github.com/user-attachments/assets/c22688f0-bfd4-4a42-928c-e9150b0e024d" />



Switch to feature-dashboard and rebase it onto main
<img width="832" height="91" alt="image" src="https://github.com/user-attachments/assets/b3371907-16e8-4d2d-aa36-41e67cbe9998" />


Observe your git log --oneline --graph --all — how does the history look compared to a merge?
<img width="856" height="273" alt="image" src="https://github.com/user-attachments/assets/7357cbf0-657f-4146-b2ad-e658b5bef7c8" />

Answer in your notes:
What does rebase actually do to your commits?
-> Rebase rewrites your commits by replaying them on top of another base commit, creating new commit hashes.
How is the history different from a merge?
-> Rebase creates a linear history while merge preserves branch history with a merge commit.
Why should you never rebase commits that have been pushed and shared with others?
-> Because rebase rewrites commit history and can cause conflicts and confusion for others who pulled the original commits.
When would you use rebase vs merge?
-> Use rebase to keep history clean and linear locally, and use merge when integrating shared branches to preserve history.

Task 3: Squash Commit vs Merge Commit
Create a branch feature-profile, add 4-5 small commits (typo fix, formatting, etc.)
Merge it into main using --squash — what happens?
Check git log — how many commits were added to main?
Now create another branch feature-settings, add a few commits
Merge it into main without --squash (regular merge) — compare the history
Answer in your notes:
What does squash merging do?
When would you use squash merge vs regular merge?
What is the trade-off of squashing?

Task 4: Git Stash — Hands-On
Start making changes to a file but do not commit
Now imagine you need to urgently switch to another branch — try switching. What happens?
Use git stash to save your work-in-progress
Switch to another branch, do some work, switch back
Apply your stashed changes using git stash pop
Try stashing multiple times and list all stashes
Try applying a specific stash from the list
Answer in your notes:
What is the difference between git stash pop and git stash apply?
When would you use stash in a real-world workflow?

Task 5: Cherry Picking
Create a branch feature-hotfix, make 3 commits with different changes
Switch to main
Cherry-pick only the second commit from feature-hotfix onto main
Verify with git log that only that one commit was applied
Answer in your notes:
What does cherry-pick do?
When would you use cherry-pick in a real project?
What can go wrong with cherry-picking?
