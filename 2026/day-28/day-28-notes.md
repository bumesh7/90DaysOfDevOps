## Challenge Tasks

### Task 1: Self-Assessment Checklist
Go through the checklist below. For each item, mark yourself honestly:
- **Can do confidently**
- **Need to revisit**
- **Haven't done yet**

#### Linux
- [ ] Navigate the file system, create/move/delete files and directories
- [ ] Manage processes — list, kill, background/foreground
- [ ] Work with systemd — start, stop, enable, check status of services
- [ ] Read and edit text files using vi/vim or nano
- [ ] Troubleshoot CPU, memory, and disk issues using top, free, df, du
- [ ] Explain the Linux file system hierarchy (/, /etc, /var, /home, /tmp, etc.)
- [ ] Create users and groups, manage passwords
- [ ] Set file permissions using chmod (numeric and symbolic)
- [ ] Change file ownership with chown and chgrp
- [ ] Create and manage LVM volumes
- [ ] Check network connectivity — ping, curl, netstat, ss, dig, nslookup
- [ ] Explain DNS resolution, IP addressing, subnets, and common ports

#### Shell Scripting
- [ ] Write a script with variables, arguments, and user input
- [ ] Use if/elif/else and case statements
- [ ] Write for, while, and until loops
- [ ] Define and call functions with arguments and return values
- [ ] Use grep, awk, sed, sort, uniq for text processing
- [ ] Handle errors with set -e, set -u, set -o pipefail, trap
- [ ] Schedule scripts with crontab

#### Git & GitHub
- [ ] Initialize a repo, stage, commit, and view history
- [ ] Create and switch branches
- [ ] Push to and pull from GitHub
- [ ] Explain clone vs fork
- [ ] Merge branches — understand fast-forward vs merge commit
- [ ] Rebase a branch and explain when to use it vs merge
- [ ] Use git stash and git stash pop
- [ ] Cherry-pick a commit from another branch
- [ ] Explain squash merge vs regular merge
- [ ] Use git reset (soft, mixed, hard) and git revert
- [ ] Explain GitFlow, GitHub Flow, and Trunk-Based Development
- [ ] Use GitHub CLI to create repos, PRs, and issues

---

### Task 2: Revisit Your Weak Spots
1. Pick **3 topics** from the checklist where you marked "Need to revisit"
2. Go back to that day's challenge and redo the hands-on tasks
3. Document what you re-learned in `day-28-notes.md`
```
Task 1 and 2 I can do it
```
---

### Task 3: Quick-Fire Questions
Answer these from memory (no Googling). Then verify your answers:

1. What does `chmod 755 script.sh` do?
2. What is the difference between a process and a service?
3. How do you find which process is using port 8080?
4. What does `set -euo pipefail` do in a shell script?
5. What is the difference between `git reset --hard` and `git revert`?
6. What branching strategy would you recommend for a team of 5 developers shipping weekly?
7. What does `git stash` do and when would you use it?
8. How do you schedule a script to run every day at 3 AM?
9. What is the difference between `git fetch` and `git pull`?
10. What is LVM and why would you use it instead of regular partitions?

```
1. What does `chmod 755 script.sh` do? — Sets permissions so owner can read/write/execute and others can read/execute

2. What is the difference between a process and a service? — A process is a running program instance, while a service is a background process managed by the system

3. How do you find which process is using port 8080? — Use `lsof -i :8080` or `netstat -tulnp | grep 8080`

4. What does `set -euo pipefail` do in a shell script? — It makes the script exit on errors, undefined variables, and failed pipes

5. What is the difference between `git reset --hard` and `git revert`? — `git reset --hard` rewrites history and discards changes, while `git revert` creates a new commit to undo changes

6. What branching strategy would you recommend for a team of 5 developers shipping weekly? — Use a simple feature branch + main branch strategy with pull requests

7. What does `git stash` do and when would you use it? — Temporarily saves uncommitted changes to switch context without committing

8. How do you schedule a script to run every day at 3 AM? — Use cron job: `0 3 * * * /path/to/script.sh`

9. What is the difference between `git fetch` and `git pull`? — `git fetch` downloads changes without merging, while `git pull` fetches and merges

10. What is LVM and why would you use it instead of regular partitions? — LVM allows flexible disk management like resizing and snapshots unlike fixed partitions

```
---

### Task 4: Organize Your Work
1. Make sure all your daily submissions (day-1 through day-27) are committed and pushed
2. Check that your `git-commands.md` is up to date
3. Check that your shell scripting cheat sheet is complete
4. Verify your GitHub profile and repos are clean (from Day 27)

```
Yes
```
---

### Task 5: Teach It Back
Pick **one topic** you've learned and write a short explanation (5-10 lines) as if you're teaching it to someone who has never heard of it. Add it to your `day-28-notes.md`.

Examples:
- Explain Git branching to a non-developer
- Explain file permissions to a new Linux user
- Explain what a crontab is and why sysadmins use it
```
A crontab (cron table) is a configuration file used by the cron scheduler in Unix/Linux systems to define and automate tasks that run at specific times or intervals. 
Sysadmins use it to automate repetitive jobs like backups, log rotation, monitoring scripts, and system maintenance.
Ensuring tasks run reliably without manual intervention.
```

Teaching is the best test of understanding.

```
Done
```
---
