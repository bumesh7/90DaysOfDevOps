Task 1: Understanding Branches
Answer these in your day-23-notes.md:

What is a branch in Git?
-> A branch is like seprate workapsce where you can implimiment new changes and doesnot affect main branch.

Why do we use branches instead of committing everything to main?
-> If we commit everthing in main bugs can cause the production in danger. Branches helps to work safe, review before merge and coolaborate.

What is HEAD in Git?
-> This is the current commit you are working on.

What happens to your files when you switch branches?
-> Changes the HEAD pointer, updates the working directory files and make inline with switched branch.

Task 2: Branching Commands — Hands-On
In your devops-git-practice repo, perform the following:

List all branches in your repo
$ git branch

Create a new branch called feature-1
$ git checkout -b feature-1

Switch to feature-1
$ git switch branch feature-1

Create a new branch and switch to it in a single command — call it feature-2
$ git switch -c feature-2

Try using git switch to move between branches — how is it different from git checkout?
$ git checkout main / git switch main
both act same point HEAD to main but switch is safer to switch between branches.

git switch introduced in v2.23 => switch branches and cannot accidentally chekout files.
Git checkout has multiple roles like switch branch, restor files and checkout specific commits.

Make a commit on feature-1 that does not exist on main
$ vim file.txt
Hello

$ git add .
$ git commit -m "Added file in feature-1 branch"

Switch back to main — verify that the commit from feature-1 is not there
$ git chekout main

Delete a branch you no longer need
$ git branch -d feature-2   (d = if the branch is merged then we can delete it)
$ git branch -D feature-2  (D = Delete the unmerged branch)

Add all branching commands to your git-commands.md

Task 3: Push to GitHub

Create a new repository on GitHub (do NOT initialize it with a README)
Connect your local devops-git-practice repo to the GitHub remote
-> Local to Remote repo connected

Push your main branch to GitHub
$ git checkout main
$ git push -u origin main

Push feature-1 branch to GitHub
$ git checkout feature-1
$ git push -u origin featue-1

Verify both branches are visible on GitHub

<img width="437" height="390" alt="image" src="https://github.com/user-attachments/assets/dcbc7767-ebf2-44ab-9c79-0c4348a28a84" />


Answer in your notes: What is the difference between origin and upstream?

Task 4: Pull from GitHub
Make a change to a file directly on GitHub (use the GitHub editor)
Pull that change to your local repo
$ git pull origin feature-1

Answer in your notes: What is the difference between git fetch and git pull?
git fetch => download the changes and does not update the changes to current branch
git pull => download the changes and update the changes to current branch


Task 5: Clone vs Fork

Clone any public repository from GitHub to your local machine
$ git clone https://github.com/TrainWithShubham/90DaysOfDevOps.git


Fork the same repository on GitHub, then clone your fork
-> Done

Answer in your notes:

What is the difference between clone and fork?

Clone = Copies the entire repository, includes all branches and history, Stores in your computer
Fork = A fork creates a new copy of a repository under your GitHub account.

When would you clone vs fork?
Clone = when you want to just run the files and check. you are contributing to main branch.
Fork = When you need to contribute to open source project.

After forking, how do you keep your fork in sync with the original repo

Clone = download to your computer

Fork = create your own copy on GitHub

Upstream remote = keeps your fork updated
