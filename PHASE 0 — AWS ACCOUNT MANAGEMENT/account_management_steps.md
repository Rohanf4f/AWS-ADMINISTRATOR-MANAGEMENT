# AWS ACCOUNT MANAGEMENT

> **Goal:** Understand how an AWS account is structured and how you safely operate the account before going deep into IAM, Organizations, networking, compute, databases, etc.
>
> **Important:** Phase 0 is about account-level understanding and safe operation. Deep IAM, IAM Identity Center, and AWS Organizations configuration will come later.

---

# 1. What You Need to Learn in Phase 0

You should understand these topics:

- AWS Account
- Root User
- IAM
- IAM Identity Center
- AWS Organizations
- AWS Regions
- Availability Zones
- Service Quotas
- Billing
- AWS Support
- Resource Groups
- Tags

You should also become familiar with where these services appear in the AWS Console:

- AWS Console
- IAM
- IAM Identity Center
- Organizations
- Billing & Cost Management
- EC2
- VPC
- S3
- RDS
- ECR
- CloudFront
- Route 53
- CloudWatch
- CloudTrail
- Security Hub
- GuardDuty
- Config
- SageMaker

---

# 2. Phase 0 Mental Model

Think about AWS like this:

```text
AWS Account
│
├── Root User
│
├── Billing
│
├── Regions
│   └── Availability Zones
│
├── Security / Daily Access
│   ├── IAM
│   └── IAM Identity Center
│
├── Resources
│   ├── EC2
│   ├── S3
│   ├── RDS
│   ├── ECR
│   └── etc.
│
├── Tags
│   └── Resource Groups
│
└── Monitoring / Auditing
    ├── CloudWatch
    ├── CloudTrail
    ├── Security Hub
    ├── GuardDuty
    └── Config

Usage
  ↓
Billing
  ↓
Cost Explorer
  ↓
Budgets
```

Your goal is to understand this structure before moving to Phase 1.

---

# 3. AWS Account

## What is an AWS Account?

An AWS account is the main security, billing, and resource boundary in AWS.

An account contains:

- AWS resources
- users/identities
- permissions
- billing information
- security configuration
- regional resources
- logs
- quotas
- account settings

For a company, the account is an important administrative boundary.

---

# 4. Open the AWS Management Console

Open:

https://console.aws.amazon.com/

Sign in to your AWS account.

After signing in, look at the top-right area of the AWS Console.

You should be able to find things such as:

- AWS Region
- Account ID
- Account name
- signed-in identity

Your first exercise:

```text
Find:
✓ Account ID
✓ Account name
✓ Current Region
✓ Signed-in identity
```

Write these down somewhere safe for your own learning notes.

Do not share your AWS credentials, passwords, secret keys, or MFA codes.

---

# 5. Root User

## What is the Root User?

When an AWS account is created, AWS creates the AWS account root user.

The root user has complete access to the AWS account.

AWS strongly recommends that you do **not** use the root user for everyday AWS work.

Use the root user only for tasks that specifically require root credentials.

AWS documentation:

https://docs.aws.amazon.com/IAM/latest/UserGuide/root-user-best-practices.html

The root user has unrestricted access, including access to billing information.

---

# 6. Root User Security

Your first important Phase 0 security task is to secure the root user.

AWS recommends:

```text
Root User
   ↓
Strong unique password
   ↓
MFA
   ↓
Do NOT create root access keys
   ↓
Use root only when specifically required
```

AWS currently recommends MFA for the root user and strongly recommends not creating root access keys.

Source:

https://docs.aws.amazon.com/IAM/latest/UserGuide/root-user-best-practices.html

---

# 7. Enable Root MFA

Sign in as the root user.

Go to:

```text
AWS Console
    ↓
Account / Security credentials
    ↓
Root user
    ↓
Multi-factor authentication (MFA)
```

Follow the AWS console instructions to register your MFA device.

You may use a supported MFA method such as:

- authenticator application
- TOTP hardware device
- FIDO security key

After enabling MFA:

```text
Root Password
      +
     MFA
      ↓
Root Account Access
```

Do not share:

- root password
- MFA code
- recovery information
- root credentials

---

# 8. Check Root Access Keys

Check whether the root account has access keys.

The recommended state for normal company administration is:

```text
Root access keys
      ↓
Do not create them
```

If you discover existing root access keys, do not casually delete them without understanding their usage and impact.

AWS recommends avoiding root access keys because the root user has full access.

Documentation:

https://docs.aws.amazon.com/IAM/latest/UserGuide/root-user-best-practices.html

---

# 9. Root User vs Normal AWS Access

Understand this distinction:

```text
ROOT USER
    ↓
Complete account-level access
    ↓
Use only for root-required tasks


NORMAL ADMINISTRATIVE ACCESS
    ↓
IAM / IAM Identity Center / roles
    ↓
Daily AWS work
```

Do not start using root as your everyday AWS administrator.

Deep IAM and IAM Identity Center configuration will be covered in later phases.

---

# 10. AWS Account Settings

Open:

```text
AWS Console
    ↓
Account
```

Explore the account-level settings.

Look for things such as:

- Account ID
- Account name
- contact information
- alternate contacts
- security-related account settings
- billing-related settings
- region-related settings

Do not randomly modify settings just to experiment.

Your goal at this stage is:

```text
Know where account-level configuration lives.
```

---

# 11. AWS Regions

## What is a Region?

An AWS Region is a separate geographic area where AWS operates infrastructure.

Examples include:

```text
ap-south-1
us-east-1
us-west-2
eu-west-1
```

For example:

```text
ap-south-1
      ↓
Mumbai Region
```

AWS Regions documentation:

https://docs.aws.amazon.com/global-infrastructure/latest/regions/aws-regions-availability-zones.html

---

# 12. Practice: Switch AWS Regions

In the AWS Console, find the Region selector at the top-right.

Click it.

You will see regions such as:

```text
US East
US West
Asia Pacific
Europe
Middle East
etc.
```

For learning, you can switch between regions without creating resources.

For example:

```text
ap-south-1
      ↓
us-east-1
      ↓
ap-south-1
```

Understand:

> AWS resources are often regional.

This becomes extremely important later.

---

# 13. Why Region Matters

Suppose you create:

```text
EC2 instance
Region = ap-south-1
```

Then switch the console to:

```text
us-east-1
```

You may not see that EC2 instance because you are now looking at another region.

Mental model:

```text
AWS Account
│
├── ap-south-1
│   ├── EC2
│   ├── RDS
│   └── VPC
│
├── us-east-1
│   ├── EC2
│   └── S3-related resources
│
└── other Regions
```

Some AWS services/resources are global or have global aspects, while many are regional.

Always know which region you are operating in.

---

# 14. Availability Zones

A Region contains Availability Zones.

Example:

```text
Region
ap-south-1
│
├── Availability Zone
├── Availability Zone
└── Availability Zone
```

An Availability Zone is an isolated location within an AWS Region.

AWS documentation:

https://docs.aws.amazon.com/global-infrastructure/latest/regions/aws-availability-zones.html

---

# 15. Region vs Availability Zone

Remember:

```text
Region
    ↓
Geographic AWS area

Availability Zone
    ↓
Isolated infrastructure location inside a Region
```

Example:

```text
ap-south-1
│
├── ap-south-1a
├── ap-south-1b
└── ap-south-1c
```

The exact AZ mapping behavior can vary by account and AWS documentation has additional details about AZ IDs.

For Phase 0, just understand the architecture.

---

# 16. Why Availability Zones Matter

Later, when you build highly available applications, you may use multiple AZs.

For example:

```text
                    Load Balancer
                         │
              ┌──────────┴──────────┐
              │                     │
          AZ-a                    AZ-b
              │                     │
           Server                 Server
```

If one Availability Zone has a problem, the application may continue operating through another AZ, depending on how the architecture is designed.

Do not build this yet in Phase 0.

Just understand the concept.

---

# 17. Billing & Cost Management

Open:

```text
AWS Console
    ↓
Billing and Cost Management
```

Explore:

- Billing Dashboard
- Bills
- Cost Explorer
- Budgets
- Payments
- Cost allocation / cost management features available to your account

Your first billing goal is simply:

> Know where to look to understand how much AWS is costing the account.

---

# 18. Billing Dashboard

Open:

```text
Billing and Cost Management
    ↓
Billing Dashboard
```

Look at:

- current charges
- previous charges
- estimated charges where available
- service-level spending information

Do not worry about understanding every billing field yet.

You are learning where the information is.

---

# 19. Bills

Open:

```text
Billing
    ↓
Bills
```

Explore the current billing period.

Try to understand:

```text
Service
   ↓
Usage
   ↓
Charge
```

For example:

```text
EC2
S3
RDS
Data Transfer
CloudFront
etc.
```

The exact services shown depend on what you have used.

---

# 20. Cost Explorer

Open:

```text
Billing and Cost Management
    ↓
Cost Explorer
```

Cost Explorer helps you analyze AWS spending.

For learning, inspect:

```text
Cost over time
Cost by service
Cost by region
Cost by usage
```

Ask yourself:

> If my manager asks "Why did AWS cost increase this month?", where would I investigate?

The answer starts with:

```text
Billing
   ↓
Cost Explorer
   ↓
Service / Region / Usage analysis
```

---

# 21. AWS Budgets

Open:

```text
Billing
    ↓
Budgets
```

Understand the concept:

```text
Budget
   ↓
Threshold
   ↓
Actual / forecasted spending
   ↓
Alert
```

Example:

```text
Monthly budget = $50

Actual spending
       ↓
      $40

Alert threshold
       ↓
      80%

Notification
```

You do not need to create many complicated budgets during Phase 0.

Understand what Budgets is used for.

---

# 22. AWS Pricing Calculator

Use:

https://calculator.aws/

This is useful before deploying infrastructure.

Example:

```text
Application
    ↓
EC2
    ↓
RDS
    ↓
S3
    ↓
CloudFront
    ↓
Data transfer
    ↓
Estimated monthly cost
```

Practice estimating a small backend system.

Example architecture:

```text
EC2
+
RDS
+
S3
```

You don't need exact production numbers yet.

The important habit is:

> Estimate cost before deploying infrastructure.

---

# 23. Service Quotas

AWS services have quotas, sometimes called limits.

Examples can include limits on:

- number of resources
- API request rates
- concurrent resources
- regional capacity-related limits
- other service-specific values

AWS documentation:

https://docs.aws.amazon.com/servicequotas/

AWS states that quotas are the maximum number of resources or related limits applicable to an AWS account/service, and many quotas are regional.

---

# 24. Open Service Quotas

In the AWS Console search:

```text
Service Quotas
```

Open it.

Choose a service.

Explore:

```text
Service
   ↓
Quota
   ↓
Current value / default quota
   ↓
Adjustable?
```

You may see quota information such as:

```text
EC2
S3
VPC
RDS
etc.
```

Do not request quota increases just for practice.

The goal is:

> Know that quotas exist and know where to inspect them.

---

# 25. Why Service Quotas Matter

Imagine your application needs:

```text
100 EC2 resources
```

but the account/service quota is:

```text
50
```

Your architecture or deployment may fail until the relevant limit is addressed.

Therefore:

```text
Architecture planning
       ↓
Expected resource usage
       ↓
Service quotas
       ↓
Capacity planning
```

Later, this becomes important for production deployments.

---

# 26. Tags

Tags are key-value metadata attached to AWS resources.

Example:

```text
Environment = dev
Project     = aws-learning
Application = test-api
Team        = backend
Owner       = admin
CostCenter  = learning
ManagedBy   = manual
```

A tag looks like:

```text
Key = Environment
Value = dev
```

---

# 27. Why Tags Matter

Tags help you:

- identify resources
- organize resources
- search resources
- group resources
- support cost allocation
- automate management
- identify ownership
- distinguish environments

Think:

```text
Resource
   ↓
Tags
   ↓
Identification
Organization
Cost tracking
Automation
```

---

# 28. Recommended Learning Tag Standard

For your learning account, you can use:

```text
Environment=dev
Project=aws-learning
Application=test-api
Team=backend
Owner=admin
CostCenter=learning
ManagedBy=manual
```

For a company, the exact standard should be agreed with the organization's engineering/finance/security requirements.

Do not put:

- passwords
- API keys
- secret tokens
- sensitive personal data

inside tags.

Tags are metadata, not secret storage.

---

# 29. Resource Groups

AWS Resource Groups lets you organize AWS resources using criteria such as tags.

AWS documentation:

https://docs.aws.amazon.com/ARG/latest/userguide/gettingstarted.html

Mental model:

```text
Resources
    ↓
Tags
    ↓
Resource Group
```

Example:

```text
EC2
Environment=dev

RDS
Environment=dev

S3
Environment=dev
```

You can create a group for:

```text
Environment=dev
```

and use that criteria to find matching resources.

---

# 30. Practice Resource Groups

In the AWS Console search:

```text
Resource Groups & Tag Editor
```

Open it.

Explore:

```text
Tag Editor
Resource Groups
```

AWS's documentation explains that Resource Groups can organize resources based on tags and resource types.

Try to understand:

```text
Tag:
Environment=dev

        ↓

Resource Group:
Dev Resources
```

You may need existing resources before the grouping becomes useful.

---

# 31. AWS Support

Open:

```text
AWS Console
    ↓
Support / Support Center
```

Learn where you can:

- view support information
- create support cases when your account/plan permits
- review existing cases
- access AWS support resources

The goal in Phase 0 is simply:

> Know where AWS support is located and where an operational issue can be raised.

Do not spend too much time here yet.

---

# 32. IAM — Phase 0 Understanding Only

IAM means:

```text
Identity and Access Management
```

IAM controls:

```text
WHO
  ↓
CAN DO WHAT
  ↓
ON WHICH AWS RESOURCE
```

Example:

```text
Developer
   ↓
Can read S3
   ↓
Cannot delete production database
```

For Phase 0, understand the purpose.

Do **not** try to master IAM policies yet.

IAM gets its own deep phase later.

---

# 33. IAM Identity Center — Phase 0 Understanding Only

IAM Identity Center is used to centrally manage human access to AWS accounts and applications.

Think:

```text
Company Employee
       ↓
IAM Identity Center
       ↓
Permission Set
       ↓
AWS Account
       ↓
Temporary Access
```

For a company environment, this becomes important for centralized workforce access.

Deep IAM Identity Center configuration comes later.

---

# 34. AWS Organizations — Phase 0 Understanding Only

AWS Organizations lets you centrally manage multiple AWS accounts.

Think:

```text
AWS Organization
│
├── Management Account
│
├── Development Account
│
├── Staging Account
│
└── Production Account
```

This becomes important when a company separates environments or business units into multiple AWS accounts.

Do not configure a production-like organization just for Phase 0.

Deep Organizations learning comes later.

---

# 35. Major AWS Console Areas

During Phase 0, you only need to know what these services are for.

## IAM

```text
Identity
Permissions
Access
```

## IAM Identity Center

```text
Centralized workforce access
SSO
Permission sets
```

## Organizations

```text
Multiple AWS accounts
Centralized account management
```

## Billing & Cost Management

```text
AWS spending
Bills
Cost Explorer
Budgets
Payments
```

## EC2

```text
Virtual servers
```

## VPC

```text
Networking
```

## S3

```text
Object storage
```

## RDS

```text
Managed relational databases
```

## ECR

```text
Container image registry
```

## CloudFront

```text
CDN
Content delivery
```

## Route 53

```text
DNS
Domain-related services
```

## CloudWatch

```text
Metrics
Logs
Alarms
Monitoring
```

## CloudTrail

```text
AWS API/activity auditing
```

## Security Hub

```text
Security findings
Security posture aggregation
```

## GuardDuty

```text
Threat detection
```

## AWS Config

```text
Resource configuration
Compliance evaluation
```

## SageMaker

```text
Machine learning / AI platform
```

At this stage:

```text
KNOW WHAT IT DOES
        ↓
DO NOT MASTER IT YET
```

---

# 36. Console Exploration Exercise

Open the AWS Console.

Use the search bar and search each service:

```text
IAM
IAM Identity Center
Organizations
Billing
EC2
VPC
S3
RDS
ECR
CloudFront
Route 53
CloudWatch
CloudTrail
Security Hub
GuardDuty
Config
SageMaker
```

For each service, answer:

```text
1. What is this service?
2. Why would a company use it?
3. Is it mainly security, networking, compute, storage, database,
   monitoring, cost, or AI?
4. Is it something I will learn deeply later?
```

Do not create resources in every service.

This is only orientation.

---

# 37. Phase 0 Practical Exercise

Perform these actions in your AWS account.

## Exercise 1 — Account

Find:

```text
[ ] Account ID
[ ] Account name
[ ] Current Region
[ ] Signed-in identity
```

---

## Exercise 2 — Root Security

Check:

```text
[ ] Root MFA enabled
[ ] Root password secured
[ ] Root access keys are not being used
[ ] Root credentials are not shared
```

AWS recommends using the root user only for tasks requiring root credentials.

---

## Exercise 3 — Regions

Practice:

```text
[ ] Find Region selector
[ ] Switch regions
[ ] Return to your learning Region
```

Understand:

```text
Region
  ↓
Availability Zones
```

---

## Exercise 4 — Billing

Open:

```text
[ ] Billing Dashboard
[ ] Bills
[ ] Cost Explorer
[ ] Budgets
```

Understand:

```text
Where do I see AWS spending?
```

---

## Exercise 5 — Pricing

Open:

```text
https://calculator.aws/
```

Estimate:

```text
EC2
+
RDS
+
S3
```

---

## Exercise 6 — Service Quotas

Open:

```text
Service Quotas
```

Check at least:

```text
[ ] EC2 quotas
[ ] VPC quotas
[ ] Another service quota
```

Understand:

```text
Quota
Current/default value
Adjustable or not
```

---

## Exercise 7 — Tags

Create a simple tagging standard:

```text
Environment=dev
Project=aws-learning
Application=test-api
Team=backend
Owner=admin
CostCenter=learning
ManagedBy=manual
```

When you create future learning resources, practice applying these tags where supported.

---

## Exercise 8 — Resource Groups

Open:

```text
Resource Groups & Tag Editor
```

Understand:

```text
Tags
  ↓
Resource query
  ↓
Resource Group
```

---

## Exercise 9 — Support

Find:

```text
Support Center
```

Understand where you would go if you had an AWS operational/account issue.

---

# 38. What NOT to Do in Phase 0

Do not try to learn everything simultaneously.

Avoid:

```text
❌ Deep IAM policy writing
❌ Complex VPC architecture
❌ Kubernetes
❌ Terraform
❌ Complex CI/CD
❌ Production database architecture
❌ Advanced CloudFront
❌ Advanced SageMaker
❌ Multi-account production organization
```

Those are later phases.

Phase 0 is about understanding the AWS environment.

---

# 39. 7-Day Phase 0 Plan

## Day 1 — AWS Account

Learn:

```text
AWS Account
Account ID
Account name
Account settings
AWS Console
```

Practice:

```text
Find your account information.
```

---

## Day 2 — Root User

Learn:

```text
Root user
Root security
MFA
Root access keys
```

Practice:

```text
Secure root.
```

AWS documentation:

https://docs.aws.amazon.com/IAM/latest/UserGuide/root-user-best-practices.html

---

## Day 3 — Regions & AZs

Learn:

```text
Region
Availability Zone
Regional resources
```

Practice:

```text
Switch regions in the Console.
```

Documentation:

https://docs.aws.amazon.com/global-infrastructure/latest/regions/aws-regions-availability-zones.html

---

## Day 4 — Billing

Learn:

```text
Billing Dashboard
Bills
Cost Explorer
Budgets
```

Practice:

```text
Find current spending.
```

---

## Day 5 — Service Quotas

Learn:

```text
Quotas
Limits
Regional quotas
Adjustable quotas
```

Practice:

```text
Open Service Quotas.
Inspect EC2/VPC quotas.
```

Documentation:

https://docs.aws.amazon.com/servicequotas/

---

## Day 6 — Tags

Learn:

```text
Key
Value
Resource metadata
Cost allocation
Organization
```

Practice:

```text
Create your tagging standard.
```

---

## Day 7 — Resource Groups + Support

Learn:

```text
Resource Groups
Tag Editor
Support Center
```

Practice:

```text
Create/inspect resource groups.
Find AWS Support.
```

Resource Groups documentation:

https://docs.aws.amazon.com/ARG/latest/userguide/gettingstarted.html

---

# 40. Phase 0 Official Documentation

## AWS Account Setup

https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started-account-iam.html

Use this to learn how AWS recommends setting up an account.

---

## Root User Best Practices

https://docs.aws.amazon.com/IAM/latest/UserGuide/root-user-best-practices.html

Important topics:

- Root security
- MFA
- Root access keys
- Daily access
- Root credentials

---

## AWS Root User

https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-user.html

---

## AWS Regions and Availability Zones

https://docs.aws.amazon.com/global-infrastructure/latest/regions/aws-regions-availability-zones.html

Use this to understand:

```text
Region
Availability Zone
AWS global infrastructure
```

---

## AWS Availability Zones

https://docs.aws.amazon.com/global-infrastructure/latest/regions/aws-availability-zones.html

---

## Service Quotas

https://docs.aws.amazon.com/servicequotas/

Use this to understand:

```text
Quotas
Limits
Quota values
Quota increases
```

---

## Resource Groups

https://docs.aws.amazon.com/ARG/latest/userguide/gettingstarted.html

Use this to learn:

```text
Tag Editor
Resource Groups
Resource organization
```

---

## AWS Pricing Calculator

https://calculator.aws/

Use this before deploying resources when you want to estimate cost.

---

## AWS Documentation Home

https://docs.aws.amazon.com/

Use the official AWS documentation as your primary reference.

---

## AWS Skill Builder

https://skillbuilder.aws/

Use AWS Skill Builder for structured AWS learning and training.

---

# 41. Phase 0 Completion Checklist

Before moving to Phase 1, make sure you understand:

```text
AWS ACCOUNT
[ ] What an AWS account is
[ ] Account ID
[ ] Account name
[ ] Account settings


ROOT USER
[ ] What root user is
[ ] Why root should not be used for daily work
[ ] Root MFA
[ ] Root access keys
[ ] Root security


REGIONS
[ ] What a Region is
[ ] What an Availability Zone is
[ ] Region vs AZ
[ ] Why Region matters
[ ] How to switch regions


BILLING
[ ] Billing Dashboard
[ ] Bills
[ ] Cost Explorer
[ ] Budgets
[ ] Pricing Calculator


SERVICE QUOTAS
[ ] What a quota is
[ ] Where to view quotas
[ ] Regional quota concept
[ ] Adjustable vs non-adjustable quotas


TAGS
[ ] Key/value tags
[ ] Why tags matter
[ ] Basic company tagging concept
[ ] Never put secrets in tags


RESOURCE GROUPS
[ ] Tag Editor
[ ] Resource Groups
[ ] Tag-based organization


SUPPORT
[ ] Where Support Center is
[ ] Where AWS support information/cases are accessed


AWS SERVICES
[ ] IAM
[ ] IAM Identity Center
[ ] Organizations
[ ] EC2
[ ] VPC
[ ] S3
[ ] RDS
[ ] ECR
[ ] CloudFront
[ ] Route 53
[ ] CloudWatch
[ ] CloudTrail
[ ] Security Hub
[ ] GuardDuty
[ ] Config
[ ] SageMaker
```

---

# 42. The Most Important Phase 0 Mental Model

You should be able to explain this without looking at notes:

```text
                         AWS ACCOUNT
                              │
             ┌────────────────┼────────────────┐
             │                │                │
          ROOT             BILLING          SECURITY
             │                │                │
          MFA              Costs          IAM / Identity
             │                │                │
      Root-only tasks     Cost Explorer   Daily access
                              │
                           Budgets
                              │
                         Cost control


                    AWS INFRASTRUCTURE
                           │
                       REGION
                           │
                 ┌─────────┴─────────┐
                 │                   │
                AZ-A                AZ-B
                 │                   │
              Resources          Resources


                       RESOURCES
                           │
                         TAGS
                           │
                    RESOURCE GROUPS
```

And:

```text
Account
   ↓
Region
   ↓
Availability Zone
   ↓
Resource
   ↓
Tags
   ↓
Organization / Management
```

---

# 43. Phase 0 Final Goal

At the end of Phase 0, you should be comfortable opening the AWS Console and answering:

```text
Which AWS account am I in?

Which identity am I using?

Which Region am I operating in?

What Availability Zones are in that Region?

How much is AWS costing?

Where can I see the bill?

Where can I analyze cost?

Where can I configure a budget?

Where can I check service quotas?

How do I organize resources with tags?

How do I find tagged resources?

Where is AWS Support?

What is IAM?

What is IAM Identity Center?

What is AWS Organizations?

What do EC2, VPC, S3, RDS, ECR,
CloudFront, Route 53, CloudWatch,
CloudTrail, Security Hub, GuardDuty,
Config, and SageMaker do?
```

If you can answer those questions, you are ready to start the next phase.

---

# 44. One Rule for Learning AWS

Use this learning cycle for every future AWS service:

```text
LEARN
  ↓
BUILD
  ↓
BREAK
  ↓
TROUBLESHOOT
  ↓
SECURE
  ↓
MONITOR
  ↓
CHECK COST
  ↓
AUTOMATE
```

For Phase 0, however, focus mainly on:

```text
LEARN
  ↓
EXPLORE CONSOLE
  ↓
PRACTICE
  ↓
CHECK SECURITY
  ↓
CHECK COST
```

Do not rush into Phase 1 until the Phase 0 checklist is comfortable.
