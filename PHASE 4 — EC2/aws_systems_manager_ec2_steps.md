# PHASE 4.1 — AWS SYSTEMS MANAGER

## Goal

AWS Systems Manager (SSM) is the operational control plane you should learn after understanding EC2.

The goal is:

> Manage EC2 machines without relying on direct SSH for every operational task.

Core tools:

```text
Systems Manager
 |
 +-- Session Manager
 |
 +-- Run Command
 |
 +-- Patch Manager
 |
 +-- Parameter Store
 |
 +-- Automation
 |
 +-- State Manager
```

Official documentation:

https://docs.aws.amazon.com/systems-manager/latest/userguide/

---

# 4.1.1 The Old Operational Model

A traditional model looks like:

```text
Engineer
   |
   v
Internet
   |
   v
SSH :22
   |
   v
Bastion Host
   |
   v
EC2
```

This can require:

```text
SSH keys
Bastion hosts
Inbound port 22
Firewall rules
Key rotation
Network access management
```

Systems Manager provides alternatives for many operational tasks.

---

# 4.1.2 The SSM Model

Conceptually:

```text
Engineer
    |
    v
AWS Console / CLI
    |
    v
IAM Permissions
    |
    v
Systems Manager
    |
    v
SSM Agent
    |
    v
EC2
```

Session Manager can provide interactive shell access without requiring inbound SSH ports, bastion hosts, or SSH keys. AWS documents IAM-based access control and session logging capabilities for Session Manager. citeturn0search6turn0search10

---

# 4.1.3 SSM Managed Node

For Systems Manager operations, the EC2 instance must be available as a managed node.

Learn the prerequisites:

```text
EC2
 |
 +-- SSM Agent
 |
 +-- IAM instance role
 |
 +-- Network connectivity to required AWS Systems Manager endpoints
 |
 v
Managed Node
```

Do not assume:

```text
EC2 created
=
SSM automatically works
```

Always verify the instance appears as a managed node.

---

# 4.1.4 IAM Role for SSM

Your EC2 instance needs an appropriate IAM role.

For standard EC2 Systems Manager onboarding, learn the AWS-managed policy:

```text
AmazonSSMManagedInstanceCore
```

The exact permissions and network setup should be reviewed in the current AWS documentation before production deployment.

Official setup documentation:

https://docs.aws.amazon.com/systems-manager/latest/userguide/setup-instance-permissions.html

---

# 4.1.5 Session Manager

Session Manager provides interactive access to managed nodes.

Architecture:

```text
Engineer
   |
   v
AWS Console
   |
   v
Session Manager
   |
   v
EC2
```

You can open a shell without opening inbound TCP 22 when the environment is correctly configured.

AWS documents browser-based and CLI sessions, IAM-controlled access, and optional session logging. citeturn0search6

---

# 4.1.6 Session Manager Lab

## Step 1

Launch an EC2 instance.

Use:

```text
IAM Role:
SSM-capable role
```

## Step 2

Make sure the instance has the required network path to Systems Manager endpoints.

Depending on architecture, this may use internet/NAT connectivity or VPC endpoints.

## Step 3

Open:

```text
AWS Console
 ->
Systems Manager
 ->
Session Manager
 ->
Node Management / Sessions
```

The exact console labels can change over time.

## Step 4

Start a session with the EC2 instance.

You should receive a shell.

Test:

```bash
whoami
hostname
pwd
```

Then:

```bash
systemctl status nginx
```

if Nginx is installed.

---

# 4.1.7 Why Session Manager Matters

Traditional:

```text
SSH
 |
 +-- Port 22
 +-- Key
 +-- Bastion
```

Session Manager:

```text
IAM
 |
 v
Session Manager
 |
 v
EC2
```

This centralizes access control through IAM.

You can grant an operations team permission to start sessions without distributing an SSH private key to every engineer.

---

# 4.1.8 Session Logging

Session Manager can be configured to log session activity.

Conceptually:

```text
Engineer
   |
   v
Session
   |
   +--------> CloudWatch Logs
   |
   +--------> S3
```

You can also use KMS encryption according to your security requirements.

For production:

```text
Session Access
     |
     v
IAM
     |
     v
Logging
     |
     v
Audit
```

Review the current AWS documentation for supported logging destinations and encryption configuration.

Official documentation:

https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-logging.html

---

# 4.1.9 Run Command

Run Command lets you execute commands remotely on managed nodes.

Examples:

```text
Install Nginx
Restart service
Collect logs
Run shell script
Update configuration
Check disk space
```

Conceptually:

```text
Engineer
   |
   v
Run Command
   |
   +-- EC2-A
   +-- EC2-B
   +-- EC2-C
   +-- EC2-D
```

AWS documents Run Command as a way to perform one-time or recurring administrative tasks across managed nodes without manually logging into each server. citeturn0search0turn0search5

---

# 4.1.10 Run Command Lab

Open:

```text
Systems Manager
 ->
Run Command
 ->
Run command
```

Choose an appropriate document such as:

```text
AWS-RunShellScript
```

for Linux.

Example:

```bash
hostname
```

Then:

```bash
df -h
```

Then:

```bash
systemctl status nginx
```

Run the command against:

```text
One EC2
```

Then repeat against:

```text
Multiple EC2 instances
```

using instance selection or tags.

---

# 4.1.11 Run Command with Tags

Suppose:

```text
Environment=production
Application=web
```

is attached to 20 instances.

You can target the appropriate nodes by tags rather than manually selecting instance IDs.

Conceptually:

```text
Run Command
     |
     v
Environment=production
     |
     +-- EC2-1
     +-- EC2-2
     +-- EC2-3
     ...
```

This is much more scalable than:

```text
SSH into EC2-1
SSH into EC2-2
SSH into EC2-3
...
```

AWS Run Command supports targeting managed nodes by tags/resource groups. citeturn0search3

---

# 4.1.12 Important Run Command Security Rule

Do not send secrets as plaintext command parameters.

For example, do not do:

```bash
echo "MY_DATABASE_PASSWORD"
```

AWS warns that command execution activity can be logged, so sensitive values should not be placed in plaintext commands. Use secure parameter mechanisms instead. citeturn0search0

---

# 4.1.13 Patch Manager

Patch Manager helps manage operating-system patching for managed nodes.

Conceptually:

```text
Managed Nodes
 |
 +-- EC2-A
 +-- EC2-B
 +-- EC2-C
 |
 v
Patch Manager
 |
 v
Patch Baseline
 |
 v
Patching operation
```

Learn:

```text
Patch Baseline
Patch Compliance
Maintenance Windows
Patch operations
```

Official documentation:

https://docs.aws.amazon.com/systems-manager/latest/userguide/patch-manager.html

---

# 4.1.14 Patch Manager Lab

Create several lab instances:

```text
Environment=lab
Role=web
```

Then learn to:

1. Inspect patch compliance.
2. Review missing patches.
3. Understand the patch baseline.
4. Run a patch operation.
5. Review the result.

For production, patching should be scheduled and tested rather than blindly applying updates to every server at once.

AWS provides Maintenance Windows and Run Command integrations for scheduled operational tasks. citeturn0search8

---

# 4.1.15 Parameter Store

Parameter Store is used for centralized configuration values.

Examples:

```text
/application/dev/log-level
/application/dev/api-url
/application/prod/api-url
```

Parameter types include:

```text
String
StringList
SecureString
```

AWS documents Parameter Store as a centralized store for configuration data and other parameters. citeturn0search11

---

# 4.1.16 Parameter Store vs Secrets Manager

Do not treat every secret as an ordinary parameter.

General mental model:

```text
Configuration
     |
     v
Parameter Store

Application secrets / credentials
     |
     v
Secrets Manager
```

Parameter Store supports `SecureString`, but AWS recommends Secrets Manager for secrets such as database credentials, API keys, and tokens because it provides purpose-built secret-management capabilities. citeturn0search11

---

# 4.1.17 Parameter Store Lab

Create:

```text
Name:
/company/lab/app/log-level

Type:
String

Value:
INFO
```

Then create:

```text
/company/lab/app/api-url
```

Retrieve them through the Systems Manager console.

Learn the hierarchy:

```text
/company/
    lab/
        app/
            log-level
            api-url
```

This naming structure becomes useful for IAM policies and application configuration.

---

# 4.1.18 SecureString Lab

Create a test `SecureString` parameter.

Conceptually:

```text
/company/lab/test-value
```

Type:

```text
SecureString
```

Then understand:

```text
Application
    |
    v
IAM Role
    |
    v
ssm:GetParameter
    |
    v
Parameter Store
```

Do not use real production credentials in a learning lab.

---

# 4.1.19 IAM for Parameter Store

The application should receive access through its IAM role.

Example conceptual policy:

```text
EC2 Role
 |
 +-- ssm:GetParameter
       |
       v
/company/lab/app/*
```

Avoid:

```text
ssm:GetParameter
Resource: *
```

when you can restrict access to specific parameter paths/resources.

---

# 4.1.20 Automation

Systems Manager Automation uses runbooks to automate operational tasks.

Conceptually:

```text
Automation
    |
    v
Runbook
    |
    +-- Step 1
    +-- Step 2
    +-- Step 3
    |
    v
Result
```

Examples:

```text
Restart EC2
Create AMI
Update infrastructure
Remediate configuration
Run operational workflow
```

AWS provides predefined Automation runbooks and supports custom automation documents. citeturn0search1

---

# 4.1.21 Automation Lab

Open:

```text
Systems Manager
 ->
Automation
 ->
Execute automation
```

Select an appropriate AWS runbook.

For learning, use a harmless operation such as restarting a lab EC2 instance.

Understand:

```text
Runbook
 |
 v
Parameters
 |
 v
Steps
 |
 v
Execution
 |
 v
Result
```

AWS's current console workflow supports executing runbooks step-by-step, which is useful for understanding exactly what each automation step does. citeturn0search1

---

# 4.1.22 State Manager

State Manager helps keep managed nodes in a defined configuration.

Example:

```text
Desired State:

Nginx installed
Nginx running
Configuration present
```

State Manager helps maintain the desired state over time.

This is different from a one-time command:

```text
Run Command
= Do this now
```

while:

```text
State Manager
= Keep this configuration/state
```

AWS describes State Manager as a tool for keeping managed nodes in a defined state. citeturn0search10

---

# 4.1.23 SSM Operational Architecture

A mature EC2 environment can look like:

```text
                    Engineer
                       |
                       v
                      IAM
                       |
             +---------+---------+
             |         |         |
             v         v         v
          Session   Run Cmd   Automation
          Manager
             |         |         |
             +---------+---------+
                       |
                       v
                  EC2 Fleet
                 /    |    \
                /     |     \
              EC2    EC2    EC2
                |
        +-------+-------+
        |               |
        v               v
 Parameter Store    CloudWatch
```

---

# 4.1.24 Complete SSM Lab

Build three EC2 instances:

```text
web-1
web-2
web-3
```

Tags:

```text
Environment=lab
Application=web
```

Then perform:

## Exercise A — Session Manager

Connect to:

```text
web-1
```

without SSH.

## Exercise B — Run Command

Run:

```bash
hostname
```

against all three.

## Exercise C — Install Nginx

Use Run Command to install Nginx on all three.

## Exercise D — Verify

Run:

```bash
systemctl status nginx
```

against all three.

## Exercise E — Parameter Store

Create:

```text
/company/lab/web/message
```

with:

```text
Hello from Parameter Store
```

## Exercise F — Application Configuration

Retrieve the parameter from the instance using the appropriate IAM role and AWS tooling.

## Exercise G — Automation

Run a safe Automation runbook against one test instance.

## Exercise H — Patching

Inspect patch compliance and understand the patch workflow.

---

# 4.1.25 Troubleshooting SSM

If an EC2 instance does not appear as a managed node:

```text
EC2
 |
 +-- Is SSM Agent installed/running?
 |
 +-- Does EC2 have the correct IAM role?
 |
 +-- Does IAM allow required SSM actions?
 |
 +-- Does the instance have network connectivity
 |   to Systems Manager endpoints?
 |
 +-- Is the instance in the expected Region/account?
 |
 +-- Are there endpoint/DNS/security issues?
```

Do not immediately reinstall the agent.

Find which layer is broken.

---

# 4.1.26 Security Model

Your SSM security model should look like:

```text
Human
 |
 v
IAM / Identity Center
 |
 v
Systems Manager permissions
 |
 v
Managed Node
```

The EC2 instance itself uses:

```text
EC2 IAM Role
 |
 v
Systems Manager
```

This gives you two separate permission questions:

```text
Who can manage the EC2?
        |
        v
Human IAM permissions

What can the EC2 do?
        |
        v
EC2 IAM role
```

This is the same IAM mental model from Phase 1.

---

# 4.1.27 Completion Checklist

You should be able to explain:

- Systems Manager.
- Managed node.
- SSM Agent.
- EC2 IAM role for SSM.
- Session Manager.
- Run Command.
- Patch Manager.
- Patch baseline.
- Maintenance Window.
- Parameter Store.
- String.
- StringList.
- SecureString.
- Parameter Store vs Secrets Manager.
- Automation.
- Runbooks.
- State Manager.
- SSM troubleshooting.
- IAM control of SSM access.

You should be able to perform:

```text
EC2
 |
 v
SSM Managed Node
 |
 +-- Session Manager
 |
 +-- Run Command
 |
 +-- Patch Manager
 |
 +-- Parameter Store
 |
 +-- Automation
```

---

# Final Mental Model

```text
                    IAM
                     |
                     v
              Systems Manager
                     |
       +-------------+-------------+
       |             |             |
       v             v             v
   Session        Run Cmd       Automation
   Manager
       |             |             |
       +-------------+-------------+
                     |
                     v
                 EC2 Fleet
                     |
             +-------+-------+
             |               |
             v               v
      Parameter Store     CloudWatch
```

The key operational principle is:

> Don't think of EC2 as a machine you manually SSH into. Think of EC2 as a managed fleet that can be accessed, configured, patched, monitored, and automated through controlled AWS services.

Official documentation:

- Systems Manager: https://docs.aws.amazon.com/systems-manager/latest/userguide/
- Session Manager: https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html
- Run Command: https://docs.aws.amazon.com/systems-manager/latest/userguide/run-command.html
- Patch Manager: https://docs.aws.amazon.com/systems-manager/latest/userguide/patch-manager.html
- Parameter Store: https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html
- Automation: https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-automation.html
- State Manager: https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-state.html
