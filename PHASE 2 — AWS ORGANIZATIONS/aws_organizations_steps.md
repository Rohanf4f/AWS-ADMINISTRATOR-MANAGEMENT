# PHASE 2 — AWS ORGANIZATIONS

> **Goal:** Learn how companies manage multiple AWS accounts centrally.
>
> After becoming comfortable with IAM, AWS Organizations is the next major step because it moves the mental model from:
>
> ```text
> One AWS Account
> ```
>
> to:
>
> ```text
> Multiple AWS Accounts
>         ↓
> Central Governance
>         ↓
> Security
>         ↓
> Billing
>         ↓
> Access Control
>         ↓
> Organizational Guardrails
> ```
>
> AWS Organizations helps centrally manage AWS accounts, group them, apply organization-level policies, and consolidate billing. citeturn0search9turn0search0

---

# 1. What You Need to Learn

Learn these concepts:

```text
Organization

Management Account

Member Account

Root

Organizational Unit (OU)

SCP

Consolidated Billing

Cross-Account Access

IAM Identity Center

Trusted Access

Delegated Administration

Account Lifecycle
```

The most important mental shift is:

```text
IAM
 ↓
Controls access inside / across AWS resources

Organizations
 ↓
Controls and governs multiple AWS accounts
```

---

# 2. AWS Organizations Mental Model

Think about Organizations like this:

```text
                         AWS ORGANIZATION
                                │
                         Management Account
                                │
                         Organization Root
                                │
          ┌─────────────────────┼─────────────────────┐
          │                     │                     │
      Security OU        Infrastructure OU       Workloads OU
          │                     │                     │
     ┌────┴────┐                │          ┌──────────┼──────────┐
     │         │                │          │          │          │
 Security   Log Archive     Network      Dev       Staging    Production
 Account     Account        Account     Account     Account     Account
```

This is an example architecture for learning.

AWS Organizations supports a hierarchy where accounts can be grouped into OUs, and OUs can contain other OUs. citeturn0search5turn0search10

---

# 3. Why Companies Use AWS Organizations

Imagine a company has:

```text
1 AWS Account
```

At the beginning, this may be manageable.

But eventually the company may have:

```text
Development
Staging
Production
Security
Logging
Networking
Data
Analytics
Sandbox
```

If everything lives in one account, separating:

```text
Security
Billing
Access
Production
Development
Auditing
```

becomes harder.

Organizations provides a way to centrally govern multiple AWS accounts. citeturn0search9

---

# 4. AWS Account vs Organization

Understand the difference.

## AWS Account

An AWS account is a boundary for:

```text
Resources
Security
Billing
IAM
Quotas
Service configuration
```

---

## AWS Organization

An Organization is a collection of AWS accounts managed together.

Conceptually:

```text
Organization
    │
    ├── Account A
    ├── Account B
    ├── Account C
    └── Account D
```

The organization gives you centralized governance capabilities.

---

# 5. Management Account

Every AWS Organization has a management account.

Conceptually:

```text
Organization
      │
      ↓
Management Account
      │
      ├── Manage organization
      ├── Manage member accounts
      ├── Organization policies
      ├── Billing responsibility
      └── Organization-wide configuration
```

AWS documentation states that the management account is responsible for paying charges incurred by member accounts. citeturn0search0turn0search2

---

# 6. Member Accounts

Other accounts inside the organization are member accounts.

Example:

```text
Management Account
       │
       ├── Development Account
       ├── Staging Account
       ├── Production Account
       ├── Security Account
       └── Logging Account
```

Member accounts remain separate AWS accounts.

This is one of the biggest benefits of multi-account architecture:

```text
Separate account
      ↓
Separate resource/security boundary
      ↓
Central organization governance
```

---

# 7. Organization Root

AWS Organizations creates one root for the organization.

Think:

```text
Organization
      │
     Root
      │
      ├── OU
      ├── OU
      └── Account
```

Important:

```text
AWS Organization Root
```

is **not the same thing as**:

```text
AWS account root user
```

Do not confuse them.

---

# 8. Organization Root vs Root User

## AWS Account Root User

```text
Root User
   ↓
Identity
   ↓
Complete account access
```

## AWS Organizations Root

```text
Organization Root
   ↓
Top-level node in organization hierarchy
   ↓
Contains OUs / accounts
```

These are completely different concepts.

---

# 9. Organizational Unit — OU

An Organizational Unit is a container used to group AWS accounts.

Example:

```text
Workloads OU
│
├── Development
├── Staging
└── Production
```

More realistically:

```text
Workloads OU
│
├── Development Account
├── Staging Account
└── Production Account
```

OUs can also contain other OUs. citeturn0search10

---

# 10. Why OUs Matter

Suppose you have:

```text
Development
Staging
Production
```

You may want:

```text
Development
   ↓
More flexible controls

Production
   ↓
Stricter controls
```

You can group accounts into OUs and apply organization-level controls to the appropriate OU.

AWS states that policies attached to an OU can be inherited by accounts within that OU. citeturn0search10

---

# 11. Example Company Organization

Use this as your learning model:

```text
Organization
│
├── Management Account
│
├── Security OU
│   │
│   ├── Security Account
│   │
│   └── Log Archive Account
│
├── Infrastructure OU
│   │
│   └── Network Account
│
└── Workloads OU
    │
    ├── Development Account
    │
    ├── Staging Account
    │
    └── Production Account
```

This is a conceptual architecture.

The exact OU structure should depend on the organization's security, operational, compliance, and business requirements.

---

# 12. OU Design

Do not create OUs randomly.

Ask:

```text
Why are these accounts grouped together?

Do they require the same policies?

Do they have the same security requirements?

Do they have similar operational requirements?

Do they have similar compliance requirements?

Should they share the same SCP guardrails?
```

The purpose of an OU is not simply:

```text
Folder for accounts
```

It is also a governance boundary for controls that apply to the accounts inside it.

---

# 13. Example OU Structure

A company might have:

```text
Root
│
├── Security OU
│
├── Infrastructure OU
│
├── Workloads OU
│
│   ├── Production OU
│   │
│   └── NonProduction OU
│       ├── Development
│       └── Staging
│
└── Sandbox OU
```

This allows different policies to be applied to different groups of accounts.

---

# 14. SCP — Service Control Policy

SCP means:

```text
Service Control Policy
```

An SCP is an organization-level authorization guardrail.

Most important rule:

> **SCPs do not grant permissions.**

Instead:

```text
SCP
 ↓
Defines the maximum permissions available
for principals in applicable member accounts
```

AWS documents SCPs as policies that control the maximum available permissions for IAM users and roles in member accounts. citeturn0search5turn0search9

---

# 15. IAM Policy vs SCP

This distinction must become automatic.

## IAM Policy

Question:

```text
What is this identity allowed to do?
```

Example:

```text
Developer Role
    ↓
Allow:
ec2:DescribeInstances
s3:GetObject
```

---

## SCP

Question:

```text
What permissions can principals in this
account have at maximum?
```

Example:

```text
Production Account
    ↓
SCP
    ↓
Deny actions in unapproved Regions
```

The SCP does not itself grant:

```text
ec2:DescribeInstances
```

It acts as a guardrail on what permissions can be used in the account.

---

# 16. Critical SCP Mental Model

Remember:

```text
IAM Policy
    ↓
Can grant permissions


SCP
    ↓
Does NOT grant permissions
    ↓
Sets organization-level maximum / guardrails
```

Example:

```text
IAM Policy
    ↓
Allow s3:*
```

but:

```text
SCP
    ↓
Deny s3:DeleteBucket
```

Result:

```text
s3:GetObject
    ↓
Potentially allowed

s3:DeleteBucket
    ↓
Denied
```

An SCP does not replace IAM permissions.

---

# 17. SCP + IAM Together

Think:

```text
IAM Policy
      ↓
"What does this identity request permission to do?"
      │
      ↓
SCP
      ↓
"Is this action permitted within the
organization's maximum boundary?"
      │
      ↓
Final authorization evaluation
```

A user generally needs an applicable permission grant, while an SCP can prevent that permission from being usable.

---

# 18. SCP Example — Region Guardrail

Company wants to restrict workloads to:

```text
ap-south-1
```

Conceptually:

```text
Organization
      ↓
Workloads OU
      ↓
SCP
      ↓
Restrict selected services/actions
to approved Regions
```

Then:

```text
Developer
   ↓
IAM allows EC2
   ↓
Request in ap-south-1
   ↓
Potentially allowed

Developer
   ↓
IAM allows EC2
   ↓
Request in unapproved Region
   ↓
SCP may deny
```

Do not deploy a Region-deny SCP blindly. Some AWS services and global operations require careful handling of Regions and exceptions.

---

# 19. SCP Example — Protect Security Services

Company may want to prevent ordinary administrators from disabling critical security controls.

Concept:

```text
Security Controls
       ↓
SCP
       ↓
Deny selected disabling/deletion actions
```

For example, an organization may design controls around:

```text
CloudTrail
GuardDuty
Security Hub
Config
```

The exact actions should be determined from the current AWS service authorization documentation and tested carefully.

---

# 20. SCP Example — Prevent Leaving Organization

Organizations also provides organization-level controls around account membership and governance.

The exact permissions and management mechanisms should be verified in current AWS Organizations documentation before implementing them.

Do not treat an SCP as a universal security solution.

---

# 21. SCP Inheritance

Suppose:

```text
Root
│
└── Workloads OU
    │
    ├── Development Account
    ├── Staging Account
    └── Production Account
```

An SCP attached at the Workloads OU can apply to the accounts under that OU.

Concept:

```text
Workloads OU
      ↓
SCP
      ↓
Inherited by child accounts
```

AWS documents OU policy inheritance as a core Organizations behavior. citeturn0search10

---

# 22. SCP and Organization Root

An SCP attached to the organization root can apply to the member accounts beneath it.

Important:

> SCPs do not apply to the management account in the same way they apply to member accounts; AWS documents the management-account exception.

Therefore, do not assume:

```text
Root SCP
   ↓
Management Account automatically restricted
```

Always verify the current AWS Organizations policy behavior. citeturn0search5

---

# 23. Consolidated Billing

AWS Organizations provides consolidated billing.

Conceptually:

```text
Management Account
        │
        ├── Development
        ├── Staging
        ├── Production
        └── Security
              │
              ↓
       Consolidated Billing
              │
              ↓
        Central payment
```

AWS states that the management account is responsible for paying charges incurred by member accounts and that consolidated billing combines organization account charges. citeturn0search0turn0search3

---

# 24. Why Consolidated Billing Matters

Instead of separately managing:

```text
Account A → Bill A

Account B → Bill B

Account C → Bill C
```

the organization can have:

```text
Organization
      ↓
Consolidated billing
      ↓
Combined organization costs
```

This can make centralized cost tracking easier.

AWS Organizations also supports combining usage across member accounts for billing purposes. citeturn0search0

---

# 25. Important Billing Point

Organizations structure and billing reporting are not the same thing.

Do not assume:

```text
OU
 ↓
Automatic cost report
```

AWS notes that the bill does not directly reflect the OU hierarchy. Cost allocation tags and other billing/cost-management mechanisms can be used to categorize and track costs. citeturn0search2

Therefore:

```text
Organization structure
        +
Account
        +
Tags
        +
Cost allocation
        ↓
Cost visibility
```

---

# 26. Cross-Account Access

One of the most important Organizations concepts is:

```text
Account A
    ↓
Principal
    ↓
AssumeRole
    ↓
Account B
    ↓
Role
    ↓
Temporary credentials
    ↓
Resources
```

Organizations itself does not automatically give users permission to access every account.

You still need an authorization mechanism.

A common mechanism is:

```text
IAM Role
+
STS AssumeRole
```

---

# 27. Cross-Account Example

Company has:

```text
Development Account
Production Account
```

Developer normally works in:

```text
Development
```

For approved production troubleshooting:

```text
Developer
   ↓
Controlled access
   ↓
Production Role
   ↓
STS AssumeRole
   ↓
Temporary credentials
   ↓
Production resource
```

This allows access to be temporary and role-based.

---

# 28. Cross-Account + IAM Identity Center

A modern workforce model can be:

```text
Employee
    ↓
IAM Identity Center
    ↓
Group
    ↓
Permission Set
    ↓
AWS Account
```

Example:

```text
Developer Group
       ↓
Developer Permission Set
       ↓
Development Account
```

and perhaps:

```text
Security Group
       ↓
Security Permission Set
       ↓
Security Account
       ↓
Other authorized accounts
```

AWS recommends using an organization instance of IAM Identity Center with AWS Organizations for centralized capabilities. citeturn0search4

---

# 29. IAM Identity Center + Organizations

Mental model:

```text
                 AWS ORGANIZATION
                         │
                         ↓
              IAM Identity Center
                         │
                  Users / Groups
                         │
                  Permission Sets
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
      Dev Account    Staging Account  Prod Account
```

This lets organizations centrally manage workforce access across multiple AWS accounts.

AWS documents the integration between IAM Identity Center and AWS Organizations and recommends an organization instance for centralized management. citeturn0search4

---

# 30. Trusted Access

AWS services can integrate with Organizations through trusted access.

Concept:

```text
AWS Organizations
        │
        ↓
Trusted Service
        │
        ↓
Organization-wide integration
```

IAM Identity Center is one service that integrates with Organizations.

AWS documents that IAM Identity Center requires trusted access with AWS Organizations for organization-level operation. citeturn0search8

---

# 31. Delegated Administration

The management account does not necessarily need to perform every task directly.

AWS Organizations supports delegated administration for supported AWS services.

Concept:

```text
Management Account
       │
       ↓
Delegate administration
       │
       ↓
Security / Audit / Specialized Account
       │
       ↓
Manage supported service centrally
```

This is useful for separating:

```text
Management
Security
Audit
Operations
```

and reducing dependence on the management account.

---

# 32. Management Account Security

The management account is extremely sensitive.

Treat it differently from ordinary workload accounts.

Avoid using it for:

```text
Application workloads
Development
Random testing
General EC2 servers
General databases
```

Instead think:

```text
Management Account
       ↓
Organization management
Billing
Governance
Central administration
```

Keep its workload footprint minimal.

---

# 33. Security Account

A company can have:

```text
Security Account
```

used for security-related services and centralized security operations.

Conceptually:

```text
Security OU
   │
   ├── Security Account
   └── Log Archive Account
```

The exact architecture depends on the organization's requirements and may use services such as AWS Security Hub, GuardDuty, CloudTrail, Config, and centralized logging.

---

# 34. Log Archive Account

A separate account can be used for centralized log storage.

Concept:

```text
AWS Accounts
    │
    ├── Dev
    ├── Staging
    ├── Production
    └── Security
            │
            ↓
       Central Logs
            ↓
      Log Archive Account
```

This helps separate audit/log data from workload accounts.

The exact implementation should be designed using AWS logging/security architecture guidance.

---

# 35. Infrastructure Account

Example:

```text
Infrastructure OU
       │
       └── Network Account
```

This account can conceptually contain centrally managed infrastructure such as:

```text
Networking
Shared services
DNS
Connectivity
```

The exact boundaries depend on the company architecture.

---

# 36. Workload Accounts

Workloads can be separated:

```text
Workloads OU
│
├── Development
├── Staging
└── Production
```

This creates strong account-level boundaries.

For example:

```text
Development Account
    ↓
Development resources

Production Account
    ↓
Production resources
```

This is generally easier to govern than trying to separate every environment only through IAM policies inside one account.

---

# 37. Why Multi-Account Architecture Matters

Instead of:

```text
ONE ACCOUNT
│
├── dev
├── staging
├── production
├── security
└── logs
```

you can have:

```text
ORGANIZATION
│
├── Management
├── Security
├── Logging
├── Network
├── Development
├── Staging
└── Production
```

The account boundary itself becomes a major security and operational boundary.

---

# 38. Organization Feature Sets

AWS Organizations supports feature sets including:

```text
All Features
Consolidated Billing
```

AWS currently recommends the **All features** option for full Organizations capabilities. New organizations are created with all features enabled by default. citeturn0search5turn0search7

---

# 39. All Features

With:

```text
All Features
```

you can use broader Organizations capabilities, including organization-level policies and integrations with AWS services.

This is the normal model to understand for company environments.

---

# 40. Consolidated Billing Feature Set

The consolidated-billing-only feature set provides shared billing functionality but does not provide the broader governance capabilities of the full Organizations feature set.

AWS documents that the consolidated-billing feature set does not include the more advanced organizational policy and service integration capabilities. citeturn0search5

For Phase 2, focus primarily on:

```text
All Features
```

---

# 41. Organization Account Lifecycle

Learn the lifecycle:

```text
Create / Invite Account
        ↓
Join Organization
        ↓
Place Account in OU
        ↓
Apply Governance
        ↓
Assign Access
        ↓
Deploy Resources
        ↓
Monitor
        ↓
Move / Suspend / Remove
```

Account lifecycle management becomes important as the company grows.

---

# 42. Creating Member Accounts

Organizations can create new AWS accounts or invite existing AWS accounts to join.

Concept:

```text
Management Account
        ↓
Create Account
        OR
Invite Existing Account
        ↓
Member Account
```

AWS Organizations provides tutorials for creating an organization, inviting member accounts, creating OUs, and applying SCPs. citeturn0search1turn0search12

---

# 43. OrganizationAccountAccessRole

When AWS Organizations creates a member account directly, AWS automatically creates a role named by default:

```text
OrganizationAccountAccessRole
```

This role can provide administrative access from the management account to the member account, subject to the current AWS behavior and account setup.

AWS documents this default role for accounts created directly through Organizations. citeturn0search17

Do not assume this role is created for accounts provisioned through every other account-creation mechanism.

---

# 44. Important Account Boundary

Understand:

```text
Management Account
       │
       ├── Organization controls
       │
       └── Member Accounts
               │
               ├── Own resources
               ├── Own IAM
               ├── Own service configuration
               └── Organization guardrails
```

An AWS Organization does not turn all accounts into one giant AWS account.

They remain separate accounts.

---

# 45. Organization Policy Types

For Phase 2, focus primarily on:

```text
Service Control Policies (SCPs)
```

But know that Organizations supports other policy/control mechanisms and integrations.

Do not assume every organization policy works exactly like an IAM identity policy.

Always identify:

```text
Policy type
Where it attaches
What it controls
Which accounts it affects
Whether it grants or restricts
```

---

# 46. SCP Design Principle

Never start with:

```text
"Deny everything."
```

Start with:

```text
What should the organization prevent?
```

Examples:

```text
Prevent use of unapproved Regions

Prevent certain high-risk actions

Protect security configurations

Protect logging configurations

Restrict specific services where required
```

Then test carefully.

---

# 47. SCP Allow List vs Deny List

There are two broad strategies.

## Deny List

```text
Allow most things
       +
Deny dangerous/unapproved actions
```

## Allow List

```text
Allow only approved services/actions
       +
Everything else remains unavailable
```

AWS's Organizations tutorial demonstrates both styles in controlled examples. citeturn0search12

Do not choose a strategy purely because it sounds safer.

Consider:

```text
Company requirements
Operational complexity
AWS service growth
Exceptions
Break-glass access
Security requirements
```

---

# 48. SCP Testing

Never test a new organization-wide SCP directly against production first.

Use:

```text
Learning Account
        ↓
Test OU
        ↓
Test Account
        ↓
Test SCP
        ↓
Test allowed actions
        ↓
Test denied actions
```

Example:

```text
Test SCP
   ↓
Deny selected EC2 action
```

Then test:

```text
EC2 action
   ↓
Expected Deny
```

and unrelated actions:

```text
S3 action
   ↓
Expected behavior
```

---

# 49. SCP Troubleshooting

If a user suddenly receives:

```text
AccessDenied
```

after moving their account into an OU, check:

```text
[ ] Which OU is the account in?

[ ] Which SCPs are attached to the OU?

[ ] Which SCPs are attached to the root?

[ ] Which SCPs are attached directly to the account?

[ ] Is there an explicit Deny?

[ ] Does the IAM policy allow the action?

[ ] Is the resource policy relevant?

[ ] Is a permissions boundary involved?

[ ] Is a session policy involved?

[ ] Is the management-account exception relevant?
```

Organizations becomes part of the IAM troubleshooting process.

---

# 50. IAM + SCP Troubleshooting Example

Suppose:

```text
Developer IAM Policy
```

contains:

```text
Allow:
ec2:StartInstances
```

But:

```text
Production OU SCP
```

contains:

```text
Deny:
ec2:StartInstances
```

Result:

```text
IAM
 ↓
Allows

SCP
 ↓
Denies

Final
 ↓
DENY
```

This is why:

```text
"I have an IAM Allow"
```

does not automatically mean:

```text
"I can perform the action."
```

---

# 51. Organization + IAM Identity Center

Company model:

```text
                    Organization
                         │
                         │
                IAM Identity Center
                         │
              ┌──────────┴──────────┐
              │                     │
           Groups                Permission Sets
              │                     │
              └──────────┬──────────┘
                         ↓
                   AWS Accounts
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
        Dev           Staging         Production
```

Example:

```text
Developers
    ↓
Developer Permission Set
    ↓
Development Account
```

Security:

```text
Security-Team
    ↓
Security Permission Set
    ↓
Security Account
```

---

# 52. Company Simulation

Build this architecture on paper first:

```text
Organization
│
├── Management Account
│
├── Security OU
│   ├── Security Account
│   └── Log Archive Account
│
├── Infrastructure OU
│   └── Network Account
│
└── Workloads OU
    ├── Development Account
    ├── Staging Account
    └── Production Account
```

Then define access:

```text
Developers
    ↓
Development

DevOps
    ↓
Development + Staging
    ↓
Controlled Production access

Database-Admins
    ↓
Database resources / accounts

Security-Team
    ↓
Security + authorized organization-wide visibility

Billing-Team
    ↓
Billing

ReadOnly
    ↓
Read-only visibility
```

This connects:

```text
Phase 1 IAM
+
Phase 2 Organizations
```

---

# 53. Company Simulation — Governance

Add organization controls:

```text
Organization
      │
      ├── Security OU
      │      ↓
      │   Security SCPs
      │
      ├── Infrastructure OU
      │      ↓
      │   Infrastructure SCPs
      │
      └── Workloads OU
             ↓
          Workload SCPs
```

Example:

```text
Production
   ↓
Strict SCPs

Development
   ↓
Less restrictive operational controls

Sandbox
   ↓
Cost/resource restrictions
```

The exact controls should be based on business and security requirements.

---

# 54. Practical Lab 1 — Open Organizations

In the AWS Console search:

```text
Organizations
```

Open:

```text
AWS Organizations
```

Explore:

```text
AWS accounts
Organizational units
Policies
Services
Settings
```

Do not change production organization settings without authorization.

---

# 55. Practical Lab 2 — Inspect Current Organization

If your learning account is already part of an organization, inspect:

```text
Organization ID

Management Account

Member Accounts

Root

OUs

SCPs

Feature Set
```

Write down the hierarchy.

Example:

```text
Root
│
├── OU
│   ├── Account
│   └── Account
│
└── Account
```

---

# 56. Practical Lab 3 — Create a Learning Organization

If you are using a personal learning account and it is not already part of an organization, you can create a test organization.

AWS provides an official tutorial for this.

Official tutorial:

https://docs.aws.amazon.com/organizations/latest/userguide/orgs_tutorials_basic.html

AWS documents that the basic Organizations tutorial creates an organization, OUs, accounts, and SCPs for testing, and that AWS Organizations itself does not add a service charge. citeturn0search12turn0search2

Important:

```text
Only perform organization/account changes
in an account you control and are authorized to modify.
```

---

# 57. Practical Lab 4 — Create OUs

Create a safe learning structure:

```text
Root
│
├── Security
│
├── Infrastructure
│
└── Workloads
```

Then:

```text
Workloads
│
├── Development
├── Staging
└── Production
```

For a small learning organization, you can also keep the structure simpler.

The purpose is to understand:

```text
Root
 ↓
OU
 ↓
Account
```

---

# 58. Practical Lab 5 — Create Test Accounts

If you have the ability and need for multiple learning accounts, create or invite test accounts.

Example:

```text
Development Account
Staging Account
Production-Simulation Account
Security Account
```

Be aware that while Organizations itself is offered without an additional charge, the AWS resources used in member accounts can incur normal AWS charges. citeturn0search2

Do not create unnecessary resources just because you have multiple accounts.

---

# 59. Practical Lab 6 — Move Accounts Between OUs

Practice:

```text
Development Account
        ↓
Development OU
```

Then:

```text
Development Account
        ↓
Sandbox OU
```

Observe:

```text
OU membership changes
        ↓
Applicable organizational controls may change
```

This teaches why OU placement matters.

---

# 60. Practical Lab 7 — Create a Test SCP

Use a test account.

Create a simple controlled SCP.

For example, create a guardrail that denies a carefully selected action that is safe to test.

Then:

```text
Attach SCP
    ↓
Test Account
    ↓
Try action
    ↓
AccessDenied
```

Then:

```text
Detach SCP
    ↓
Test again
```

This is one of the most useful Phase 2 exercises.

---

# 61. Practical Lab 8 — IAM + SCP

Create:

```text
IAM Role
```

with:

```text
Allow:
specific action
```

Then apply:

```text
SCP
```

with:

```text
Deny:
same action
```

Test.

You should observe:

```text
IAM Policy
    ↓
ALLOW

SCP
    ↓
DENY

Final
    ↓
DENY
```

This connects Phase 1 and Phase 2.

---

# 62. Practical Lab 9 — Consolidated Billing

Open:

```text
Billing and Cost Management
```

and inspect organization-level cost information where available.

Understand:

```text
Management Account
       ↓
Member Account costs
       ↓
Consolidated billing
```

AWS provides consolidated billing so the organization can combine charges from member accounts. citeturn0search0turn0search3

---

# 63. Practical Lab 10 — IAM Identity Center + Organizations

If you have a safe test organization:

```text
Organization
      ↓
IAM Identity Center
      ↓
Group
      ↓
Permission Set
      ↓
Development Account
```

Test:

```text
Sign in
   ↓
Select AWS account
   ↓
Select permission set / role
   ↓
Access AWS Console
```

AWS recommends the organization instance of IAM Identity Center for centralized account access. citeturn0search4

---

# 64. Practical Lab 11 — Cross-Account Access

Use two safe learning accounts:

```text
Account A
Development

Account B
Testing
```

Create:

```text
Role in Account B
```

Configure its trust policy to allow the intended principal/account.

Then:

```text
Account A
    ↓
STS AssumeRole
    ↓
Account B Role
    ↓
Temporary credentials
    ↓
Test resource
```

Verify:

```text
Allowed action
Denied action
```

---

# 65. Practical Lab 12 — Organization Architecture Diagram

Draw this yourself:

```text
                     AWS ORGANIZATION
                            │
                  ┌─────────┴─────────┐
                  │                   │
             Management            Member
               Account             Accounts
                  │                   │
                  │          ┌────────┼────────┐
                  │          │        │        │
                  │        Security  Infra   Workloads
                  │          │        │        │
                  │          │        │    ┌───┼────┐
                  │          │        │    │   │    │
                  │          │        │   Dev Stage Prod
                  │          │        │
                  └──────────┴────────┴────────────
                               │
                              SCP
                               │
                        Organization Guardrails
```

If you can draw this from memory, your Organizations mental model is becoming solid.

---

# 66. Practical Lab 13 — Access Matrix

Create this table for your simulated company:

| Team | Development | Staging | Production | Security | Billing |
|---|---|---|---|---|---|
| Developers | Yes | Limited | No / controlled | No | No |
| DevOps | Yes | Yes | Controlled | Limited | No |
| Database-Admins | Yes | Yes | Database scope | No | No |
| Security-Team | Inspect | Inspect | Inspect | Yes | No |
| Billing-Team | No | No | No | No | Yes |
| ReadOnly | Read | Read | Read | Read | As required |
| AWS-Administrators | Administrative | Administrative | Administrative | Administrative | Administrative |

This is a design exercise, not a production permission policy.

---

# 67. Practical Lab 14 — OU Policy Mapping

Create:

```text
OU
 ↓
Purpose
 ↓
Required guardrails
```

Example:

```text
Security OU
    ↓
Protect security services

Infrastructure OU
    ↓
Protect shared networking

Workloads OU
    ↓
Workload-specific controls

Sandbox OU
    ↓
Cost/resource controls
```

Then determine which SCPs should apply to each OU.

---

# 68. Practical Lab 15 — SCP Troubleshooting

Create an intentional denial.

Example:

```text
Developer
    ↓
IAM Allow
    ↓
S3 action

SCP
    ↓
Explicit Deny
```

Then run the request.

Receive:

```text
AccessDenied
```

Now troubleshoot using:

```text
1. Principal
2. Account
3. OU
4. IAM Policy
5. SCP
6. Resource Policy
7. Permissions Boundary
8. Session Policy
9. Conditions
```

This should become a normal debugging workflow.

---

# 69. Organizations + IAM Mental Model

Remember:

```text
IAM
│
├── Users
├── Groups
├── Roles
├── Policies
└── Permissions
        │
        ↓
   Identity Access
```

Organizations:

```text
AWS Organizations
│
├── Accounts
├── OUs
├── SCPs
├── Billing
├── Cross-account governance
└── Service integrations
        │
        ↓
   Organization Governance
```

Together:

```text
IAM
  +
Organizations
  ↓
Company AWS Governance
```

---

# 70. IAM Policy vs SCP — Final Comparison

| Concept | IAM Policy | SCP |
|---|---|---|
| Main purpose | Grant/control permissions for an identity | Organization-level permission guardrail |
| Applies to | Users, groups, roles, and resource policies depending on type | Organization root, OUs, member accounts |
| Grants permissions? | Identity policies can grant permissions | No |
| Limits permissions? | Can deny / constrain depending on policy type | Yes |
| Main question | "What can this identity do?" | "What can principals in this account do at maximum?" |
| Main scope | Identity/resource | Organization/account/OU governance |
| Explicit Deny | Yes | Yes |
| Requires Organizations? | No | Yes |

AWS documents SCPs as maximum-permission controls and IAM policies as permission controls, so keep these models separate. citeturn0search1turn0search5

---

# 71. What NOT to Do

Avoid:

```text
❌ Put every account directly under the root forever

❌ Create dozens of meaningless OUs

❌ Use SCPs without testing

❌ Assume SCPs grant permissions

❌ Put all workloads in the management account

❌ Give everyone access to every account

❌ Treat Organizations as one giant AWS account

❌ Use root credentials for normal administration

❌ Apply a deny-all SCP to production without testing

❌ Forget the management-account behavior

❌ Assume an OU automatically gives users access

❌ Assume joining an Organization automatically grants cross-account access
```

---

# 72. Phase 2 Learning Order

Follow this order:

```text
1. Why multi-account architecture?
          ↓
2. AWS Organization
          ↓
3. Management Account
          ↓
4. Member Accounts
          ↓
5. Organization Root
          ↓
6. Organization Root vs Root User
          ↓
7. Organizational Units
          ↓
8. OU hierarchy
          ↓
9. SCP
          ↓
10. IAM Policy vs SCP
          ↓
11. SCP inheritance
          ↓
12. SCP testing
          ↓
13. Consolidated Billing
          ↓
14. Cross-account access
          ↓
15. IAM Identity Center integration
          ↓
16. Trusted Access
          ↓
17. Delegated Administration
          ↓
18. Account lifecycle
          ↓
19. Security / Log Archive / Infrastructure accounts
          ↓
20. Company organization architecture
```

---

# 73. Official AWS Documentation

## AWS Organizations — Main Documentation

https://docs.aws.amazon.com/organizations/

---

## What is AWS Organizations?

https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html

Learn:

```text
Organizations
Accounts
Governance
Billing
Policies
```

---

## Getting Started with AWS Organizations

https://docs.aws.amazon.com/organizations/latest/userguide/orgs_getting-started.html

AWS's official getting-started guide covers creating/configuring an organization and working with member accounts and OUs. citeturn0search1

---

## AWS Organizations Concepts

https://docs.aws.amazon.com/organizations/latest/userguide/orgs_getting-started_concepts.html

Learn:

```text
Organization
Root
Account
OU
Policies
Feature sets
```

---

## Creating an Organization

https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_org_create.html

Learn how to create an organization from the AWS Console and understand the available feature sets. citeturn0search7

---

## Creating and Configuring an Organization Tutorial

https://docs.aws.amazon.com/organizations/latest/userguide/orgs_tutorials_basic.html

This is the main hands-on tutorial for Phase 2.

It covers:

```text
Create Organization
        ↓
Create OUs
        ↓
Create / invite accounts
        ↓
Create SCPs
        ↓
Attach SCPs
        ↓
Test policies
```

AWS explicitly provides this as a step-by-step Organizations tutorial. citeturn0search12

---

## Organizational Units

https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_ous.html

Learn:

```text
OUs
Nested OUs
Account placement
Policy inheritance
```

---

## Creating an OU

https://docs.aws.amazon.com/organizations/latest/userguide/create_ou.html

---

## Service Control Policies

https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html

Learn:

```text
SCP
Allow
Deny
Inheritance
Root / OU / Account attachment
```

---

## IAM and Organizations

https://docs.aws.amazon.com/organizations/latest/userguide/security_iam_service-with-iam.html

Use this to understand how IAM interacts with AWS Organizations. citeturn0search6

---

## Cross-Account Access

https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_accounts_access.html

Learn how member-account access works and understand the `OrganizationAccountAccessRole` behavior for accounts created directly through Organizations. citeturn0search17

---

## Consolidated Billing

https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/consolidated-billing.html

Learn:

```text
Management Account
Member Accounts
Combined billing
Combined usage
Cost tracking
```

AWS documents consolidated billing as a core Organizations capability. citeturn0search0

---

## Billing Process

https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/useconsolidatedbilling-procedure.html

---

## IAM Identity Center + Organizations

https://docs.aws.amazon.com/singlesignon/latest/userguide/identity-center-and-orgs.html

Learn:

```text
Organizations
        +
IAM Identity Center
        ↓
Centralized workforce access
```

AWS recommends an organization instance of IAM Identity Center for centralized management. citeturn0search4

---

## IAM Identity Center Integration with Organizations

https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-sso.html

Learn:

```text
Trusted Access
Service integration
Organization-wide IAM Identity Center
```

---

# 74. Phase 2 Completion Checklist

Before moving forward, you should understand:

```text
ORGANIZATION
[ ] What AWS Organizations is
[ ] Why companies use multiple AWS accounts
[ ] Management Account
[ ] Member Account
[ ] Organization Root
[ ] Organization Root vs Root User


ORGANIZATIONAL UNITS
[ ] What an OU is
[ ] Why OUs exist
[ ] Nested OUs
[ ] Account placement
[ ] Policy inheritance


SCP
[ ] What SCP means
[ ] SCP does not grant permissions
[ ] SCP acts as a guardrail
[ ] SCP maximum-permission concept
[ ] Explicit Deny
[ ] SCP inheritance
[ ] Root / OU / Account attachment
[ ] Management-account behavior
[ ] SCP testing


BILLING
[ ] Consolidated Billing
[ ] Management account payment responsibility
[ ] Member account costs
[ ] Organization-level cost visibility
[ ] Cost allocation concepts


CROSS-ACCOUNT
[ ] Account A → Account B
[ ] IAM Role
[ ] Trust Policy
[ ] STS AssumeRole
[ ] Temporary Credentials
[ ] Cross-account authorization


IAM IDENTITY CENTER
[ ] Organizations integration
[ ] Users
[ ] Groups
[ ] Permission Sets
[ ] Account Assignments
[ ] Centralized account access
[ ] Trusted Access


ORGANIZATION ARCHITECTURE
[ ] Security OU
[ ] Log Archive Account
[ ] Infrastructure OU
[ ] Network Account
[ ] Workloads OU
[ ] Development Account
[ ] Staging Account
[ ] Production Account
[ ] Management Account
```

---

# 75. Final Phase 2 Mental Model

You should be able to visualize the company environment like this:

```text
                         AWS ORGANIZATION
                                │
                                │
                      MANAGEMENT ACCOUNT
                                │
                                ↓
                       ORGANIZATION ROOT
                                │
              ┌─────────────────┼──────────────────┐
              │                 │                  │
              ↓                 ↓                  ↓
         SECURITY OU      INFRASTRUCTURE OU    WORKLOADS OU
              │                 │                  │
        ┌─────┴─────┐           │          ┌───────┼────────┐
        │           │           │          │       │        │
    Security    Log Archive   Network     Dev    Staging   Prod
     Account      Account      Account    Account Account Account
        │           │           │          │       │        │
        └───────────┴───────────┴──────────┴───────┴────────┘
                                │
                                ↓
                               SCPs
                                │
                       Organization Guardrails
                                │
                                ↓
                     IAM Identity Center
                                │
                         Users / Groups
                                │
                         Permission Sets
                                │
                                ↓
                         AWS Accounts
                                │
                                ↓
                         AWS Resources
```

---

# 76. IAM + Organizations Final Mental Model

Phase 1:

```text
WHO CAN DO WHAT?
        ↓
       IAM
```

Phase 2:

```text
WHAT CAN ACCOUNTS IN THE ORGANIZATION
BE ALLOWED TO DO AT MAXIMUM?
        ↓
   ORGANIZATIONS
        ↓
       SCP
```

Together:

```text
                 COMPANY AWS
                     │
          ┌──────────┴──────────┐
          │                     │
         IAM               ORGANIZATIONS
          │                     │
          ↓                     ↓
      Identity              Accounts
      Roles                 OUs
      Policies              SCPs
      Permissions           Billing
          │                     │
          └──────────┬──────────┘
                     ↓
             COMPANY GOVERNANCE
                     │
                     ↓
              AWS RESOURCES
```

---

# 77. Phase 2 Final Goal

At the end of Phase 2, you should be able to answer:

```text
Why would a company use multiple AWS accounts?

What is an AWS Organization?

What is the Management Account?

What is a Member Account?

What is the Organization Root?

What is an OU?

Why would Production be in a separate account?

What is an SCP?

Does an SCP grant permissions?

What is the difference between an IAM policy and an SCP?

How does an SCP affect an IAM Allow?

What is consolidated billing?

How does cross-account access work?

What is STS AssumeRole?

How does IAM Identity Center work with Organizations?

What is trusted access?

What is delegated administration?

Why should the Management Account be protected?

What is a Security Account?

What is a Log Archive Account?

What is an Infrastructure / Network Account?

How would you structure Development, Staging,
and Production accounts?

How would you troubleshoot AccessDenied
after moving an account into a different OU?
```

If you can answer those questions and complete the labs, you have the foundation needed for the next major AWS area: **VPC and Networking**.

---

# 78. One Important Rule

Do not think:

```text
IAM = AWS security
```

Think:

```text
IAM
 ↓
Identity and authorization

Organizations
 ↓
Multi-account governance

SCP
 ↓
Organization guardrails

IAM Identity Center
 ↓
Centralized workforce access

Account Boundary
 ↓
Strong workload / environment separation

Billing
 ↓
Centralized financial management
```

The combination is what creates a scalable company AWS environment.

---

# 79. Phase 2 Learning Cycle

Use the same AWS learning method:

```text
LEARN
  ↓
DESIGN
  ↓
BUILD
  ↓
TEST
  ↓
BREAK
  ↓
TROUBLESHOOT
  ↓
SECURE
  ↓
MONITOR
  ↓
DOCUMENT
```

For Organizations specifically:

```text
Learn Account
      ↓
Learn OU
      ↓
Learn SCP
      ↓
Create Test Organization
      ↓
Create Test OU
      ↓
Create Test Account
      ↓
Attach Test SCP
      ↓
Test Allow / Deny
      ↓
Connect IAM Identity Center
      ↓
Test Cross-Account Access
      ↓
Understand Billing
```

**Do not move to the next phase until you can explain the Organization → OU → Account → SCP model without looking at your notes.**
