# PHASE 4 — AMAZON EC2

## Goal

Amazon EC2 is more than:

> Create a virtual machine.

You need to understand the complete lifecycle:

```text
AMI
 |
 v
Launch Template
 |
 v
EC2 Instance
 |
 +-- Instance Type
 +-- EBS
 +-- Key Pair
 +-- IAM Role
 +-- User Data
 +-- Security Group
 |
 v
Application
 |
 v
Load Balancer
 |
 v
Auto Scaling
 |
 v
CloudWatch / Systems Manager
```

The objective is to become capable of launching, securing, monitoring, scaling, and troubleshooting EC2 workloads.

Official EC2 documentation:

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/

---

# 4.1 AMI

AMI means:

> Amazon Machine Image.

An AMI is a template used to launch EC2 instances.

Conceptually:

```text
AMI
 |
 +-- Operating System
 +-- Software/configuration
 +-- Storage configuration
 |
 v
EC2 Instance
```

Examples include Amazon Linux, Ubuntu, Windows Server, and custom application images.

A custom AMI can be created from a configured instance and reused to launch instances with the same baseline.

Think:

```text
AMI = machine blueprint
EC2 = running machine
```

Official documentation:

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html

---

# 4.2 EC2 Instance

An EC2 instance is a running virtual server.

It has:

```text
CPU
Memory
Network
Storage
Operating System
IAM Role
Security Groups
Private IP
```

The instance lives inside your VPC subnet.

Typical architecture:

```text
VPC
 |
 v
Subnet
 |
 v
EC2
 |
 v
Application
```

---

# 4.3 Instance Type

Instance type determines the hardware profile available to the instance.

Think:

```text
Instance Type
 |
 +-- vCPU
 +-- Memory
 +-- Network performance
 +-- Storage capabilities
 +-- Architecture
```

Examples belong to different families designed for different workloads.

Conceptually:

```text
General purpose -> balanced
Compute optimized -> CPU-heavy
Memory optimized -> memory-heavy
Storage optimized -> storage-heavy
Accelerated computing -> GPU/accelerators
```

Do not choose an instance only because it has more CPU.

Consider:

```text
CPU
RAM
Network
EBS bandwidth
Architecture
Workload
Cost
```

Official documentation:

https://docs.aws.amazon.com/ec2/latest/instancetypes/ec2-instance-types.html

---

# 4.4 EBS

Amazon EBS is persistent block storage for EC2.

Think:

```text
EC2
 |
 v
EBS Volume
 |
 v
Filesystem
```

Unlike the ephemeral compute instance itself, EBS volumes are designed to persist independently of the instance lifecycle depending on their configuration.

Common use:

```text
OS disk
Application disk
Database/data disk
```

Official documentation:

https://docs.aws.amazon.com/ebs/latest/userguide/what-is-ebs.html

---

# 4.5 EBS Volume

A volume is a block-storage device.

Example:

```text
EC2
 |
 +-- /dev/root
 |
 +-- /dev/data
```

The volume type affects performance characteristics and cost.

Important concepts:

```text
Size
IOPS
Throughput
Encryption
Snapshots
Availability Zone
```

An EBS volume is associated with an Availability Zone and is normally attached to EC2 instances in the same AZ.

---

# 4.6 EBS Snapshots

An EBS snapshot is a point-in-time backup of an EBS volume.

Conceptually:

```text
EBS Volume
    |
    v
Snapshot
    |
    v
New EBS Volume
```

Snapshots are useful for:

- Backup
- Recovery
- Creating new volumes
- Building AMI-related workflows

Do not confuse:

```text
Snapshot = point-in-time backup
Volume   = active block storage
```

Official documentation:

https://docs.aws.amazon.com/ebs/latest/userguide/ebs-snapshots.html

---

# 4.7 Elastic IP

An Elastic IP is a static public IPv4 address allocated to your AWS account.

Use it when you specifically need a persistent public IPv4 address.

Do not assume every EC2 instance needs one.

Modern architectures often put EC2 behind a load balancer and keep application instances private.

---

# 4.8 Key Pair

EC2 key pairs are commonly used for SSH access to Linux instances.

Conceptually:

```text
Private Key
   |
   | held by you
   v
SSH Client
   |
   v
EC2
```

The private key must be protected.

Do not commit private keys to Git.

Do not send private keys through Slack/email.

Do not put private keys inside application source code.

For many operational tasks, Systems Manager Session Manager can reduce the need for direct SSH access.

---

# 4.9 IAM Role for EC2

An EC2 instance should use an IAM role when the application running on it needs AWS API access.

Example:

```text
EC2
 |
 v
IAM Role
 |
 +-- S3 permissions
 +-- CloudWatch permissions
 +-- SSM permissions
```

The application receives temporary credentials through the instance's role rather than requiring hard-coded access keys.

This connects directly to Phase 1 IAM.

Mental model:

```text
Human
  -> IAM Identity Center

EC2 Application
  -> IAM Role
```

Avoid putting long-lived AWS access keys in EC2 user-data, source code, or config files.

---

# 4.10 User Data

User data is bootstrap configuration supplied when launching an EC2 instance.

Example:

```bash
#!/bin/bash
dnf update -y
dnf install -y nginx
systemctl enable nginx
systemctl start nginx
```

Conceptually:

```text
EC2 starts
   |
   v
User Data
   |
   v
Install/configure software
   |
   v
Application ready
```

User data is useful for:

- Installing packages
- Creating configuration
- Starting services
- Bootstrapping application dependencies

Do not put long-lived secrets in user data.

---

# 4.11 Security Groups

EC2 uses security groups for network access control.

Example:

```text
Inbound
SSH    22   YOUR_IP/32
HTTP   80   0.0.0.0/0
HTTPS  443  0.0.0.0/0
```

For private application servers:

```text
HTTP 8080
Source:
ALB-SG
```

This is preferable to opening the application port to the entire internet.

See Phase 3.2 for the deeper Security Group vs NACL model.

---

# 4.12 Placement and Availability Zones

EC2 instances run in an Availability Zone.

For normal application architecture:

```text
Region
 |
 +-- AZ-A
 |    |
 |    +-- EC2
 |
 +-- AZ-B
      |
      +-- EC2
```

Spreading application capacity across multiple AZs improves resilience to an AZ-level disruption.

EC2 placement groups are a separate advanced concept.

Know the major placement strategies:

```text
Cluster
Spread
Partition
```

Use them for specialized workloads; do not assume a placement group is required for ordinary web applications.

Official documentation:

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html

---

# 4.13 CloudWatch

CloudWatch is used for monitoring EC2 and the surrounding application.

Useful concepts:

```text
Metrics
Logs
Alarms
Dashboards
Events / automation integrations
```

Example:

```text
EC2
 |
 v
CloudWatch
 |
 +-- CPUUtilization
 +-- Status checks
 +-- Logs
 +-- Alarms
```

For detailed operating-system metrics, additional configuration or an agent may be required.

Official documentation:

https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html

---

# 4.14 Launch Template

A Launch Template is a reusable EC2 launch configuration.

It can define:

```text
AMI
Instance Type
Key Pair
Security Groups
EBS
IAM Role
User Data
Tags
Network settings
```

Think:

```text
Launch Template
       |
       +--> EC2-1
       +--> EC2-2
       +--> EC2-3
```

Launch templates are especially important for Auto Scaling.

AWS recommends launch templates for Auto Scaling groups. Official documentation:

https://docs.aws.amazon.com/autoscaling/ec2/userguide/create-launch-template.html

---

# 4.15 Auto Scaling

EC2 Auto Scaling manages a group of EC2 instances.

You specify:

```text
Minimum
Desired
Maximum
```

Example:

```text
Min = 2
Desired = 2
Max = 6
```

The Auto Scaling group attempts to maintain the desired capacity and can scale according to configured policies.

It can also replace unhealthy instances.

Official documentation:

https://docs.aws.amazon.com/autoscaling/ec2/userguide/

---

# 4.16 Target Group

A target group contains the destinations that a load balancer sends traffic to.

Example:

```text
Target Group
 |
 +-- EC2-A
 +-- EC2-B
```

The target group also contains health-check configuration.

Conceptually:

```text
ALB
 |
 v
Target Group
 |
 +-- Healthy EC2
 +-- Healthy EC2
 +-- Unhealthy EC2
```

The load balancer should not send normal application traffic to unhealthy targets.

---

# 4.17 Application Load Balancer

An Application Load Balancer (ALB) operates at the application layer and is commonly used for HTTP/HTTPS applications.

Typical architecture:

```text
Internet
   |
   v
ALB
   |
   v
Target Group
   |
   +-- EC2
   +-- EC2
```

A common secure architecture is:

```text
Internet
   |
   v
Public ALB
   |
   v
Private EC2
```

The ALB's security group allows internet/application traffic.

The EC2 security group allows traffic only from the ALB security group.

Official documentation:

https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html

---

# 4.18 Health Checks

Health checks answer:

> Can this target actually serve application traffic?

Example:

```text
GET /health
```

Expected:

```text
HTTP 200
```

If the application stops responding correctly:

```text
ALB
 |
 v
Target = unhealthy
```

When ALB health checks are enabled for an Auto Scaling group, EC2 Auto Scaling can use those health results to replace unhealthy instances.

Official documentation:

https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-health-checks.html

---

# 4.19 First Project — Public EC2 + Nginx

Build:

```text
Internet
    |
    v
Internet Gateway
    |
    v
Public Subnet
    |
    v
EC2
    |
    v
Nginx
    |
    v
HTTP Response
```

## Step 1

Use the VPC created in Phase 3.1.

## Step 2

Launch EC2 in:

```text
Public-A
```

## Step 3

Choose:

```text
AMI:
Amazon Linux or another supported Linux AMI

Instance Type:
small learning-appropriate type

Key Pair:
create/use your lab key

Security Group:
HTTP 80 -> 0.0.0.0/0
SSH 22 -> YOUR_IP/32
```

## Step 4

Use user data to install Nginx.

## Step 5

Verify:

```text
http://PUBLIC_IP
```

You should see the Nginx response.

## Step 6

Trace the packet:

```text
Browser
 |
 v
Internet
 |
 v
IGW
 |
 v
Public Route Table
 |
 v
EC2 Security Group
 |
 v
EC2
 |
 v
Nginx
```

---

# 4.20 Second Project — ALB -> Private EC2

Now remove direct internet exposure from the application instance.

Architecture:

```text
Internet
    |
    v
Internet Gateway
    |
    v
Public Subnets
    |
    v
ALB
    |
    v
Target Group
    |
    v
Private Subnet
    |
    v
EC2
    |
    v
Nginx
```

Security groups:

```text
ALB-SG
Inbound:
80 from Internet

EC2-SG
Inbound:
80 from ALB-SG
```

Do not expose:

```text
EC2-SG
80
0.0.0.0/0
```

The ALB becomes the public entry point.

---

# 4.21 Third Project — ALB + Auto Scaling

Final architecture:

```text
                    Internet
                       |
                       v
                 Application
                 Load Balancer
                       |
                 Target Group
                  /         \
                 v           v
              EC2-A        EC2-B
             Private-A    Private-B
```

Create:

```text
Launch Template
```

with:

```text
AMI
Instance Type
IAM Role
Security Group
User Data
EBS
Tags
```

Then create:

```text
Auto Scaling Group
```

Example:

```text
Min     = 2
Desired = 2
Max     = 4
```

Use:

```text
Private-App-A
Private-App-B
```

The Auto Scaling group can launch instances in multiple AZs.

AWS documentation specifically recommends using multiple AZs for high availability in these workflows. citeturn1search1turn1search7

---

# 4.22 Connect ALB to Auto Scaling

The flow is:

```text
Launch Template
       |
       v
Auto Scaling Group
       |
       v
EC2 Instances
       |
       v
Target Group
       |
       v
ALB
```

When instances are launched by the Auto Scaling group, they can be automatically registered with the attached target group; terminated instances are deregistered. citeturn1search0turn1search9

---

# 4.23 Scaling Test

Start:

```text
Min = 2
Desired = 2
Max = 4
```

Then configure a scaling policy.

For a learning exercise, use a simple target-tracking policy.

Example concept:

```text
CPU target = 50%
```

If demand increases:

```text
2 EC2
  |
  v
CPU rises
  |
  v
Auto Scaling
  |
  v
3 EC2
```

If demand decreases:

```text
3 EC2
  |
  v
CPU falls
  |
  v
Auto Scaling
  |
  v
2 EC2
```

Auto Scaling supports scaling policies and maintains configured capacity boundaries. citeturn1search5

---

# 4.24 Health Check Failure Test

This is an important operational exercise.

Start with:

```text
EC2-A = healthy
EC2-B = healthy
```

Then intentionally make the application on one instance fail the ALB health check.

Example:

```text
Stop Nginx
```

Expected:

```text
ALB
 |
 v
Target EC2-A
 |
 v
UNHEALTHY
```

If ELB health checks are enabled for the Auto Scaling group:

```text
ASG
 |
 v
Detect unhealthy instance
 |
 v
Terminate/replace instance
 |
 v
New EC2
 |
 v
Nginx
 |
 v
Healthy target
```

AWS documents that Auto Scaling can replace an instance when ELB reports it unhealthy after ELB health checks are enabled for the group. citeturn1search6

---

# 4.25 EC2 Troubleshooting Framework

When EC2 is unreachable, check in this order:

```text
1. Instance state
2. VPC
3. Subnet
4. Route table
5. Public/private addressing
6. Security Group
7. NACL
8. OS firewall
9. Application process
10. Application port
11. Load balancer
12. Target health
```

Example:

```text
ALB returns 503
```

Do not immediately restart EC2.

Check:

```text
ALB
 |
 v
Listener
 |
 v
Target Group
 |
 v
Target health
 |
 v
EC2 SG
 |
 v
EC2 port
 |
 v
Nginx/application
```

---

# 4.26 Completion Checklist

You should be able to explain:

- AMI
- EC2 instance
- Instance type
- EBS
- EBS volume
- Snapshot
- Elastic IP
- Key pair
- IAM role
- User data
- Security group
- Availability Zones
- Placement groups
- CloudWatch
- Launch template
- Auto Scaling group
- Target group
- ALB
- Health checks
- Scaling policy

You should also be able to build:

```text
VPC
 |
 +-- Public Subnet
 |     |
 |     +-- ALB
 |
 +-- Private App Subnet
       |
       +-- EC2
       +-- EC2
```

And explain every connection.

---

# Final Mental Model

```text
             AMI
              |
              v
       Launch Template
              |
              v
       Auto Scaling Group
          /         \
         v           v
       EC2-A       EC2-B
         \           /
          \         /
           Target Group
                |
                v
               ALB
                |
                v
             Internet
```

Monitoring and management:

```text
EC2
 |
 +-- CloudWatch
 |
 +-- Systems Manager
 |
 +-- IAM Role
 |
 +-- EBS
```

The key question is:

> How can I launch the same server configuration repeatedly, keep the application available, replace unhealthy instances, scale capacity, and operate the machines securely?
