Task 1: Create Files (10 minutes)
Create empty file devops.txt using touch
Create notes.txt with some content using cat or echo
Create script.sh using vim with content: echo "Hello DevOps"
Verify: ls -l to see permissions

<img width="891" height="327" alt="image" src="https://github.com/user-attachments/assets/b77f2818-627f-4df9-9f00-1f267ed6d7d9" />

Task 2: Read Files (10 minutes)
Read notes.txt using cat
View script.sh in vim read-only mode
Display first 5 lines of /etc/passwd using head
Display last 5 lines of /etc/passwd using tail

<img width="660" height="361" alt="image" src="https://github.com/user-attachments/assets/0c131071-8d56-473f-bc71-57e24ff2707a" />


Task 3: Understand Permissions (10 minutes)
Format: rwxrwxrwx (owner-group-others)

r = read (4), w = write (2), x = execute (1)
Check your files: ls -l devops.txt notes.txt script.sh

<img width="658" height="148" alt="image" src="https://github.com/user-attachments/assets/5579b68b-ab87-437f-bb1e-7c3dcbdb922a" />


Answer: What are current permissions? Who can read/write/execute?


Task 4: Modify Permissions (20 minutes)

Make script.sh executable → run it with ./script.sh
Set devops.txt to read-only (remove write for all)
Set notes.txt to 640 (owner: rw, group: r, others: none)
Create directory project/ with permissions 755
Verify: ls -l after each change

<img width="692" height="263" alt="image" src="https://github.com/user-attachments/assets/d6bff019-6687-4e1a-beb1-0110f514b77f" />


Task 5: Test Permissions (10 minutes)
Try writing to a read-only file - what happens

<img width="567" height="37" alt="image" src="https://github.com/user-attachments/assets/fa0869b4-3189-41ee-abca-f2c5bde59f45" />

Try executing a file without execute permission

<img width="615" height="67" alt="image" src="https://github.com/user-attachments/assets/6bac9444-0e6a-43fa-a154-21589a2834e1" />

Document the error messages



![20260203_183451](https://github.com/user-attachments/assets/6cd131d0-2f31-409d-97cb-486f01199fb9)

![20260203_183502](https://github.com/user-attachments/assets/dad6c53f-946d-495e-a540-9fa1a916c56f)
