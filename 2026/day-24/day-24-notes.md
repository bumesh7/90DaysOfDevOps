Task 1: Git Merge — Hands-On

Create a new branch feature-login from main, add a couple of commits to it
<img width="890" height="412" alt="image" src="https://github.com/user-attachments/assets/8928c7d5-66f5-4a28-b659-f0c47234f176" />

Switch back to main and merge feature-login into main
<img width="820" height="202" alt="image" src="https://github.com/user-attachments/assets/86b43b26-3edd-4b08-a0ed-66b8f3aef1c9" />

Observe the merge — did Git do a fast-forward merge or a merge commit?
-> Fast-Forward

Now create another branch feature-signup, add commits to it — but also add a commit to main before merging
<img width="937" height="506" alt="image" src="https://github.com/user-attachments/assets/59ae7a7d-992a-4fe7-b432-9223295a4a96" />

<img width="927" height="343" alt="image" src="https://github.com/user-attachments/assets/96697d9f-995e-4fe2-b96a-05c5a4c058cf" />

Merge feature-signup into main — what happens this time?
<img width="692" height="133" alt="image" src="https://github.com/user-attachments/assets/e1898f91-c8ed-4b7a-91a7-52dfbcb78be9" />

ORT = Ostensibly Recursive’s Twin

Answer in your notes:
What is a fast-forward merge?
-> A fast-forward merge happens when Git just moves the branch pointer forward because there are no separate changes to combine.

When does Git create a merge commit instead?
-> Git creates a merge commit when both branches have new changes and their histories need to be combined.

What is a merge conflict? (try creating one intentionally by editing the same line in both branches)
-> A merge conflict happens when Git cannot decide which changes to keep because the same line was edited differently in both branches.

Task 2: Git Rebase — Hands-On

Create a branch feature-dashboard from main, add 2-3 commits
<img width="1093" height="411" alt="image" src="https://github.com/user-attachments/assets/b1c12bb0-80fc-498f-9da3-4043bcf3e379" />

While on main, add a new commit (so main moves ahead)
<img width="932" height="232" alt="image" src="https://github.com/user-attachments/assets/11d64d48-aaa7-4ea6-81e4-c4505af4880e" />

Switch to feature-dashboard and rebase it onto main
<img width="955" height="158" alt="image" src="https://github.com/user-attachments/assets/7a114d81-ac10-42aa-9c8b-71024d193851" />

Observe your git log --oneline --graph --all — how does the history look compared to a merge?
<img width="902" height="267" alt="image" src="https://github.com/user-attachments/assets/ba4660ee-dd81-4490-804a-bf4ca9b6e991" />


Answer in your notes:
What does rebase actually do to your commits?
-> Rebase moves your commits to the top of another branch and rewrites them as new commits.

How is the history different from a merge?
-> Rebase creates a clean straight history, while merge keeps a branch history with a merge commit.

Why should you never rebase commits that have been pushed and shared with others?
-> Because rebasing changes commit history and can break other people’s work.

When would you use rebase vs merge?
-> Use rebase for a clean history on your own branch, and merge when working with shared branches.

Task 3: Squash Commit vs Merge Commit

Create a branch feature-profile, add 4-5 small commits (typo fix, formatting, etc.)
<img width="1137" height="692" alt="image" src="https://github.com/user-attachments/assets/942b24c8-eaa7-4518-b535-bf3f5b7cb2b4" />

Merge it into main using --squash — what happens?
<img width="1080" height="683" alt="image" src="https://github.com/user-attachments/assets/caafe1ff-ac54-46ac-b4fe-365c8ef14e57" />

Check git log — how many commits were added to main?
<img width="772" height="223" alt="image" src="https://github.com/user-attachments/assets/99baf15f-78e5-42fb-b080-f71c00bcfa3e" />

Now create another branch feature-settings, add a few commits
<img width="1162" height="717" alt="image" src="https://github.com/user-attachments/assets/4db2ec1c-36db-469b-a46f-b31ef957723c" />

Merge it into main without --squash (regular merge) — compare the history
<img width="902" height="287" alt="image" src="https://github.com/user-attachments/assets/c07a9c01-9396-43b9-9ac6-7830eec8ff20" />

Answer in your notes:
What does squash merging do?
-> Squash merging combines all branch commits into one single commit.

When would you use squash merge vs regular merge?
-> Use squash to keep history clean, and regular merge to keep all commit details.

What is the trade-off of squashing?
-> You lose the detailed commit history of that branch.

Task 4: Git Stash — Hands-On

Start making changes to a file but do not commit
Now imagine you need to urgently switch to another branch — try switching. What happens?
<img width="1095" height="456" alt="image" src="https://github.com/user-attachments/assets/a786131d-127c-497e-9e21-d17e2a4f7c54" />

Use git stash to save your work-in-progress
<img width="1328" height="622" alt="image" src="https://github.com/user-attachments/assets/3f9e2632-3f9e-4b91-9260-3c2340bb2a56" />

Switch to another branch, do some work, switch back
<img width="1178" height="308" alt="image" src="https://github.com/user-attachments/assets/8fa519d5-238a-41bd-8ffe-4528501060cf" />

Apply your stashed changes using git stash pop
<img width="1115" height="673" alt="image" src="https://github.com/user-attachments/assets/869d6aca-6acc-444f-933a-6fd0c902f792" />

Try stashing multiple times and list all stashes
Try applying a specific stash from the list
<img width="1115" height="673" alt="image" src="https://github.com/user-attachments/assets/a09ff72c-3a9f-407d-8eab-caacfae69606" />
<img width="608" height="231" alt="image" src="https://github.com/user-attachments/assets/7ce045b8-f34c-42b5-85b8-46a1125fa7ba" />

Answer in your notes:
What is the difference between git stash pop and git stash apply?
-> git stash pop applies the stash and deletes it, while git stash apply applies it but keeps it saved.

When would you use stash in a real-world workflow?
-> Use stash to temporarily save unfinished work so you can switch branches without committing.

Task 5: Cherry Picking

Create a branch feature-hotfix, make 3 commits with different changes
<img width="1370" height="715" alt="image" src="https://github.com/user-attachments/assets/e56fa0c5-a9fa-480f-afb2-a8f1ceb3d7c7" />

Switch to main
Cherry-pick only the second commit from feature-hotfix onto main
Verify with git log that only that one commit was applied
<img width="1483" height="396" alt="image" src="https://github.com/user-attachments/assets/e16ff163-2ca9-478e-80ad-9fa9e5194638" />

Answer in your notes:
What does cherry-pick do?
-> Cherry-pick applies a specific commit from another branch onto your current branch.

When would you use cherry-pick in a real project?
-> Use when to single bug fix or feature commit into another branch without merging everything.

What can go wrong with cherry-picking?
-> It can cause conflicts or duplicate commits if not handled carefully.
