---
title: Learn Terraform Basic
description: A practical guide to learning Terraform basics, written from real operations experience. Covers core concepts, step by step workflow, AWS examples, environment separation, and remote state best practices without fluff or hype.
slug: learn-terraform-basic
date: 2025-12-11 00:00:00+0000
#image : cover.webp
categories : Devops
tags:
    - Devops
    - System Administrator
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

# Learning Terraform Basics

I write this from daily ops experience.
Terraform is a tool for people who hate guessing.
It turns infrastructure into files you can read, review, and track.
Logs matter, and state files matter even more.

Terraform manages infrastructure using code.
The code is plain text.
You can store it in Git.
You can review changes before touching production.
That alone already saves sleep.

Think of Terraform like a wiring diagram.
You do not power anything yet.
You only describe how cables should connect.
Terraform reads the diagram.
Then it builds exactly that.

## What Problem Terraform Solves

Manual setup works.
It works until it does not.

Common issues appear fast.

* Servers differ between environments
* Changes are forgotten
* Rollback depends on memory
* Documentation drifts from reality

Terraform fixes this by force.
It only builds what the code says.
No more hidden clicks.
No more silent changes at midnight.

## Core Concepts You Must Understand

Terraform looks simple at first.
The basics decide everything later.

### Provider

A provider is a bridge.
It connects Terraform to a platform.
AWS, Azure, GCP, and many others use providers.

If you manage AWS EC2.
You use the AWS provider.

### Resource

A resource is one object.
A server.
A network.
A firewall rule.

Each resource has a type and a name.
Terraform tracks each resource in its state.

### State

State is the source of truth.
Not the cloud console.
Not your memory.

The state file records real infrastructure.
Lose it and you lose control.
Protect it like production data.

### Plan

Plan shows what will change.
Nothing executes yet.

This step prevents surprises.
Always read the plan.

### Apply

Apply makes changes real.
Terraform compares code and state.
Then it acts.

## Installing Terraform

Use the official binary.
Avoid random packages.

Example for Linux.

```bash
# Download Terraform binary
# This gets the exact version we want
wget https://releases.hashicorp.com/terraform/1.6.6/terraform_1.6.6_linux_amd64.zip

# Unzip the binary
# This extracts the terraform executable
unzip terraform_1.6.6_linux_amd64.zip

# Move binary to PATH
# This allows terraform command anywhere
sudo mv terraform /usr/local/bin/

# Verify installation
# This confirms terraform is ready
terraform version
```

If this fails.
Fix it now.
Do not continue.

## First Terraform Project Structure

Keep structure boring.
Boring is stable.

Example layout.

* main.tf
* variables.tf
* outputs.tf
* terraform.tfstate

Do not rename files randomly.
Terraform loads all .tf files together.

## Writing Your First Terraform Code

We start small.
One local file.
No cloud yet.

This removes noise.

### main.tf

```hcl
# Configure Terraform settings
# Required version avoids unexpected behavior
terraform {
  required_version = ">= 1.6.0"
}

# Local file resource
# This creates a file on the local system
resource "local_file" "example" {
  # File content
  # This text will be written into the file
  content  = "Terraform was here"

  # File name
  # Path where the file will be created
  filename = "./hello.txt"
}
```

This code does one thing.
It creates a text file.

That is enough to learn the workflow.

## Terraform Workflow Step by Step

This flow never changes.
Learn it once.

### Step 1. Initialize

```bash
# Initialize Terraform project
# This downloads required providers
terraform init
```

Init prepares the working directory.
Run it once per project.
Run it again if providers change.

### Step 2. Validate

```bash
# Validate configuration files
# This checks syntax and basic errors
terraform validate
```

This catches mistakes early.
Use it before plan.

### Step 3. Plan

```bash
# Show execution plan
# This previews changes without applying
terraform plan
```

Read the output.
Every line matters.
If something looks wrong.
Stop here.

### Step 4. Apply

```bash
# Apply changes
# This creates or updates resources
terraform apply
```

Terraform asks for confirmation.
Say yes only when ready.

After apply.
Check the file system.
hello.txt should exist.

## Understanding State Behavior

Terraform stores state locally by default.
This is fine for learning.
It is risky for teams.

State maps code to real objects.
Delete state and Terraform forgets everything.

For real projects.
Use remote state.
S3, GCS, or similar backends.

## Variables for Reusable Code

Hardcoded values do not scale.
Variables fix that.

### variables.tf

```hcl
# Define a variable
# This allows flexible input
variable "file_content" {
  # Variable type
  type = string

  # Default value
  default = "Terraform was here"
}
```

Update main.tf.

```hcl
# Use variable inside resource
resource "local_file" "example" {
  # Content comes from variable
  content  = var.file_content

  # File path stays the same
  filename = "./hello.txt"
}
```

Now the code adapts.
No rewrite needed.

## Outputs for Visibility

Outputs show useful data.
They help other tools.

### outputs.tf

```hcl
# Output file path
# This displays after apply
output "file_path" {
  value = local_file.example.filename
}
```

Run apply again.
Terraform prints the output.

## Simple Analogy That Actually Works

Terraform is not magic.
It is a strict technician.

You give a checklist.
It follows the checklist exactly.

If the checklist is wrong.
Terraform is not polite.
It still executes it.

That is why review matters.

## Common Beginner Mistakes

These appear every week.

* Skipping plan
* Editing resources manually
* Ignoring state files
* Mixing environments in one project

Terraform rewards discipline.
It punishes shortcuts.

## When Terraform Is a Bad Idea

Terraform is not for everything.

Avoid it for.

* One time experiments
* Manual emergency fixes
* Systems without stable APIs

Use the right tool.
Sleep matters.

## Final Notes

Terraform basics are simple.
The impact is not.

Treat code like production.
Review changes.
Protect state.
Write boring configs.

If it feels boring.
You are doing it right.

## Example Using AWS

This example uses AWS EC2.
It stays minimal on purpose.

You need an AWS account.
You need an IAM user.
Use access key and secret key.

### Provider Configuration

```hcl
# AWS provider configuration
# This tells Terraform how to talk to AWS
provider "aws" {
  # AWS region
  # Choose one and stay consistent
  region = "ap-southeast-1"
}
```

### EC2 Instance Resource

```hcl
# EC2 instance resource
# This creates a single virtual server
resource "aws_instance" "lab_server" {
  # Amazon Linux AMI
  # AMI ID is region specific
  ami = "ami-0df7a207adb9748c7"

  # Instance size
  # Small enough for lab usage
  instance_type = "t3.micro"

  # Tags help tracking and cleanup
  tags = {
    Name = "terraform-lab-server"
  }
}
```

Run init, plan, and apply.
Check AWS console after apply.
The instance should exist.

## Separating Lab and Production

Never mix environments.
It ends badly.

Use separate directories.

* envs/lab
* envs/production

Each environment has its own state.
Each environment has its own variables.

Example structure.

* modules/ec2
* envs/lab/main.tf
* envs/production/main.tf

### Lab Environment Example

```hcl
# Lab environment EC2
# Uses smaller instance and cheaper setup
module "ec2" {
  source = "../../modules/ec2"

  instance_type = "t3.micro"
  name          = "lab-server"
}
```

### Production Environment Example

```hcl
# Production environment EC2
# Uses more stable instance size
module "ec2" {
  source = "../../modules/ec2"

  instance_type = "t3.medium"
  name          = "production-server"
}
```

Same code.
Different behavior.
No risk of cross damage.

## Remote State and Locking Best Practice

Local state does not scale.
Teams overwrite each other.

Use remote backend.
Use locking.

AWS S3 with DynamoDB works well.

### Remote State Configuration

```hcl
# Remote backend configuration
# Stores state in S3 and locks via DynamoDB
terraform {
  backend "s3" {
    # S3 bucket for state file
    bucket = "my-terraform-state-bucket"

    # Path inside the bucket
    key    = "lab/terraform.tfstate"

    # AWS region
    region = "ap-southeast-1"

    # DynamoDB table for locking
    dynamodb_table = "terraform-locks"

    # Encrypt state at rest
    encrypt = true
  }
}
```

### Why This Matters

State contains sensitive data.
IPs, IDs, and metadata.

Locking prevents parallel apply.
Parallel apply breaks infrastructure.

Create the S3 bucket first.
Create DynamoDB table first.
Terraform will not do that for you.

## Operational Notes From the Field

Use one state per environment.
Never share state files.

Protect backend credentials.
Rotate keys regularly.

Review plan output as a team.
One missed line can cost hours.

Terraform works best with discipline.
It does not forgive carelessness.
