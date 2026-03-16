Task 1: Understanding Ownership (10 minutes)

Run ls -l in your home directory
Identify the owner and group columns
Check who owns your files
Format: -rw-r--r-- 1 owner group size date filename

Ubuntu Ubuntu owns the files.

<img width="836" height="357" alt="image" src="https://github.com/user-attachments/assets/062db61f-c11d-4182-832c-727217ff2288" />

Document: What's the difference between owner and group?
-> Owner => Who creates the file or directory
-> group => List of users are in that group and share the common permissions.

Task 2: Basic chown Operations (20 minutes)

Create file devops-file.txt
Check current owner: ls -l devops-file.txt
Change owner to tokyo (create user if needed)
Change owner to berlin
Verify the changes
Try:

<img width="885" height="207" alt="image" src="https://github.com/user-attachments/assets/9c5390c5-a3ef-49eb-a60c-6d29579e04bb" />
sudo chown tokyo devops-file.txt

Task 3: Basic chgrp Operations (15 minutes)

Create file team-notes.txt
Check current group: ls -l team-notes.txt
Create group: sudo groupadd heist-team
Change file group to heist-team
Verify the change

<img width="922" height="236" alt="image" src="https://github.com/user-attachments/assets/b5a61e21-73c0-4b32-b43c-3c61467ea66b" />

Task 4: Combined Owner & Group Change (15 minutes)

Using chown you can change both owner and group together:

Create file project-config.yaml
Change owner to professor AND group to heist-team (one command)
Create directory app-logs/
Change its owner to berlin and group to heist-team
Syntax: sudo chown owner:group filename

<img width="1027" height="170" alt="image" src="https://github.com/user-attachments/assets/00d5b608-f733-4127-abb8-0977c6b98912" />


Task 5: Recursive Ownership (20 minutes)
Create directory structure:

mkdir -p heist-project/vault
mkdir -p heist-project/plans
touch heist-project/vault/gold.txt
touch heist-project/plans/strategy.conf
Create group planners: sudo groupadd planners

Change ownership of entire heist-project/ directory:

Owner: professor
Group: planners
Use recursive flag (-R)
Verify all files and subdirectories changed: ls -lR heist-project/

<img width="1038" height="142" alt="image" src="https://github.com/user-attachments/assets/9035e5c0-752e-467c-8c6b-c0f09efcf622" />

<img width="982" height="310" alt="image" src="https://github.com/user-attachments/assets/fb2d06df-3463-49c7-a52d-665e2d0e4b5f" />



Task 6: Practice Challenge (20 minutes)
Create users: tokyo, berlin, nairobi (if not already created)

Create groups: vault-team, tech-team

Create directory: bank-heist/

Create 3 files inside:

touch bank-heist/access-codes.txt
touch bank-heist/blueprints.pdf
touch bank-heist/escape-plan.txt
Set different ownership:

access-codes.txt → owner: tokyo, group: vault-team
blueprints.pdf → owner: berlin, group: tech-team
escape-plan.txt → owner: nairobi, group: vault-team
Verify: ls -l bank-heist/

<img width="1155" height="368" alt="image" src="https://github.com/user-attachments/assets/c29457fa-399b-463e-9129-9bec13334322" />






![20260204_225802](https://github.com/user-attachments/assets/b5c9b598-fdd9-4056-b249-efa12fa93669)

![20260204_225813](https://github.com/user-attachments/assets/a5df18f1-e646-4873-90b8-71db403fd285)

![20260204_225856](https://github.com/user-attachments/assets/27d6441a-77ec-4deb-a612-95564f924029)
