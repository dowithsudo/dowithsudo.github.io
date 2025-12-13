---
title: Linux Server Hardening
description: A no nonsense linux server hardening article. Practical steps, real commands, and proven practices to secure SSH, firewall, services, and logs without breaking production.

slug: linux-server-hardening
date: 2025-12-13 00:00:00+0000
author: Danu Wijaya
image : cover.webp
categories : Linux
tags:
    - Linux
    - System Administrator
    - Security
    - Devops
weight: 2      # You can add weight to some posts to override the default sorting (date descending)
---

# Linux Server Hardening

*A practical, boring, and reliable approach (which is exactly what we want)*

---

I’ve hardened Linux servers for years.
Web server, database server, jump host, you name it.

One lesson that never changes:

> **Most incidents are not zero-days. They’re misconfigurations.**

So this article is not about “next-gen AI security magic”.
This is about making your Linux server **less attractive**, **less noisy**, and **less forgiving** when someone does something stupid.

Hardening is not a checklist.
It’s a habit.

---

## What We’re Actually Protecting Against

Let’s be realistic.

Most Linux servers get compromised because of:

* Exposed services no one remembers enabling
* Weak SSH setup
* No patching discipline
* Running everything as root
* “Temporary” firewall rules that lived forever

Not because attackers are geniuses.
But because admins are tired, rushed, or undocumented.

So we harden to:

* Reduce attack surface
* Limit blast radius
* Make logs useful when things go sideways

---

## Baseline Assumptions

Before we start, some ground rules:

* OS: Ubuntu Server / Debian-based (adjust if needed)
* You have `root` or `sudo`
* This is a **server**, not a personal laptop
* You’re okay trading convenience for stability

---

## Basic System Hardening

I’ll go layer by layer.
Do not skip steps just because “it works now”.

---

### Update the System

If you skip this, stop here.

```bash
sudo apt update && sudo apt upgrade -y
```

**Why this matters:**

* Most exploits target known vulnerabilities
* Early patching avoids dependency chaos later

Real-world note:
I’ve seen servers breached just because `openssh` was three years old.

---

### Create a Non-Root User

Root is powerful.
Root is also dangerous.

```bash
# Create a new user
sudo adduser danuadmin

# Allow the user to run sudo
sudo usermod -aG sudo danuadmin
```

**What this does:**

* Creates a regular user with limited privileges
* Makes activity tracking clearer in logs

---

### Harden SSH Configuration

Most attacks start here.

Edit SSH configuration:

```bash
sudo nano /etc/ssh/sshd_config
```

Recommended settings:

```conf
# Disable direct root login
PermitRootLogin no

# Disable password-based login
PasswordAuthentication no

# Restrict SSH access to specific users
AllowUsers danuadmin

# Enforce modern protocol
Protocol 2

# Reduce brute-force attempts
MaxAuthTries 3
```

Restart SSH service:

```bash
sudo systemctl restart ssh
```

> **Important**: Test SSH key login before closing your session.

---

### Use SSH Key Authentication

On your local machine:

```bash
# Generate a strong SSH key
ssh-keygen -t ed25519 -C "danu@laptop"
```

Copy the key to the server:

```bash
ssh-copy-id danuadmin@your-server-ip
```

**Why:**

* Keys can’t be brute-forced
* Credentials don’t travel over the network

---

### Enable Firewall (UFW)

Simple. Readable. Reliable.

```bash
# Allow SSH access
sudo ufw allow OpenSSH

# Enable firewall
sudo ufw enable

# Verify rules
sudo ufw status verbose
```

For web servers:

```bash
sudo ufw allow 80
sudo ufw allow 443
```

---

### Disable Unused Services

List listening services:

```bash
sudo ss -tulnp
```

Disable what you don’t need:

```bash
sudo systemctl stop avahi-daemon
sudo systemctl disable avahi-daemon
```

Rule of thumb:

> If you don’t know what it does, it shouldn’t be running.

---

### Install Fail2Ban

Cheap protection. Big noise reduction.

```bash
sudo apt install fail2ban -y
```

Create local config:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

Example SSH jail:

```conf
[sshd]
enabled = true
port = ssh
maxretry = 3
bantime = 1h
```

Restart Fail2Ban:

```bash
sudo systemctl restart fail2ban
```

---

## File Permission Hygiene

Find world-writable files:

```bash
sudo find / -xdev -type f -perm -0002 -print
```

Fix permissions:

```bash
sudo chmod o-w /path/to/file
```

**Why:**

* Prevents privilege escalation
* Protects sensitive configuration files

---

## Kernel Hardening (sysctl)

Create hardening config:

```bash
sudo nano /etc/sysctl.d/99-hardening.conf
```

Add the following:

```conf
# Disable IP source routing
net.ipv4.conf.all.accept_source_route = 0

# Ignore ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

# Enable SYN cookies
net.ipv4.tcp_syncookies = 1
```

Apply settings:

```bash
sudo sysctl --system
```

---

## Logging and Log Rotation

Ensure logs don’t eat your disk:

```bash
sudo apt install logrotate -y
```

Check configuration:

```bash
cat /etc/logrotate.conf
```

No logs means no evidence.
No rotation means future outage.

---

## Automatic Security Updates (Optional)

Install unattended upgrades:

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure unattended-upgrades
```

**Use with care:**

* Test on staging first
* Not all production systems like auto updates

Security is always contextual.

---

## Final Thoughts

Hardening is not paranoia.
It’s predictability.

A hardened server:

* Behaves consistently
* Fails loudly, not silently
* Produces logs you can trust

> Better to be tired during setup
> than exhausted during an incident.

If you want to go deeper:

* CIS Benchmark alignment
* Docker or Kubernetes hardening
* Database-specific hardening

Say the word.
Coffee is ready.
