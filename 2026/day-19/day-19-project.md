### Task 1: Log Rotation Script
Create `log_rotate.sh` that:
1. Takes a log directory as an argument (e.g., `/var/log/myapp`)
2. Compresses `.log` files older than 7 days using `gzip`
3. Deletes `.gz` files older than 30 days
4. Prints how many files were compressed and deleted
5. Exits with an error if the directory doesn't exist

```
#!/bin/bash

LOG_DIR=$1

# Check directory
if [ ! -d "$LOG_DIR" ]; then
  echo "Directory not found!"
  exit 1
fi

compressed=0
deleted=0

# Compress old logs
for file in $(find "$LOG_DIR" -name "*.log" -mtime +7)
do
  gzip "$file"
  ((compressed++))
done

# Delete old gz files
for file in $(find "$LOG_DIR" -name "*.gz" -mtime +30)
do
  rm "$file"
  ((deleted++))
done

echo "Compressed: $compressed"
echo "Deleted: $deleted"
```
<img width="1064" height="185" alt="image" src="https://github.com/user-attachments/assets/7618bf16-e0d1-4158-bf1f-d69a7c36e878" />

---

### Task 2: Server Backup Script
Create `backup.sh` that:
1. Takes a source directory and backup destination as arguments
2. Creates a timestamped `.tar.gz` archive (e.g., `backup-2026-02-08.tar.gz`)
3. Verifies the archive was created successfully
4. Prints archive name and size
5. Deletes backups older than 14 days from the destination
6. Handles errors — exit if source doesn't exist

```
#!/bin/bash

# Check arguments
if [ $# -ne 2 ]; then
  echo "Usage: $0 <source_dir> <backup_dir>"
  exit 1
fi

SOURCE=$1
DEST=$2

# Check if source exists
if [ ! -d "$SOURCE" ]; then
  echo "Error: Source directory does not exist!"
  exit 1
fi

# Create destination if not exists
mkdir -p "$DEST"

# Timestamp
TIMESTAMP=$(date +"%Y-%m-%d")
ARCHIVE="$DEST/backup-$TIMESTAMP.tar.gz"

# Create backup
tar -czf "$ARCHIVE" "$SOURCE"

# Verify backup
if [ $? -ne 0 ]; then
  echo "Error: Backup failed!"
  exit 1
fi

# Get file size
SIZE=$(du -h "$ARCHIVE" | cut -f1)

# Print details
echo "Backup created: $ARCHIVE"
echo "Size: $SIZE"

# Delete old backups (older than 14 days)
find "$DEST" -name "backup-*.tar.gz" -mtime +14 -exec rm {} \;

echo "Old backups cleaned up!"

exit 0
```
<img width="1240" height="170" alt="image" src="https://github.com/user-attachments/assets/0457e32d-3e02-4875-b0c2-436e46d9270c" />

---

### Task 3: Crontab
1. Read: `crontab -l` — what's currently scheduled?
2. Understand cron syntax:
   ```
   * * * * *  command
   │ │ │ │ │
   │ │ │ │ └── Day of week (0-7)
   │ │ │ └──── Month (1-12)
   │ │ └────── Day of month (1-31)
   │ └──────── Hour (0-23)
   └────────── Minute (0-59)
   ```
3. Write cron entries (in your markdown, don't apply if unsure) for:
   - Run `log_rotate.sh` every day at 2 AM
   - Run `backup.sh` every Sunday at 3 AM
   - Run a health check script every 5 minutes
```
$ corntab -l  => No corntab

$ corntab -e  => create
```
---

### Task 4: Combine — Scheduled Maintenance Script
Create `maintenance.sh` that:
1. Calls your log rotation function
2. Calls your backup function
3. Logs all output to `/var/log/maintenance.log` with timestamps
4. Write the cron entry to run it daily at 1 AM

```
#!/bin/bash

LOG_FILE="/var/log/maintenance.log"

echo "----- $(date) -----" >> $LOG_FILE

# Run log rotation
/home/umesh/Documents/Devops/Day-19/log_rotates.sh /var/log/nginx >> $LOG_FILE 2>&1

# Run backup
/home/umesh/Documents/Devops/Day-19/backup.sh /var/log/nginx /home/umesh/backups >> $LOG_FILE 2>&1

echo "Maintenance done" >> $LOG_FILE
```
```
$ corntab -e

# Log rotation - daily at 2 AM
0 2 * * * /home/umesh/Documents/Devops/Day-19/log_rotates.sh /var/log/nginx

# Backup - every Sunday at 3 AM
0 3 * * 0 /home/umesh/Documents/Devops/Day-19/backup.sh /var/logs/nginx

# Backup - every Sunday at 1AM
0 1 * * * /home/umesh/Documents/Devops/Day-19/maintenance.sh
```
<img width="1795" height="782" alt="image" src="https://github.com/user-attachments/assets/4ee9e88b-b432-4b74-b590-ccf40d797922" />

---

