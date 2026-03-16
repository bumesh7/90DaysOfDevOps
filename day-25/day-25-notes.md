Task 1: Git Reset — Hands-On

Make 3 commits in your practice repo (commit A, B, C)
<img width="1092" height="490" alt="image" src="https://github.com/user-attachments/assets/7773f04e-cedf-4538-8503-a24df6ed301a" />

Use git reset --soft to go back one commit — what happens to the changes?
-> Removes commit, keeps changes staged
<img width="992" height="753" alt="image" src="https://github.com/user-attachments/assets/76f4eb83-8e1a-4659-8405-57d533953788" />

Re-commit, then use git reset --mixed to go back one commit — what happens now?
-> Removes commit, keeps changes unstaged
<img width="1015" height="762" alt="image" src="https://github.com/user-attachments/assets/2afc2bd6-a945-4063-9e90-a3592f1355e1" />

Re-commit, then use git reset --hard to go back one commit — what happens this time?
-> Removes commit AND deletes changes
<img width="876" height="687" alt="image" src="https://github.com/user-attachments/assets/cf30b58d-7555-4198-af42-c0763398731e" />

cmd      
--soft -> Commit removed (Yes), Changes Kept (Yes), Staged (Yes)
--mixed -> Commit removed (Yes), Changes Kept (Yes), Staged (No)
--hard -> Commit removed (Yes), Changes Kept (No), Staged (No)

Answer in your notes:
What is the difference between --soft, --mixed, and --hard?
--soft => Removes commit, keeps changes staged
--mixed => Removes commit, keeps changes unstaged
--hard => Removes commit AND deletes changes

Which one is destructive and why?
--hard is destructive because it deletes the entire commit and changes done before and make working tree clean.

When would you use each one?
--soft => to change commit message, to add forgetten file and recommit
--mixed => undo a commit, edit files before committing, accindentaly commited early
--hard => make working tree clean, completely discard the changes.

Should you ever use git reset on commits that are already pushed?
No dont reset once you have pusshed, other have already pulled the code.

Task 2: Git Revert — Hands-On

Make 3 commits (commit X, Y, Z)
<img width="942" height="736" alt="image" src="https://github.com/user-attachments/assets/6e571c53-f981-4f31-bbd6-b1c59828452c" />

Revert commit Y (the middle one) — what happens?

Check git log — is commit Y still in the history?
Answer in your notes:
How is git revert different from git reset?
Why is revert considered safer than reset for shared branches?
When would you use revert vs reset?

Task 3: Reset vs Revert — Summary

Create a comparison in your notes:

git reset	git revert
What it does	?	?
Removes commit from history?	?	?
Safe for shared/pushed branches?	?	?
When to use	?	?

Task 4: Branching Strategies
Research the following branching strategies and document each in your notes with:

How it works (short description)
A simple diagram or flow (text-based is fine)
When/where it's used
Pros and cons
GitFlow — develop, feature, release, hotfix branches
GitHub Flow — simple, single main branch + feature branches
Trunk-Based Development — everyone commits to main, short-lived branches
Answer:
Which strategy would you use for a startup shipping fast?
Which strategy would you use for a large team with scheduled releases?
Which one does your favorite open-source project use? (check any repo on GitHub)

Task 5: Git Commands Reference Update
Update your git-commands.md to cover everything from Days 22–25:

Setup & Config
Basic Workflow (add, commit, status, log, diff)
Branching (branch, checkout, switch)
Remote (push, pull, fetch, clone, fork)
Merging & Rebasing
Stash & Cherry Pick
Reset & Revert
