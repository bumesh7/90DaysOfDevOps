Part 1: Linux File System Hierarchy (30 minutes)

Document the purpose of these essential directories:

Core Directories (Must Know):

    / (root) - The starting point of everything
    
    -> The top-level directory of the entire filesystem. All other directories branch from here.
    -> I will use to navigate the full filesystem structure or understand how directories are organized from the top level.
    ls -l 
    bin => /usr/bin
    home
    dev
    
    /home - User home directories

    -> Each user have there own directory here.
    ls -l /home
    umesh/
    sharp/
    
    /root - Root user's home directory

    -> The home directory for the root user. 
    ls -l /root
    .bashrc
    scripts/
    -> use when need to manage administrative scripts or configurations.
    
    /etc - Configuration files

    -> System-wide configuration files for the OS and installed services.
    ls -l /etc
    passwd – User account information
    ssh/ – SSH server/client configuration directory
    -> use to modify system or service configuration (like SSH, networking, or user settings).
    
    /var/log - Log files (very important for DevOps!)

    -> System and service log files used for monitoring, debugging, and troubleshooting.
    ls -l /var/log
    syslog – General system activity log
    auth.log – Authentication and login attempts
    -> use to troubleshoot errors, check login attempts, or investigate system issues.
    
    /tmp - Temporary files

    -> Temporary files created by the system and applications. Files here may be automatically deleted on reboot.
    ls -l /tmp
    systemd-private-xxxx/ – Temporary service directories
    tmp.XXXXXX – Application-generated temporary files
    -> use when a temporary location to store files during script execution or testing.

Additional Directories (Good to Know):

    /bin - Essential command binaries

    -> binary files required for system functionality.
    ls -l /bin
    ls - list files
    cp - copy
    -> use to run essential system commands.
    
    /usr/bin - User command binaries

    -> Non-essential user-level command binaries and applications.
    ls -l /usr/bin
    python3
    git
    -> use to run standard user applications or development tools.
    
    /opt - Optional/third-party applications

    -> Optional software and third-party applications that are not part of the default system installation.
    ls -l /opt
    google
    -> used to manage or inspect manually installed third-party software.

For each directory:

    Write 1-2 lines explaining what it contains
    Run ls -l <directory> and note 1-2 files/folders you see
    Write one sentence: "I would use this when..."

Hands-on task:

# Find the largest log file in /var/log
du -sh /var/log/* 2>/dev/null | sort -h | tail -5
<img width="1127" height="192" alt="image" src="https://github.com/user-attachments/assets/fbb0902d-f4dc-4360-b512-1393f45eaa71" />

# Look at a config file in /etc
cat /etc/hostname
umesh-Aspire-A515-57G

# Check your home directory
ls -la ~
<img width="1127" height="688" alt="image" src="https://github.com/user-attachments/assets/56208142-6269-42bd-af05-1b8de5050155" />

Part 2: Scenario-Based Practice (40 minutes)

Important: Focus on understanding the troubleshooting flow, not memorizing commands. Use the hints!
SOLVED EXAMPLE: Understanding How to Approach Scenarios

Example Scenario: Check if a service is running

Question: How do you check if the 'nginx' service is running?

My Solution (Step by step):

Step 1: Check service status

systemctl status nginx

Why this command? It shows if the service is active, failed, or stopped

Step 2: If service is not found, list all services

systemctl list-units --type=service

Why this command? To see what services exist on the system

Step 3: Check if service is enabled on boot

systemctl is-enabled nginx

Why this command? To know if it will start automatically after reboot

What I learned: Always check status first, then investigate based on what you see.

Now try these scenarios yourself:

Scenario 1: Service Not Starting

A web application service called 'myapp' failed to start after a server reboot.
What commands would you run to diagnose the issue?
Write at least 4 commands in order.

$ sudo systemctl status myapp => check status
$ journalctl -u myapp -n 50  => check logs
$ sudo systemctl is-enabled myapp => check if the myapp start on reboot
$ sudo systemctl --failed => Check if the service failed during boot
$ sudo systemctl restart myapp => restart

Hint:

    First check: Is the service running or failed?
    Then check: What do the logs say?
    Finally check: Is it enabled to start on boot?

Commands to explore: systemctl status myapp, systemctl is-enabled myapp, journalctl -u myapp -n 50

Scenario 2: High CPU Usage

Your manager reports that the application server is slow.
You SSH into the server. What commands would you run to identify
which process is using high CPU?

$ htop => shows all the cpu and memory usage and can scroll top and down
$ ps aux --sort=-%cpu | head -10 => shows all running processor and sort by cpu of top 10
$ ps -fp <PID> => Get more details about a specific PID

Hint:

    Use a command that shows live CPU usage
    Look for processes sorted by CPU percentage
    Note the PID (Process ID) of the top process

Commands to explore: top (press 'q' to quit), htop, ps aux --sort=-%cpu | head -10

Resource: Review Day 05 (Troubleshooting Drill - CPU & Memory section)

Scenario 3: Finding Service Logs

A developer asks: "Where are the logs for the 'docker' service?"
The service is managed by systemd.
What commands would you use?

$ sudo systemctl status docker
$ journalctl -u docker -n 10 => lsit 10 lines
$ journalctl -u docker -f => show live logs
$ journalctl -u docker -b => show logs after boot
$ journalctl -u docker --no-pager => show logs with few details

Hint:

    systemd services → logs are in journald
    Command pattern: journalctl -u <service-name>
    Use -n flag to limit number of lines
    Use -f flag to follow logs in real-time (like tail -f)

Commands to explore:

# Check service status first
systemctl status ssh

# View last 50 lines of logs
journalctl -u ssh -n 50

# Follow logs in real-time
journalctl -u ssh -f

Resource: Review Day 04 (Process and Services - Log checks section)

Scenario 4: File Permissions Issue

A script at /home/user/backup.sh is not executing.
When you run it: ./backup.sh
You get: "Permission denied"

$ pwd
$ cd /home/user/
$ pwd
$ ls -l backup.sh
$ chmode +x backup.sh
$ ./backup.sh

What commands would you use to fix this?

Hint:

    First: Check what permissions the file has
    Understand: Files need 'x' (execute) permission to run
    Fix: Add execute permission with chmod

Step-by-step solution structure:

Step 1: Check current permissions
Command: ls -l /home/user/backup.sh
Look for: -rw-r--r-- (notice no 'x' = not executable)

Step 2: Add execute permission
Command: chmod +x /home/user/backup.sh

Step 3: Verify it worked
Command: ls -l /home/user/backup.sh
Look for: -rwxr-xr-x (notice 'x' = executable)

Step 4: Try running it
Command: ./backup.sh

Resource: Review Day 02 (File Permissions and Users Management)
