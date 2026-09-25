# AWS Account Management & Cloud Mastery Roadmap

## Goal

> "I can manage, secure, operate, troubleshoot, optimize, and automate an AWS environment for a company."

The roadmap is intentionally **Console-first**, because the AWS Console helps build the mental model of the services. After becoming comfortable with the Console, the same operations should be learned through AWS CLI and Infrastructure as Code such as Terraform.

---

# 1. The Big Picture

Do not learn AWS as a random list of services.

Learn it as a connected system:

```text
Account
  ↓
Organization / Governance
  ↓
Identity & Permissions
  ↓
Networking
  ↓
Security
  ↓
Compute
  ↓
Storage
  ↓
Database
  ↓
Containers
  ↓
CDN / DNS / Load Balancing
  ↓
Monitoring / Auditing
  ↓
Cost Management
  ↓
Backup / Disaster Recovery
  ↓
Automation / Infrastructure as Code
  ↓
AI / ML / SageMaker
```

A typical company architecture may look like:

```text
                    AWS Organization
                           │
             ┌─────────────┴─────────────┐
             │                           │
       Management Account          Security / Log Account
             │
       ┌─────┼──────────────┐
       │     │              │
      Dev   Staging      Production
       │     │              │
      EC2   EC2            EC2
      RDS   RDS            RDS
      S3    S3             S3
```

Inside an account:

```text
IAM / Identity
      ↓
VPC / Networking
      ↓
Security Groups / NACL
      ↓
EC2 / ECR / RDS / S3
      ↓
CloudFront / Route53 / ALB
      ↓
CloudWatch / CloudTrail
      ↓
Cost Explorer / Budgets
```

---

# 2. Complete Learning Roadmap

| Phase | Area | Main Skills |
|---|---|---|
| 0 | AWS Account Fundamentals | Root, billing, regions, tags, quotas |
| 1 | IAM | Users, groups, roles, policies, STS |
| 2 | IAM Identity Center | Users, groups, permission sets, MFA |
| 3 | Organizations | Accounts, OUs, SCPs, consolidated billing |
| 4 | Networking | VPC, subnets, routes, IGW, NAT, SG, NACL |
| 5 | EC2 | Instances, AMIs, EBS, SSM, Auto Scaling |
| 6 | S3 | Storage, policies, encryption, versioning, lifecycle |
| 7 | RDS | Databases, subnet groups, backups, Multi-AZ |
| 8 | Containers | Docker, ECR, ECS, Fargate |
| 9 | DNS/CDN | Route53, CloudFront, ACM, WAF |
| 10 | Security | CloudTrail, GuardDuty, Security Hub, Config |
| 11 | Secrets/Encryption | KMS, Secrets Manager, Parameter Store |
| 12 | Monitoring | CloudWatch, logs, metrics, alarms |
| 13 | Cost | Cost Explorer, Budgets, tags, pricing |
| 14 | Backup/DR | RPO, RTO, snapshots, AWS Backup |
| 15 | Automation | AWS CLI, Terraform, CI/CD |
| 16 | AI/ML | SageMaker, training, endpoints, model operations |
| 17 | Architecture | Well-Architected Framework, security architecture |

---

# PHASE 0 — AWS ACCOUNT MANAGEMENT

## Learn

Start with account-level concepts:

```text
AWS Account
Root User
IAM
IAM Identity Center
AWS Organizations
Regions
Availability Zones
Service Quotas
Billing
Support
Resource Groups
Tags
```

## Console Areas

Become comfortable navigating:

```text
AWS Console
├── IAM
├── IAM Identity Center
├── Organizations
├── Billing & Cost Management
├── EC2
├── VPC
├── S3
├── RDS
├── ECR
├── CloudFront
├── Route 53
├── CloudWatch
├── CloudTrail
├── Security Hub
├── GuardDuty
├── Config
└── SageMaker
```

## Exercise

Document your AWS account:

```text
Account ID
Primary Region
Enabled Regions
Billing configuration
Root MFA
IAM configuration
CloudTrail
Budgets
Default VPCs
S3 buckets
EC2 instances
RDS databases
```

Create a personal runbook:

```text
AWS-ACCOUNT-RUNBOOK.md
```

This should become your AWS administration handbook.

---

# PHASE 1 — IAM

IAM is one of the most important AWS topics.

Do not stop at:

> "IAM creates users."

You need to understand the AWS authorization model.

## Learn

```text
Principal
User
Group
Role
Policy
Permission
Trust Policy
Identity-based Policy
Resource-based Policy
Permission Boundary
SCP
Session Policy
STS
Access Analyzer
```

Mental model:

```text
                    IAM
                     │
       ┌─────────────┼──────────────┐
       │             │              │
     Users         Groups          Roles
       │             │              │
       └─────────────┼──────────────┘
                     ↓
                  Policies
                     ↓
              AWS Resources
```

For company environments, human users should generally be managed through centralized/federated access such as IAM Identity Center rather than creating long-lived IAM users and access keys for everyone.

## Company Simulation

Create roles/groups conceptually such as:

```text
AWS-Administrators
Developers
DevOps
Database-Admins
Security-Team
ReadOnly
Billing-Team
```

### Developer

Can access:

```text
EC2
ECR
CloudWatch
S3 application bucket
```

Cannot administer:

```text
Billing
IAM
Organizations
Production database deletion
```

### Database Administrator

Can access:

```text
RDS
CloudWatch
Secrets Manager
```

Cannot administer:

```text
Organizations
IAM
Billing
```

### Security Team

Can inspect:

```text
CloudTrail
GuardDuty
Security Hub
IAM
Config
```

---

# PHASE 1.1 — IAM POLICY MASTERY

Example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::company-data/*"
    }
  ]
}
```

Understand:

```text
Effect
Action
Resource
Principal
Condition
```

Then learn conditions:

```text
aws:PrincipalArn
aws:PrincipalOrgId
aws:SourceIp
aws:RequestedRegion
aws:MultiFactorAuthPresent
aws:ResourceTag
```

Understand policy evaluation:

```text
Explicit Deny
      ↓
always wins
      ↓
Allow
      ↓
otherwise Deny
```

You should eventually be able to look at a policy and answer:

> Who can do what, to which resource, under which conditions?

---

# PHASE 1.2 — IAM IDENTITY CENTER

Learn:

```text
IAM Identity Center
       ↓
Users
Groups
       ↓
Permission Sets
       ↓
AWS Accounts
```

Example:

```text
Company Employee
       ↓
IAM Identity Center
       ↓
Developer Group
       ↓
Developer Permission Set
       ↓
Development Account
```

Learn:

- Users
- Groups
- Permission Sets
- Account Assignments
- MFA
- Federation
- External Identity Provider concepts

---

# PHASE 2 — AWS ORGANIZATIONS

Once IAM is comfortable, learn Organizations.

## Understand

```text
Organization
Account
OU
SCP
Consolidated Billing
Cross-account access
IAM Identity Center
```

Example:

```text
Organization
│
├── Management Account
│
├── Security OU
│   ├── Security Account
│   └── Log Archive
│
├── Infrastructure OU
│   └── Network Account
│
└── Workloads OU
    ├── Development
    ├── Staging
    └── Production
```

## SCP

Understand the difference:

### IAM Policy

```text
What is this identity allowed to do?
```

### SCP

```text
What is this account allowed to do at maximum?
```

Important:

> SCPs do not grant permissions. They act as organization-level guardrails.

---

# PHASE 3 — NETWORKING

Networking is one of the most important AWS areas.

Do not only memorize:

> VPC = virtual network.

Understand packet flow.

Example:

```text
Internet
   ↓
Internet Gateway
   ↓
Public Subnet
   ↓
Load Balancer
   ↓
Private Subnet
   ↓
EC2
   ↓
RDS
```

## Learn

```text
VPC
CIDR
Subnet
Public Subnet
Private Subnet
Route Table
Internet Gateway
NAT Gateway
Elastic IP
Security Group
NACL
VPC Endpoint
DNS
DHCP
VPC Peering
Transit Gateway
```

---

# PHASE 3.1 — BUILD YOUR OWN VPC

Create:

```text
VPC
10.0.0.0/16
```

Subnets:

```text
Public-A
10.0.1.0/24

Public-B
10.0.2.0/24

Private-App-A
10.0.11.0/24

Private-App-B
10.0.12.0/24

Private-DB-A
10.0.21.0/24

Private-DB-B
10.0.22.0/24
```

Then:

```text
Internet Gateway
      ↓
Public Route Table
      ↓
Public Subnets
```

Private subnet architecture:

```text
Private Subnets
      ↓
NAT Gateway
      ↓
Internet
```

Database architecture:

```text
EC2
 ↓
Security Group
 ↓
RDS
```

RDS should normally not be directly exposed to the public internet.

---

# PHASE 3.2 — SECURITY GROUPS VS NACL

## Security Group

```text
Instance-level
Stateful
Allow rules
```

## NACL

```text
Subnet-level
Stateless
Allow + Deny rules
```

## Exercise

Create an EC2 Security Group:

```text
SSH
22
My IP only

HTTP
80
0.0.0.0/0

HTTPS
443
0.0.0.0/0
```

Then intentionally break a rule and troubleshoot why the connection fails.

---

# PHASE 4 — EC2

Learn:

```text
AMI
Instance
Instance Type
EBS
Snapshots
Volumes
Elastic IP
Key Pair
IAM Role
User Data
Security Groups
Placement
Auto Scaling
Load Balancer
CloudWatch
SSM
```

## First Project

Build:

```text
VPC
 ↓
Public Subnet
 ↓
EC2
 ↓
Nginx
 ↓
Internet
```

Then build:

```text
ALB
 ↓
Private EC2
```

Then:

```text
ALB
 ↓
Auto Scaling Group
 ↓
2 EC2 Instances
```

Learn:

```text
AMI
Launch Template
Auto Scaling
Target Group
ALB
Health Checks
```

---

# PHASE 4.1 — SYSTEMS MANAGER

Learn:

```text
SSM Session Manager
Run Command
Patch Manager
Parameter Store
Automation
```

Goal:

> Manage EC2 machines without relying on direct SSH for every operational task.

---

# PHASE 5 — S3

Learn:

```text
Bucket
Object
Prefix
Versioning
Encryption
Bucket Policy
IAM Policy
ACL
Block Public Access
Lifecycle
Replication
Storage Classes
Object Lock
Presigned URL
Multipart Upload
Events
Access Logs
CloudTrail
```

## Exercise

Create:

```text
company-private-data
```

Configure:

```text
Block Public Access
Versioning
Encryption
Lifecycle
Bucket Policy
```

Then understand:

```text
Private S3
Public S3
CloudFront + private S3
```

For modern CloudFront-to-S3 architectures, learn Origin Access Control (OAC) and how to keep the bucket private.

---

# PHASE 6 — RDS

Learn:

```text
RDS
Aurora
PostgreSQL
MySQL
Subnet Groups
Security Groups
Backups
Snapshots
Multi-AZ
Read Replicas
Encryption
Parameter Groups
Option Groups
Performance Insights
Monitoring
```

## Build

```text
Internet
   ↓
ALB
   ↓
EC2
   ↓
RDS PostgreSQL
```

RDS:

```text
Private Subnet
No Public Access
```

Security Group:

```text
RDS SG
3306 / 5432
Source = Application SG
```

Avoid:

```text
0.0.0.0/0
```

for database access unless there is a very specific and controlled reason.

---

# PHASE 7 — ECR + CONTAINERS

Learn:

```text
Docker
Image
Container
Dockerfile
ECR Repository
Image Tags
Image Scanning
IAM
EC2
ECS
Fargate
```

Architecture:

```text
Developer
   ↓
Docker Build
   ↓
ECR
   ↓
ECS
   ↓
Fargate
   ↓
ALB
   ↓
Internet
```

Start with:

```text
ECR → EC2
```

Then:

```text
ECR → ECS Fargate
```

---

# PHASE 8 — ROUTE 53 + CLOUDFRONT + ACM

Learn Route 53:

```text
Hosted Zone
DNS
Record
A
AAAA
CNAME
Alias
Health Check
```

Learn CloudFront:

```text
Distribution
Origin
Cache Policy
Origin Request Policy
OAC
TLS
```

Learn ACM:

```text
TLS Certificates
HTTPS
Custom Domains
```

## Architecture

Frontend:

```text
User
 ↓
Route 53
 ↓
CloudFront
 ↓
S3
```

Backend:

```text
User
 ↓
Route 53
 ↓
CloudFront / ALB
 ↓
Backend
```

---

# PHASE 9 — SECURITY

Learn:

```text
CloudTrail
CloudWatch
AWS Config
GuardDuty
Security Hub
Inspector
Macie
KMS
Secrets Manager
WAF
Shield
```

Understand what each service answers.

## CloudTrail

Question:

> Who did what?

Examples:

```text
Who deleted this S3 object?
Who changed this Security Group?
Who created this IAM resource?
Who stopped this EC2?
```

## CloudWatch

Question:

> What is happening to my infrastructure and application?

Learn:

```text
Metrics
Logs
Alarms
Dashboards
Log Groups
Log Insights
Events
```

## GuardDuty

Question:

> Are there signs of suspicious or potentially malicious activity?

## Security Hub

Question:

> What is the security posture across AWS security findings?

## AWS Config

Question:

> Are AWS resources configured according to our rules?

---

# PHASE 10 — KMS + SECRETS

Learn:

```text
KMS
KMS Keys
Key Policies
IAM Policies
Encryption
Envelope Encryption
Secrets Manager
Parameter Store
```

Application pattern:

```text
Application
    ↓
Secrets Manager
    ↓
DB Credentials
    ↓
RDS
```

Do not put production credentials directly in source code:

```text
DB_PASSWORD=123456
```

---

# PHASE 11 — COST MANAGEMENT

This is essential for company AWS administration.

Learn:

```text
Billing Dashboard
Cost Explorer
Budgets
Cost Allocation Tags
Cost Categories
AWS Pricing Calculator
Savings Plans
Reserved Instances
EC2 Pricing
RDS Pricing
S3 Pricing
Data Transfer
NAT Gateway Costs
CloudFront Costs
ECR Costs
CloudWatch Costs
```

Your goal should be to answer:

```text
Why did the AWS bill increase?

Which service costs the most?

Which region costs the most?

Which account costs the most?

Which resources are generating cost?

What changed compared with last month?
```

---

# PHASE 11.1 — BILL INVESTIGATION

Example:

```text
EC2                  $40
RDS                   $70
NAT Gateway           $35
S3                     $5
CloudWatch             $8
Data Transfer         $20
```

Then investigate:

```text
Cost Explorer
 ↓
Service
 ↓
Region
 ↓
Account
 ↓
Resource / Tag
 ↓
Usage change
```

---

# PHASE 11.2 — NAT GATEWAY COST

Understand:

```text
Private EC2
     ↓
NAT Gateway
     ↓
Internet
```

Then learn when VPC Endpoints can reduce the need for NAT-based traffic and costs for supported AWS services.

---

# PHASE 12 — MONITORING + OPERATIONS

Become comfortable operating infrastructure.

## Operational Checklist

### Daily / Regular Checks

```text
EC2 health
RDS health
CPU
Memory
Disk
Network
Application logs
CloudWatch alarms
Failed deployments
Security findings
IAM changes
CloudTrail
S3 public access
Unused resources
Cost
```

---

# PHASE 12.1 — INCIDENT MANAGEMENT

Do not only learn how to create AWS resources.

Learn:

> Something is broken. How do I find it?

Example:

```text
Application is Down
        ↓
DNS
        ↓
CloudFront
        ↓
ALB
        ↓
Target Group
        ↓
Security Group
        ↓
EC2
        ↓
Application
        ↓
Database
```

At each layer ask:

```text
Is it reachable?
Is DNS correct?
Is the port open?
Is the Security Group correct?
Is the process running?
Is CPU high?
Is memory full?
Is disk full?
Is the database reachable?
Did something recently change?
```

This is real AWS operational skill.

---

# PHASE 13 — BACKUP + DISASTER RECOVERY

Learn:

```text
EBS Snapshots
RDS Backups
RDS Snapshots
S3 Versioning
S3 Replication
AWS Backup
Multi-AZ
Multi-Region Concepts
RTO
RPO
Disaster Recovery
```

Understand:

```text
RPO
How much data can we afford to lose?

RTO
How long can the system be unavailable?
```

---

# PHASE 14 — SAGEMAKER / AI

Only after the foundational AWS services are comfortable.

Learn:

```text
SageMaker
S3
IAM
VPC
ECR
CloudWatch
KMS
Secrets Manager
```

SageMaker topics:

```text
SageMaker Studio
Notebooks
Training Jobs
Processing Jobs
Models
Endpoints
Inference
Model Registry
Pipelines
Experiments
Monitoring
IAM Roles
VPC Configuration
S3 Integration
ECR Integration
```

Architecture:

```text
Data
 ↓
S3
 ↓
SageMaker
 ↓
Training
 ↓
Model
 ↓
Model Artifacts
 ↓
Endpoint
 ↓
Application
```

---

# PHASE 15 — AWS CLI

Do not stay Console-only forever.

Use this progression:

```text
Phase 1
Console
██████████
CLI
██
IaC
█
```

Then:

```text
Phase 2
Console
██████
CLI
██████
Terraform / CloudFormation
████
```

Eventually:

```text
Console      → Understand
CLI          → Operate
Infrastructure as Code → Reproduce
Automation   → Manage at Scale
```

## Example Commands

```bash
aws s3 ls
```

```bash
aws ec2 describe-instances
```

```bash
aws iam list-users
```

```bash
aws rds describe-db-instances
```

After learning a service in the Console, perform equivalent operations through CLI.

---

# PHASE 16 — TERRAFORM / INFRASTRUCTURE AS CODE

Learn:

```text
Terraform
Provider
Resource
Variable
Output
Module
State
Remote State
Backend
Workspace
Plan
Apply
Destroy
```

Progression:

```text
Console
   ↓
AWS CLI
   ↓
Terraform
   ↓
CI/CD
```

Eventually:

```text
Git
 ↓
Terraform
 ↓
AWS
```

The goal is to understand the infrastructure manually first and then reproduce it as code.

---

# PHASE 17 — COMPANY-SCALE PROJECT

Build one complete company-like AWS environment.

## Architecture

```text
                    Route 53
                       │
                       ↓
                  CloudFront
                       │
              ┌────────┴────────┐
              ↓                 ↓
             S3               ALB
                                │
                         ┌──────┴──────┐
                         ↓             ↓
                       EC2           EC2
                         │             │
                         └──────┬──────┘
                                ↓
                              RDS
```

Container path:

```text
ECR
 ↓
Docker
 ↓
EC2 / ECS
```

Security:

```text
IAM
IAM Identity Center
KMS
Secrets Manager
Security Groups
CloudTrail
GuardDuty
Security Hub
AWS Config
```

Monitoring:

```text
CloudWatch
```

Cost:

```text
Budgets
Cost Explorer
Tags
Pricing Calculator
```

Then reproduce the architecture using Terraform.

---

# PHASE 18 — COMPANY SIMULATION

Create a fake company structure:

```text
Company
│
├── Platform Team
├── Backend Team
├── Frontend Team
├── Data Team
├── Security Team
└── Finance
```

Users/roles:

```text
admin
devops
backend-dev
frontend-dev
data-scientist
security-auditor
billing-user
```

Build permissions based on responsibilities.

Example:

```text
backend-dev
   ↓
EC2
ECR
CloudWatch
S3 application bucket

NOT
   ↓
Organizations
IAM administration
Billing
```

This teaches much more than simply launching individual AWS services.

---

# PHASE 19 — LEARNING ORDER

A practical sequence:

```text
WEEK 1
AWS Account
Root
Billing
Regions
Tags
IAM Basics

WEEK 2
IAM Policies
Roles
Trust Policies
STS
Access Analyzer

WEEK 3
IAM Identity Center
Users
Groups
Permission Sets
MFA

WEEK 4
Organizations
Accounts
OUs
SCP
Cross-account Roles

WEEK 5
VPC
CIDR
Subnets
Route Tables
IGW
NAT

WEEK 6
Security Groups
NACL
VPC Endpoints
DNS
VPC Troubleshooting

WEEK 7
EC2
AMI
EBS
SSM
IAM Roles
User Data

WEEK 8
ALB
Target Groups
Auto Scaling
Launch Templates

WEEK 9
S3
Policies
Encryption
Versioning
Lifecycle
Replication

WEEK 10
RDS
Subnet Groups
Security
Backups
Multi-AZ
Read Replicas

WEEK 11
Docker
ECR
ECS
Fargate

WEEK 12
Route53
CloudFront
ACM
WAF
OAC

WEEK 13
CloudWatch
CloudTrail
Config
GuardDuty
Security Hub

WEEK 14
KMS
Secrets Manager
SSM

WEEK 15
Billing
Cost Explorer
Budgets
Tags
Pricing Calculator

WEEK 16
Backup
DR
RTO
RPO

WEEK 17+
Terraform
CI/CD
Advanced Networking
SageMaker
AI Infrastructure
```

These are learning stages, not strict deadlines.

---

# PHASE 20 — HOW TO USE AWS DOCUMENTATION

Do not read all AWS documentation from beginning to end.

Use three layers.

## Layer 1 — Learn

Use AWS Skill Builder:

https://skillbuilder.aws/

Use hands-on labs for:

```text
VPC
S3
EC2
IAM
KMS
CloudFront
Security
```

## Layer 2 — Understand

Use official AWS documentation:

https://docs.aws.amazon.com/

For every service, study:

```text
Overview
Getting Started
Concepts
Security
Pricing
Best Practices
Troubleshooting
FAQ
```

## Layer 3 — Reference

When you have a specific problem, search the documentation for the exact feature.

Example:

Instead of reading all of S3 documentation, search:

```text
CloudFront S3 Origin Access Control
```

when your question is about private S3 + CloudFront.

---

# PHASE 21 — IMPORTANT AWS DOCUMENTATION

## IAM

https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html

Study:

```text
Policies
Roles
Trust Policies
Policy Evaluation
Access Analyzer
Identity Center
```

## Organizations

https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html

Study:

```text
Accounts
OUs
SCP
Billing
Multi-account Architecture
```

## VPC

https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html

Study deeply.

## EC2

https://docs.aws.amazon.com/ec2/

## S3

https://docs.aws.amazon.com/s3/

## RDS

https://docs.aws.amazon.com/rds/

## ECR

https://docs.aws.amazon.com/ecr/

## CloudFront

https://docs.aws.amazon.com/cloudfront/

## CloudWatch

https://docs.aws.amazon.com/cloudwatch/

## SageMaker

https://docs.aws.amazon.com/sagemaker/

---

# PHASE 22 — AWS WELL-ARCHITECTED FRAMEWORK

Study:

https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html

Six pillars:

```text
1. Operational Excellence
2. Security
3. Reliability
4. Performance Efficiency
5. Cost Optimization
6. Sustainability
```

Do not simply memorize them.

Use them to review your own projects.

Example:

> I created an EC2 + RDS application. Is it production-ready?

Review:

```text
Security?
Reliability?
Cost?
Performance?
Operations?
Sustainability?
```

---

# PHASE 23 — AWS SECURITY REFERENCE ARCHITECTURE

Study:

https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/welcome.html

Use this to understand how AWS security can be structured across accounts and services.

Study:

```text
Centralized security
Security accounts
Log archive
Security services
Identity
Governance
Detection
Monitoring
```

---

# PHASE 24 — WHAT NOT TO DO

Avoid learning like this:

```text
Today EC2
Tomorrow Lambda
Next day DynamoDB
Next day SageMaker
Next day EKS
```

You may know dozens of services but still struggle to operate a real environment.

Instead:

```text
IAM
 ↓
Organizations
 ↓
VPC
 ↓
Security
 ↓
EC2
 ↓
S3
 ↓
RDS
 ↓
ECR
 ↓
CloudFront
 ↓
Monitoring
 ↓
Cost
 ↓
Automation
 ↓
AI
```

This creates a connected mental model.

---

# PHASE 25 — THE REAL TARGET SKILL

Eventually, someone should be able to say:

> "Our backend is down."

Your thinking should become:

```text
1. Check CloudWatch
2. Check ALB
3. Check Target Health
4. Check EC2
5. Check Security Group
6. Check Networking
7. Check Application Logs
8. Check RDS
9. Check Recent Deployments
10. Check CloudTrail for infrastructure changes
```

Another example:

> "Developer needs access to production S3."

Think:

```text
Who?
 ↓
What bucket?
 ↓
What actions?
 ↓
What objects?
 ↓
Read or Write?
 ↓
Which IAM Identity Center group?
 ↓
Which Permission Set?
 ↓
Least Privilege?
 ↓
Conditions?
 ↓
Access Analyzer?
 ↓
CloudTrail monitoring?
```

Another:

> "AWS bill suddenly increased."

Think:

```text
Cost Explorer
 ↓
Service
 ↓
Region
 ↓
Account
 ↓
Resource / Tag
 ↓
Usage Change
 ↓
NAT?
EC2?
RDS?
Data Transfer?
S3?
CloudWatch?
```

That is the target skill.

---

# MASTER AWS ROADMAP

```text
                         AWS ADMIN / CLOUD MASTER
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
          GOVERNANCE           SECURITY            COST
              │                   │                   │
       Organizations           IAM                 Billing
       Accounts                Roles               Cost Explorer
       OUs                     Policies             Budgets
       SCP                     KMS                  Tags
       Identity Center         CloudTrail           Pricing
              │                 GuardDuty
              │                 Security Hub
              │                 Config
              │
              └───────────────┐
                              ↓
                          NETWORKING
                              │
             VPC → Subnet → Route → SG → NACL
                              │
              ┌───────────────┼───────────────┐
              ↓               ↓               ↓
             EC2             S3              RDS
              │               │               │
              ↓               ↓               ↓
             ECR          CloudFront       Backup
              │
              ↓
             ECS
              │
              ↓
          Application
              │
              ↓
      CloudWatch / CloudTrail
              │
              ↓
       Automation / Terraform
              │
              ↓
       SageMaker / AI / ML
```

---

# THE LEARNING METHOD TO FOLLOW

For every AWS service, use this cycle:

```text
Learn
  ↓
Build
  ↓
Break
  ↓
Troubleshoot
  ↓
Secure
  ↓
Monitor
  ↓
Cost-check
  ↓
Automate
```

Example:

```text
Learn EC2
   ↓
Create EC2
   ↓
Break Security Group
   ↓
Troubleshoot
   ↓
Add IAM Role
   ↓
Monitor CloudWatch
   ↓
Check Cost
   ↓
Recreate with Terraform
```

Repeat this for:

```text
IAM
Organizations
VPC
EC2
S3
RDS
ECR
ECS
CloudFront
Route53
Security
Monitoring
Cost
Backup
SageMaker
```

The objective is not to memorize AWS services.

The objective is to understand:

```text
How AWS resources connect
How permissions work
How networking works
How applications communicate
How security is enforced
How failures are diagnosed
How costs are controlled
How infrastructure is reproduced
How production environments are operated
```

That is the path from AWS beginner/intermediate knowledge toward real AWS administration and cloud engineering.
