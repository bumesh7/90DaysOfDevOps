Task 1: Install and Configure Git

Verify Git is installed on your machine
Set up your Git identity — name and email
Verify your configuration

<img width="937" height="153" alt="image" src="https://github.com/user-attachments/assets/8a34c2b5-3715-4c88-8da4-8ea691f5aaff" />

Task 2: Create Your Git Project

Create a new folder called devops-git-practice
Initialize it as a Git repository
Check the status — read and understand what Git is telling you
Explore the hidden .git/ directory — look at what's inside

<img width="861" height="536" alt="image" src="https://github.com/user-attachments/assets/24fcdb5b-f003-4676-8812-e3eda8cc3213" />

Task 3: Create Your Git Commands Reference

Create a file called git-commands.md inside the repo
Add the Git commands you've used so far, organized by category:
Setup & Config
Basic Workflow
Viewing Changes
For each command, write:
What it does (1 line)
An example of how to use it

<img width="866" height="132" alt="image" src="https://github.com/user-attachments/assets/039d3c28-666e-4175-90ff-b58991883e2e" />

<img width="1025" height="175" alt="image" src="https://github.com/user-attachments/assets/702da3a0-8ba6-45da-ab0d-15d80ea99cf0" />


Task 4: Stage and Commit

Stage your file
Check what's staged
Commit with a meaningful message
View your commit history

<img width="1231" height="597" alt="image" src="https://github.com/user-attachments/assets/7c7638be-e70e-4966-a41c-30557bf96836" />

Task 5: Make More Changes and Build History

Edit git-commands.md — add more commands as you discover them
Check what changed since your last commit
Stage and commit again with a different, descriptive message
Repeat this process at least 3 times so you have multiple commits in your history
View the full history in a compact format

<img width="1038" height="242" alt="image" src="https://github.com/user-attachments/assets/30efa70f-0e9f-4c3f-8329-135876daa603" />

<img width="1171" height="748" alt="image" src="https://github.com/user-attachments/assets/74bb8239-89c4-4697-88e7-393868dd4c65" />


Task 6: Understand the Git Workflow

Answer these questions in your own words (add them to a day-22-notes.md file):

What is the difference between git add and git commit?
-> git add allows you to store the files from untracked to staging area allows what to keep and discard.
-> git commit is the final thing after changes done to files that can stored in the git log info.

What does the staging area do? Why doesn't Git just commit directly?
-> Staging area allows you to select which files need to be changed and which dont.
-> If we commit once we cant change the files those commits will be stored in log/changes will be saved permanently.

What information does git log show you?
-> It shows the list of commits history.

What is the .git/ folder and what happens if you delete it?
-> .Git/ will have all the files related to repo and changes information, if we delete it Version control info will be lost and it turn it normal folder.

What is the difference between a working directory, staging area, and repository?
-> Working Dir is the files where you edit
-> Staging area is where you prepare changes for commit
-> Repository the committed changes done to file will be stored in the repo.
