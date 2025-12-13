---
title: Backup Database Using rclone
description: Practical step by step guide to backing up MySQL databases using rclone. Covers scripting, logging, cron jobs, and real-world best practices for reliable offsite backups.

slug: backup-database-using-rclone
date: 2025-12-13 00:00:00+0000
author: Danu Wijaya
image : cover.webp
categories : Linux
tags:
    - Linux
    - System Administrator
    - Devops
#weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

# Backup MySQL Database Using rclone

I’ve been doing backups long enough to know one thing, ff you don’t trust your backup, you don’t actually have one.
This guide is written from the perspective of a sysadmin who prefers logs over assumptions, and boring but stable solutions over fancy setups that break at 2 AM.

We’ll back up a MySQL database, compress it, and ship it offsite using **rclone**. simple, predictable, auditable.

Coffee optional, but recommended.

---

## Why rclone?

Short story.

I’ve used rsync, scp, cron jobs with hope as a strategy, and even "temporary" scripts that somehow survived for years.

rclone stuck because:

* Works with S3, GCS, Azure, Backblaze, Google Drive, etc.
* CLI friendly. Logs make sense.
* Retry logic is sane.
* Encryption is available if you need it.

It doesn’t try to be clever, that’s a feature.

---

## Assumptions

Let’s be clear before touching anything:

* Linux server
* MySQL or MariaDB
* You have shell access
* rclone is already installed and configured
* You already tested `rclone ls remote:bucket`

If rclone is not configured yet, stop here, configure it properly first. Future you will say thanks.

---

## Directory Structure

I like predictable paths.
Nothing exotic.

```bash
/opt/backup/
├── mysql/
│   ├── dumps/
│   ├── logs/
│   └── scripts/
```

Create it once.
Document it.
Move on.

```bash
mkdir -p /opt/backup/mysql/{dumps,logs,scripts}
```

---

## MySQL Credentials (Non-interactive)

Please don’t hardcode passwords in scripts.
I’ve cleaned up too many incidents caused by that.

Create a MySQL option file:

```bash
nano /root/.my.cnf
```

```ini
[client]
user=backupuser
password=VerySecretPassword
host=localhost
```

```bash
chmod 600 /root/.my.cnf
```

This allows `mysqldump` to run without exposing credentials in process lists.
Security is a process, remember.

---

## Backup Script

This is the heart of the setup.
Nothing fancy.
Just reliable.

Create the script:

```bash
nano /opt/backup/mysql/scripts/mysql_backup_rclone.sh
```

```bash
#!/bin/bash

# Exit immediately if a command exits with a non-zero status
# Better to fail fast than silently produce broken backups
set -e

# -----------------------------
# Basic variables
# -----------------------------
DATE=$(date +"%Y-%m-%d_%H-%M")
HOSTNAME=$(hostname -s)
BACKUP_DIR="/opt/backup/mysql/dumps"
LOG_DIR="/opt/backup/mysql/logs"
LOG_FILE="$LOG_DIR/backup-$DATE.log"

# MySQL dump file name
DUMP_FILE="$BACKUP_DIR/mysql-$HOSTNAME-$DATE.sql.gz"

# rclone remote target
# Example: s3-backup:prod-mysql
RCLONE_REMOTE="remote:bucket/mysql"

# -----------------------------
# Logging setup
# -----------------------------
# Redirect all output (stdout and stderr) to log file
exec > >(tee -a "$LOG_FILE") 2>&1

# -----------------------------
# Start backup
# -----------------------------
echo "[$(date)] Starting MySQL backup"

# Dump all databases
# --single-transaction avoids table locking for InnoDB
# --routines and --events are included because they matter
mysqldump --all-databases \
  --single-transaction \
  --routines \
  --events \
  --triggers \
  | gzip > "$DUMP_FILE"

# Basic sanity check
if [ ! -s "$DUMP_FILE" ]; then
  echo "[$(date)] Backup file is empty. Aborting."
  exit 1
fi

echo "[$(date)] MySQL dump completed: $DUMP_FILE"

# -----------------------------
# Upload to remote storage
# -----------------------------
echo "[$(date)] Uploading backup to rclone remote"

# --checksum ensures data integrity
# --log-level INFO is usually enough
rclone copy "$DUMP_FILE" "$RCLONE_REMOTE" \
  --checksum \
  --log-level INFO

echo "[$(date)] Upload completed"

# -----------------------------
# Cleanup old local backups
# -----------------------------
# Keep last 7 days locally
find "$BACKUP_DIR" -type f -mtime +7 -delete

echo "[$(date)] Cleanup completed"

echo "[$(date)] Backup finished successfully"
```

Make it executable:

```bash
chmod +x /opt/backup/mysql/scripts/mysql_backup_rclone.sh
```

---

## Script Notes (Real World Context)

Some design choices worth explaining:

* `set -e` forces the script to stop on errors
* Logs are not optional. They are your first responder
* `--single-transaction` keeps production impact low
* Compression happens inline to save disk I/O
* No database list hardcoded. Less maintenance

This script is boring.
That’s intentional.

---

## Cron Job

Backups that rely on memory are not backups.

Edit crontab:

```bash
crontab -e
```

Example: daily at 01:30 AM

```bash
30 1 * * * /opt/backup/mysql/scripts/mysql_backup_rclone.sh
```

Tip from experience:
Schedule backups when disk IO and replication lag are calm.
Your future alerts will be quieter.

---

## Restore Test (Please Do This)

At least once.
Preferably before production needs it.

```bash
gunzip mysql-hostname-date.sql.gz
mysql < mysql-hostname-date.sql
```

If restore works, backup is valid.
If not, congratulations.
You just found a problem early.

---

## Final Thoughts

This setup won’t win awards.
But it will survive audits, incidents, and staff rotation.

* Logs are readable
* Scripts are understandable
* Dependencies are minimal

I’ve seen prettier solutions fail harder.

When the database crashes at 3 AM,
this is the kind of backup you want waiting for you.

Now grab that coffee.
You earned it.
