Task 1: Create Users (20 minutes)
Create three users with home directories and passwords:

tokyo
berlin
professor
<img width="792" height="382" alt="image" src="https://github.com/user-attachments/assets/6649a383-7a70-4926-9043-22fb5ac4f2cb" />

Verify: Check /etc/passwd and /home/ directory

<img width="730" height="116" alt="image" src="https://github.com/user-attachments/assets/1a606a33-d4ae-49bd-90f9-1d09d7658be5" />

Task 2: Create Groups (10 minutes)
Create two groups:

developers
admins

<img width="610" height="137" alt="image" src="https://github.com/user-attachments/assets/286198fd-47a3-421d-b5f3-e761bee93cf4" />

Verify: Check /etc/group

<img width="597" height="115" alt="image" src="https://github.com/user-attachments/assets/a3fe3b4c-b664-44e0-ad24-abe88ddbb11f" />

Task 3: Assign to Groups (15 minutes)

Assign users:
tokyo → developers
berlin → developers + admins (both groups)
professor → admins
Verify: Use appropriate command to check group membership

<img width="808" height="381" alt="image" src="https://github.com/user-attachments/assets/536ae3cf-8cf7-4ae0-93f9-6d0500fa06dc" />


Task 4: Shared Directory (20 minutes)

Create directory: /opt/dev-project
Set group owner to developers
Set permissions to 775 (rwxrwxr-x)
Test by creating files as tokyo and berlin
Verify: Check permissions and test file creation

<img width="920" height="253" alt="image" src="https://github.com/user-attachments/assets/1c0e5ba2-9e7d-4838-b885-3aad8b3d6333" />

Task 5: Team Workspace (20 minutes)

Create user nairobi with home directory
Create group project-team
Add nairobi and tokyo to project-team
Create /opt/team-workspace directory
Set group to project-team, permissions to 775
Test by creating file as nairobi

<img width="995" height="621" alt="image" src="https://github.com/user-attachments/assets/6f03c94f-a418-4657-ad06-9c61267bd064" />

