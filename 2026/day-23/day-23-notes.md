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
Push your main branch to GitHub
Push feature-1 branch to GitHub
Verify both branches are visible on GitHub

Answer in your notes: What is the difference between origin and upstream?

Task 4: Pull from GitHub
Make a change to a file directly on GitHub (use the GitHub editor)
Pull that change to your local repo
Answer in your notes: What is the difference between git fetch and git pull?

Task 5: Clone vs Fork
Clone any public repository from GitHub to your local machine
Fork the same repository on GitHub, then clone your fork
Answer in your notes:
What is the difference between clone and fork?
When would you clone vs fork?
After forking, how do you keep your fork in sync with the original repo?
