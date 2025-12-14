---
title: Linux File Permision
description: Learn Linux file permissions step by step with practical examples. Understand chmod, chown, numeric modes, and real-world mistakes that often break systems.

slug: linux-file-permision
date: 2025-12-13 00:00:00+0000
author: Danu Wijaya
image : cover.webp
categories : Linux
tags:
    - Linux
    - System Administrator
    - Security
    - Devops
#weight: 2      # You can add weight to some posts to override the default sorting (date descending)
---

# Learning Linux File Permissions (Without Losing Your Sanity)

I’ve broken enough systems to learn one thing the hard way: **file permissions in Linux are not optional knowledge**.

Sooner or later, something will not start, a service will refuse to read a config file, or a script will work perfectly… until you move it to production.

When that happens, I don’t guess.
I check logs, then permissions.
Usually in that order, while sipping black coffee.

This article is a practical, real-world guide to Linux file permissions.
Not academic. Not fancy.
Just the stuff that actually saves your night.

---

## Why File Permissions Matter (In Real Life)

Think of Linux permissions like **office access cards**.

* Some people can enter the building
* Some can only enter certain rooms
* Some should never be there at all

If permissions are wrong:

* Web server can’t read files
* Backup scripts fail silently
* Config files get exposed
* Or worse, writable by everyone

Security incidents often start with something simple.
Like `chmod 777` done in a hurry.

Yes, I’ve seen it.
More than once.

---

## The Basics: Ownership and Permission Model

Every file and directory in Linux has:

* **Owner** (user)
* **Group**
* **Others** (everyone else)

And three basic permissions:

* **r** = read
* **w** = write
* **x** = execute

Let’s look at a real example.

```bash
ls -l example.sh
```

Example output:

```text
-rwxr-x--- 1 danu devops 1024 Mar 10 10:00 example.sh
```

Breakdown:

* `-` : regular file
* `rwx` : owner permissions
* `r-x` : group permissions
* `---` : others permissions

Meaning:

* Owner can read, write, execute
* Group can read and execute
* Others get nothing

This is boring.
And very important.

---

## Numeric (Octal) Permissions Explained

If symbolic permissions feel abstract, numeric ones are just math.
Simple math.

| Permission | Value |
| ---------- | ----- |
| read (r)   | 4     |
| write (w)  | 2     |
| execute(x) | 1     |

Add them up:

* `7` = 4+2+1 = rwx
* `6` = 4+2   = rw-
* `5` = 4+1   = r-x
* `4` = 4     = r--

Example:

```bash
chmod 750 example.sh
```

Meaning:

* Owner: 7 (rwx)
* Group: 5 (r-x)
* Others: 0 (---)

This is one of my default choices for scripts.
Enough access to run.
Not enough to cause drama.

---

## Step-by-Step: Checking Permissions

### Step 1: Identify the Problem

Something fails.
Don’t panic.
Check logs first.

```bash
journalctl -xe
```

Or application logs.
Permissions issues usually scream pretty clearly.

---

### Step 2: Inspect File Permissions

```bash
ls -l /path/to/file
```

For directories:

```bash
ls -ld /path/to/directory
```

Remember:

* Files need **read** to be read
* Directories need **execute** to be accessed

This trips people up.
A lot.

---

## Step-by-Step: Changing Permissions Safely

### Using chmod (Symbolic)

```bash
chmod u+x script.sh
# u+x  : add execute permission for the owner only
```

```bash
chmod g-w config.conf
# g-w  : remove write permission from group
```

This is safer when you don’t want surprises.

---

### Using chmod (Numeric)

```bash
chmod 640 config.conf
# 6 (rw-) owner can read and write
# 4 (r--) group can read
# 0 (---) others get nothing
```

For sensitive configs, this is my go-to.
Especially on production servers.

---

## Ownership: chown and chgrp

Permissions alone are useless if ownership is wrong.

### Change Owner and Group

```bash
chown danu:devops example.sh
# danu    : new owner
# devops  : new group
```

Recursive (be careful):

```bash
chown -R www-data:www-data /var/www/app
# -R applies changes recursively
# Always double-check the path
```

Recursive commands are powerful.
And unforgiving.
Like production.

---

## Directories: The Special Case

Directories behave differently.

* **r** : list files
* **w** : create/delete files
* **x** : enter directory

Example:

```bash
chmod 750 /data/backups
# Owner can manage backups
# Group can access
# Others locked out
```

No execute permission?
You can see the directory.
But you can’t enter it.

Linux is very literal.

---

## Common Mistakes I Still See

* Using `777` as a quick fix
* Forgetting execute permission on directories
* Running services as root unnecessarily
* No documentation about why permissions were changed

Quick fixes age badly.
Especially at 2 AM.

---

## A Simple Permission Checklist (Realistic One)

Before moving to production:

* Who owns the file?
* Which service actually needs access?
* Read-only or writable?
* Directory permissions checked?
* Documented?

Security is a process.
Not a checkbox.

---

## Final Thoughts

Linux file permissions look simple.
Until they aren’t.

Learn them early.
Use them deliberately.
Document your decisions.

It’s cheaper than an incident.
And easier than explaining to management why a config file was world-writable.

Now excuse me.
My coffee is getting cold.
