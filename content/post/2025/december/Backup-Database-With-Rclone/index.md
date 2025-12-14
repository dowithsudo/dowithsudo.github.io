---
title: Backup Database To Google Drive With Rclone
description: Practical guide to MySQL backup automation using rclone and Google Drive. Includes mysqldump, compression, logging, and cron based scheduling.

slug: backup-database-to-google-drive-with-rclone
date: 2025-12-14 00:00:00+0000
author: Danu Wijaya
image : cover.webp
categories : Linux
tags:
    - Linux
    - System Administrators
#weight: 2      # You can add weight to some posts to override the default sorting (date descending)
---

# Backup MySQL Database to Google Drive Using rclone

I have broken more backups than I want to admit. Not because the tools were bad, but because the process was vague, undocumented, or too clever for its own good.

This article is about a **boring but reliable** way to back up a MySQL database and push it to Google Drive using `rclone`. Nothing fancy. Just something you can trust at 02:00 AM when the alert tone is already louder than your coffee machine.

I assume you are running Linux, you have shell access, and you prefer logs over hope.

---

## Why rclone + Google Drive

Short answer: it works.

Longer answer:

* Google Drive is cheap, redundant, and usually already approved by management
* rclone is stable, scriptable, and well documented
* No vendor lock-in feeling
* Easy to rotate, easy to audit

Think of it like this:

* `mysqldump` is the guy packing your stuff
* `gzip` is the vacuum bag
* `rclone` is the delivery truck

Each does one job. That is already a good sign.

---

## What We Are Building

At the end, you will have:

* A MySQL dump
* Compressed to save space
* Uploaded to Google Drive
* Logged properly
* Ready to be scheduled via cron

No magic. No GUI. No guessing.

---

## Prerequisites

Before touching anything, make sure these exist:

* Linux server (Ubuntu, Debian, Alma, Rocky, etc)
* MySQL or MariaDB
* A Google account
* Root or sudo access
* Coffee. Black. Optional but recommended

Check binaries first:

```bash
which mysqldump gzip rclone
```

If something is missing, install it. Do not continue until this is clean.

---

## Step 1: Install rclone

On most modern Linux systems:

```bash
curl https://rclone.org/install.sh | sudo bash
```

Verify installation:

```bash
rclone version
```

If this fails, stop here. Debug first. Backup scripts built on broken tools are just future incidents.

---

## Step 2: Configure rclone for Google Drive

Run interactive config:

```bash
rclone config
```

Basic flow:

* Choose `n` for new remote
* Name it something sane, example: `gdrive-backup`
* Storage type: `drive`
* Client ID and secret: leave empty unless you know why you should not
* Scope: full access (default)
* Auto config: `yes`

Your browser will open. Login. Approve.

Test it:

```bash
rclone lsd gdrive-backup:
```

If you see folders, good. If not, fix this before moving on.

---

## Step 3: Prepare Directory Structure

I like predictable paths. Future me appreciates this.

```bash
mkdir -p /opt/backup/mysql
mkdir -p /var/log/backup
```

Why:

* `/opt/backup/mysql` for dump files
* `/var/log/backup` for logs

No guessing later.

---

## Step 4: Create the Backup Script

Create the script:

```bash
nano /opt/backup/mysql/mysql_backup_to_gdrive.sh
```

Full script below. Read it. Do not just copy blindly.

```bash
#!/bin/bash

# Exit immediately if a command exits with a non-zero status
# This prevents silent failures
set -e

# Variables
DATE=$(date +"%Y-%m-%d_%H-%M-%S")
BACKUP_DIR="/opt/backup/mysql"
LOG_FILE="/var/log/backup/mysql_backup.log"
DB_NAME="your_database_name"
DB_USER="backup_user"
DB_PASS="strong_password"
REMOTE_NAME="gdrive-backup"
REMOTE_DIR="mysql-backup"

# Log start time
echo "[$(date)] Backup started" >> "$LOG_FILE"

# Create MySQL dump
# --single-transaction avoids table locking for InnoDB
# --routines and --triggers are often forgotten, so we include them
mysqldump \
  --user="$DB_USER" \
  --password="$DB_PASS" \
  --single-transaction \
  --routines \
  --triggers \
  "$DB_NAME" \
  | gzip > "$BACKUP_DIR/${DB_NAME}_${DATE}.sql.gz"

# Upload to Google Drive using rclone
# copy is safer than sync for backups
rclone copy \
  "$BACKUP_DIR" \
  "$REMOTE_NAME:$REMOTE_DIR" \
  --log-file="$LOG_FILE" \
  --log-level INFO

# Optional: remove local backups older than 7 days
# Adjust this if your disk is small
find "$BACKUP_DIR" -type f -mtime +7 -name "*.sql.gz" -delete

# Log end time
echo "[$(date)] Backup finished" >> "$LOG_FILE"
```

### Script Explanation (Important)

Key parts that matter in real life:

* `set -e`

  * Script stops on error instead of pretending everything is fine

* `--single-transaction`

  * Keeps dump consistent without locking tables

* `gzip`

  * Saves space and bandwidth

* `rclone copy` instead of `sync`

  * Backup should only add, not delete remotely

* Log file

  * When something breaks, logs talk. Scripts do not.

---

## Step 5: Secure the Script

Make it executable:

```bash
chmod 700 /opt/backup/mysql/mysql_backup_to_gdrive.sh
```

Why:

* Database credentials are inside
* Least privilege is not optional

If possible, consider:

* Using a `.my.cnf` file
* Or a dedicated MySQL backup user with limited privileges

Security is a process, not a checkbox.

---

## Step 6: Test Manually

Run it once:

```bash
/opt/backup/mysql/mysql_backup_to_gdrive.sh
```

Check:

* Local backup file exists
* File exists in Google Drive
* Log file has no errors

If something fails, fix it now. Do not postpone debugging.

---

## Step 7: Schedule with Cron

Edit crontab:

```bash
crontab -e
```

Example, daily at 01:30 AM:

```bash
30 1 * * * /opt/backup/mysql/mysql_backup_to_gdrive.sh
```

After that:

* Monitor logs for a few days
* Verify file sizes
* Try restoring once (yes, really)

A backup never tested is just a theory.

---

## Common Pitfalls I See Too Often

* Backup runs, but never tested restore
* Using `sync` and accidentally deleting old backups
* No log rotation
* Credentials with full root access
* Assuming Google Drive means infinite retention

None of these look dangerous. Until they are.

---

## Closing Notes

This setup is not glamorous.

But it is:

* Predictable
* Auditable
* Easy to explain to the next admin

If you want encryption, retention policies, or multi-region strategy, build on top of this. Do not replace it with something you do not fully understand.

When choosing between looking smart and being useful, I choose useful. Every time.

Now go document this. Your future self will thank you.
