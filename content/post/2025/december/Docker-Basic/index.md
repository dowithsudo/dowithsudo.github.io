---
title: Basic Docker
description: Practical beginner guide to Docker written from real-world IT operations experience. Learn core concepts, step by step setup, Dockerfile basics, containers, volumes, and troubleshooting with a focus on stability, logs, and best practices.

slug: basic-docker
date: 2025-12-13 00:00:00+0000
author: Danu Wijaya
image : cover.webp
categories : Devops
tags:
    - Linux
    - System Administrator
    - Security
    - Devops
#weight: 2      # You can add weight to some posts to override the default sorting (date descending)
---

## Why Docker Exists (From Real Life)

A few years back, I got the classic line:

> "It works on my laptop."

Different OS, different library versions, different configs. Same app, different behavior. Logs didn’t lie, but they didn’t help much either.

Docker showed up as a **packaging standard**.

* Same app
* Same dependencies
* Same behavior

From dev laptop to production server.

Not magic. Just consistency.

---

## What Docker Actually Is

Let’s keep it grounded.
Docker is a tool that lets you package an application together with everything it needs to run (code, libraries, config) into a single unit called a container.

### Container vs VM

Docker **containers** are not virtual machines.

* VM

  * Full OS
  * Heavy
  * Slower to boot

* Container

  * Shares host kernel
  * Lightweight
  * Starts in seconds

Think of containers as **isolated processes with rules**.

---

## Core Concepts You Must Understand

No buzzwords. Just essentials.

### Image

* Blueprint
* Read-only
* Built once, run many times

Example:

* `nginx:latest`
* `mysql:8.0`

### Container

* Running instance of an image
* Has lifecycle

Create → Start → Stop → Remove

### Dockerfile

* Recipe
* Defines how an image is built

If documentation is missing, Dockerfile becomes your last line of defense.

### Volume

* Persistent data
* Survives container removal

Databases without volumes are just ticking time bombs.

---

## Installing Docker (Linux Example)

I mostly work on Linux servers. Ubuntu in this case.

```bash
# Update package index
# Keeps local package list in sync
sudo apt update

# Install required packages
# Allows apt to use repositories over HTTPS
sudo apt install -y ca-certificates curl gnupg

# Add Docker official GPG key
# Used to verify Docker packages authenticity
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Add Docker repository
# Tells apt where to download Docker packages
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

Quick sanity check:

```bash
# Check Docker version
# Confirms Docker is installed and reachable
docker --version
```

If this fails, stop here. Check logs. Don’t continue blindly.

---

## Your First Container

Let’s start boring. Nginx.

```bash
# Run nginx container
# -d     : run in background
# -p     : map host port to container port
# --name : easier to reference later
docker run -d -p 8080:80 --name my-nginx nginx:latest
```

What just happened:

* Docker pulled the image
* Created a container
* Exposed port 80 inside container to 8080 on host

Test it:

```bash
# Simple HTTP request to verify container is responding
curl http://localhost:8080
```

If this works, Docker networking is fine. For now.

---

## Understanding Dockerfile (Hands-on)

A Dockerfile is a simple text file that tells Docker how to build an image step by step, when Docker reads the Dockerfile, it follows the instructions from top to bottom and creates the same environment every time. A Dockerfile explains what to install, how to configure it, and how to run the app — automatically and consistently.

Now we build something ourselves.

### Simple Dockerfile Example

```dockerfile
# Use official lightweight Python image
# Acts as base OS and runtime
FROM python:3.11-slim

# Set working directory inside container
# All commands will run from here
WORKDIR /app

# Copy local files to container
# Keeps application code inside image
COPY . /app

# Install dependencies
# --no-cache-dir reduces image size
RUN pip install --no-cache-dir flask

# Expose application port
# Purely documentation, not firewall
EXPOSE 5000

# Default command when container starts
# Runs Flask app
CMD ["python", "app.py"]
```

Every line matters. If you don’t understand one, stop and read.

---

## Building and Running Custom Image

```bash
# Build Docker image
# -t assigns a readable name
docker build -t my-flask-app .
```

Run it:

```bash
# Run container from custom image
# Maps Flask port to host
docker run -d -p 5000:5000 --name flask-test my-flask-app
```

Check status:

```bash
# List running containers
# Confirms container health
docker ps
```

If it exits immediately, don’t guess. Check logs.

```bash
# View container logs
# First place to look when something breaks
docker logs flask-test
```

Logs are honest. People aren’t always.

---

## Data Persistence with Volumes

Containers die. Data should not.

```bash
# Run MySQL with volume
# -v maps host directory to container data path
docker run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=secret \
  -v mysql_data:/var/lib/mysql \
  mysql:8.0
```

Why volume matters:

* Container removed → data still exists
* Upgrade container without losing DB

Seen too many incidents caused by skipping this.

---

## Basic Docker Commands You’ll Use Daily

```bash
# Stop container gracefully
docker stop <container_name>

# Remove container
docker rm <container_name>

# Remove image
docker rmi <image_name>

# Inspect container details
docker inspect <container_name>
```

I use `inspect` more than I’d like to admit.

---

## Things Beginners Usually Break

Based on real incidents.

* No volume for database
* Exposing services without firewall rules
* Running everything as root
* No image version pinning

Docker doesn’t remove responsibility. It just moves it.

---

## Final Notes from the Coffee Mug

Docker is not about being fancy.

It’s about:

* Repeatability
* Predictability
* Easier recovery at 2 AM

If your setup is boring and well-documented, you’re doing it right.

Next step after this:

* Docker Compose
* Basic container security
* Resource limits

But that’s another cup of coffee.

---

