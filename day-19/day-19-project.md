### Day 19 task

# Task1: Log Rotation Script

```bash
#!/bin/bash
set -euo pipefail

# Take directory from user
read -p "Enter the log directory: " LOG_DIR

# Check if directory exists
if [ ! -d "$LOG_DIR" ]; then
    echo "Error: Directory does not exist: $LOG_DIR"
    exit 1
fi

# Find .log files older than 7 days
compress_files=$(find "$LOG_DIR" -type f -name "*.log" -mtime +7)

# Count files to compress
compress_count=$(echo "$compress_files" | grep -c . || true)

# Compress them
for file in $compress_files
do
    gzip "$file"
done

# Find .gz files older than 30 days
delete_files=$(find "$LOG_DIR" -type f -name "*.gz" -mtime +30)

# Count files to delete
delete_count=$(echo "$delete_files" | grep -c . || true)

# Delete them
for file in $delete_files
do
    rm -f "$file"
done

echo "Files compressed: $compress_count"
echo "Files deleted: $delete_count"
```

# Task2: Server Backup Script

```bash
#!/bin/bash
set -euo pipefail

SRC_DIR="$1"
DEST_DIR="$2"

# Check if source directory exists
if [ ! -d "$SRC_DIR" ]; then
    echo "Error: Source directory does not exist."
    exit 1
fi

# Create destination directory if it doesn't exist
mkdir -p "$DEST_DIR"

# Create timestamp
TIMESTAMP=$(date +%Y-%m-%d)

# Archive name
ARCHIVE="backup-$TIMESTAMP.tar.gz"

# Full path
ARCHIVE_PATH="$DEST_DIR/$ARCHIVE"

# Create archive
tar -czf "$ARCHIVE_PATH" -C "$SRC_DIR" .

# Verify archive creation
if [ ! -f "$ARCHIVE_PATH" ]; then
    echo "Error: Backup failed."
    exit 1
fi

# Print archive name and size
SIZE=$(du -h "$ARCHIVE_PATH" | cut -f1)
echo "Backup created: $ARCHIVE"
echo "Size: $SIZE"

# Delete backups older than 14 days
find "$DEST_DIR" -type f -name "backup-*.tar.gz" -mtime +14 -delete

echo "Old backups (older than 14 days) removed."
```

# Task3: Crontab
```bash
# To view current cron jobs
crontab -l
crontab -l
# Edit this file to introduce tasks to be run by cron.
#
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
#
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
#
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
#
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
#
# For more information see the manual pages of crontab(5) and cron(8)
#
# m h  dom mon dow   command

0 2 * * * /home/rahul/scripts/log_rotate.sh
0 3 * * 0 /home/rahul/scripts/backup.sh
*/5 * * * * /home/rahul/scripts/health_check.sh
```

# Task4: Combined Maintenance Script

```bash
#!/bin/bash
set -euo pipefail

LOG_FILE="/var/log/maintenance.log"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$LOG_FILE"
}

rotate_logs() {
    LOG_DIR="/var/log/nginx"
    log "Starting log rotation..."

    find "$LOG_DIR" -type f -name "*.log" -mtime +7 -exec gzip {} \;

    find "$LOG_DIR" -type f -name "*.gz" -mtime +30 -delete

    log "Log rotation completed."
}

backup() {
    SRC_DIR="/home/rahul/scripts"
    DEST_DIR="/home/rahul/backups"

    mkdir -p "$DEST_DIR"

    TIMESTAMP=$(date +%Y-%m-%d)
    ARCHIVE="$DEST_DIR/backup-$TIMESTAMP.tar.gz"

    log "Starting backup..."

    tar -czf "$ARCHIVE" "$SRC_DIR"

    SIZE=$(du -h "$ARCHIVE" | cut -f1)

    log "Backup created: $ARCHIVE (Size: $SIZE)"
}

main() {
    log "Maintenance job started"
    rotate_logs
    backup
    log "Maintenance job completed"
}

main
```

What I learned:
1. How to use `find` to locate files based on age and type, and how to execute commands on those files.
2. How to create tar.gz archives and verify their creation, as well as how to manage backup files by deleting old ones.
3. How to schedule tasks using cron and understand the syntax for timing and commands.
