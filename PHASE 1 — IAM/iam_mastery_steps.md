# PHASE 1 — IAM

> **Goal:** Master the AWS authorization model.
>
> Do not stop at:
>
> > "IAM creates users."
>
> You should be able to look at an AWS access request and reason about:
>
> **Who is making the request → what action are they requesting → what resource are they accessing → which policies apply → what conditions apply → whether the final decision is Allow or Deny.**
>
> AWS evaluates applicable policies and request context. Requests are denied by default, and an explicit `Deny` overrides an `Allow`. citeturn0search0turn0search1

---

# 1. What You Need to Learn

Learn these IAM concepts:

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

You should eventually understand how all of these participate in authorization.

---

# 2. IAM Big Picture

Think about IAM like this:

```text
                         IAM
                          │
             ┌────────────┼─────────────┐
             │            │             │
           Users        Groups         Roles
             │            │             │
             └────────────┼─────────────┘
                          ↓
                       Policies
                          ↓
                  Authorization
                          ↓
                    AWS Resources
```

But there is an important addition:

```text
Principal
   ↓
Request
   ↓
Request Context
   ↓
Applicable Policies
   ↓
Policy Evaluation
   ↓
Allow / Deny
```

AWS's policy evaluation uses the request context and applicable policy types such as identity-based policies, resource-based policies, permissions boundaries, Organizations SCPs, and session policies. citeturn0search1

---

# 3. Authentication vs Authorization

This is one of the first things you must understand.

## Authentication

Authentication answers:

> **Who are you?**

Examples:

```text
Username + password
MFA
SSO
Federation
Temporary credentials
```

---

## Authorization

Authorization answers:

> **What are you allowed to do?**

Example:

```text
Developer
   ↓
Can:
   ec2:DescribeInstances
   ecr:GetAuthorizationToken
   s3:GetObject

Cannot:
   iam:CreateUser
   organizations:CreateAccount
   rds:DeleteDBInstance
```

Mental model:

```text
Authentication
      ↓
Who are you?
      ↓
Principal
      ↓
Authorization
      ↓
What can you do?
      ↓
Policy evaluation
      ↓
Allow / Deny
```

---

# 4. Principal

A **principal** is the identity that makes a request to AWS.

Depending on the situation, a principal can represent things such as:

```text
IAM User
IAM Role
Federated User / Session
AWS Account
AWS Service
Other supported principal types
```

Example:

```text
Developer
    ↓
Assumes IAM Role
    ↓
Temporary role session
    ↓
Makes S3 API request
```

The important question is:

> **Who is making this AWS request?**

---

# 5. IAM User

An IAM user is an IAM identity created in an AWS account.

Historically, organizations often created one IAM user for every employee:

```text
Alice → IAM User
Bob   → IAM User
John  → IAM User
```

However, for company workforce access, AWS generally recommends centralized/federated human access such as IAM Identity Center rather than creating long-lived IAM users and access keys for everyone.

Therefore, learn IAM users thoroughly because they are part of IAM, but do not assume:

```text
1 employee = 1 IAM user
```

is the preferred modern company architecture.

---

# 6. IAM Group

An IAM group is a collection of IAM users.

Example:

```text
Developers
│
├── Alice
├── Bob
└── Charlie
```

You can attach identity-based policies to a group so that users in the group receive those permissions.

Example:

```text
Developers
     ↓
S3 Read
EC2 Read
CloudWatch Read
     ↓
Users
```

Important:

> Groups contain users. IAM roles are not members of IAM groups.

---

# 7. IAM Role

An IAM role is an identity that can be assumed and provides temporary credentials.

Roles are extremely important in AWS.

Examples:

```text
EC2
  ↓
IAM Role
  ↓
S3 Access
```

or:

```text
Developer
  ↓
AssumeRole
  ↓
Temporary Credentials
  ↓
Production Account
```

or:

```text
GitHub Actions
  ↓
Federation / OIDC
  ↓
AWS IAM Role
  ↓
Deploy Application
```

Roles are central to:

- AWS workloads
- cross-account access
- federation
- temporary access
- CI/CD
- service-to-service permissions

---

# 8. Role = Two Different Policy Ideas

When learning roles, separate these two concepts:

```text
ROLE
│
├── Trust Policy
│
└── Permissions Policy
```

### Trust Policy

Answers:

> **Who can assume this role?**

Example:

```text
Developer Role
     ↓
Trust Policy
     ↓
Who is allowed to assume it?
```

### Permissions Policy

Answers:

> **What can the role do after it is assumed?**

Example:

```text
Developer Role
     ↓
Permissions Policy
     ↓
EC2 / S3 / ECR / CloudWatch
```

This distinction is critical.

---

# 9. Trust Policy

A role's trust policy defines which principals are trusted to assume the role.

Conceptually:

```text
Principal
   ↓
Can assume?
   ↓
IAM Role
   ↓
Temporary session
```

Example concept:

```json
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:root"
  },
  "Action": "sts:AssumeRole"
}
```

Do not blindly copy this into a production environment.

The exact trust relationship should be designed according to the intended principal and security requirements.

---

# 10. Permission Policy

A role also needs permissions describing what the role can do.

Example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::company-data/*"
    }
  ]
}
```

This says, conceptually:

```text
Role
 ↓
Allowed to
 ↓
s3:GetObject
 ↓
Objects inside company-data bucket
```

Trust policy:

```text
Who can assume me?
```

Permissions policy:

```text
What can I do?
```

Remember this difference.

---

# 11. Policy

A policy is a JSON document that defines permissions or controls access.

Common policy elements include:

```text
Effect
Action
Resource
Principal
Condition
```

Not every policy type uses every element.

For example:

- Identity-based policies generally use `Effect`, `Action`, `Resource`, and optionally `Condition`.
- Resource-based policies commonly include `Principal`.
- Trust policies use a `Principal` because they define who can assume a role.

---

# 12. Permission

A permission describes an allowed or denied operation against a resource under specified conditions.

Example:

```text
s3:GetObject
```

means:

```text
Read an S3 object
```

Another example:

```text
ec2:StartInstances
```

means:

```text
Start EC2 instances
```

IAM actions generally follow:

```text
service:Action
```

Examples:

```text
s3:GetObject
s3:PutObject

ec2:StartInstances
ec2:StopInstances

rds:DescribeDBInstances

logs:CreateLogGroup
```

---

# 13. Identity-Based Policy

An identity-based policy is attached to an IAM identity such as:

```text
User
Group
Role
```

It defines permissions for that identity.

Example:

```text
Developer Role
      ↓
Identity-based Policy
      ↓
Allow:
  ec2:DescribeInstances
  s3:GetObject
  ecr:GetAuthorizationToken
```

AWS documents identity-based policies as policies attached to IAM identities that grant permissions to users and roles.

---

# 14. Resource-Based Policy

A resource-based policy is attached to a resource.

Examples:

```text
S3 Bucket Policy
SQS Queue Policy
SNS Topic Policy
KMS Key Policy
```

Conceptually:

```text
Resource
   ↓
Resource-Based Policy
   ↓
Principal
   ↓
Allowed Action
```

Example:

```text
S3 Bucket
   ↓
Bucket Policy
   ↓
Allow specific role
   ↓
s3:GetObject
```

Resource-based policies commonly specify the `Principal` that receives access. citeturn0search10

---

# 15. Identity-Based vs Resource-Based Policy

Remember:

```text
Identity-Based
     ↓
Attached to identity
     ↓
"What can this identity do?"

Resource-Based
     ↓
Attached to resource
     ↓
"Who can access this resource?"
```

Example:

```text
IAM Role
   │
   └── Identity Policy
           ↓
       S3:GetObject


S3 Bucket
   │
   └── Bucket Policy
           ↓
       Principal = Role
```

Both policy types can participate in authorization.

Within an account, AWS's exact evaluation behavior depends on the principal and policy types involved, so do not reduce the model to simply "one policy wins."

---

# 16. ARN

Before becoming good at IAM policies, learn ARNs.

ARN means:

```text
Amazon Resource Name
```

General shape:

```text
arn:partition:service:region:account-id:resource
```

Example:

```text
arn:aws:s3:::company-data
```

S3 object:

```text
arn:aws:s3:::company-data/*
```

Example EC2-related ARN:

```text
arn:aws:ec2:ap-south-1:111122223333:instance/i-1234567890abcdef0
```

You should learn how to identify:

```text
service
region
account
resource
```

ARN knowledge is essential for writing precise policies.

---

# 17. IAM Policy Structure

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

Understand every part:

```text
Version
   ↓
Policy language version

Statement
   ↓
One or more policy rules

Effect
   ↓
Allow / Deny

Action
   ↓
What operation?

Resource
   ↓
Which AWS resource?
```

---

# 18. Effect

Two important values:

```text
Allow
Deny
```

Example:

```json
"Effect": "Allow"
```

means the statement grants permission when applicable.

Example:

```json
"Effect": "Deny"
```

means the statement explicitly denies access when applicable.

Important:

```text
Explicit Deny
      ↓
Overrides Allow
```

AWS documents explicit deny as overriding applicable allows. citeturn0search0

---

# 19. Action

`Action` defines what API operation is controlled.

Examples:

```text
s3:GetObject
s3:PutObject
s3:DeleteObject

ec2:StartInstances
ec2:StopInstances
ec2:DescribeInstances

rds:DescribeDBInstances
```

You should learn to read:

```text
service:action
```

Example:

```text
s3:GetObject
```

means:

```text
Service = S3
Action  = GetObject
```

---

# 20. Resource

`Resource` identifies what AWS resource the statement applies to.

Example:

```json
"Resource": "arn:aws:s3:::company-data/*"
```

This means objects under:

```text
company-data
```

The exact ARN format is service-specific.

Therefore, when writing policies, always check the service's IAM documentation for the correct resource ARN format.

---

# 21. Principal

`Principal` identifies who is allowed or denied in a resource-based policy or trust policy.

Example:

```json
"Principal": {
  "AWS": "arn:aws:iam::111122223333:role/DeveloperRole"
}
```

Conceptually:

```text
Principal
    ↓
DeveloperRole
    ↓
Can access this resource
```

Do not use broad principals such as:

```json
"Principal": "*"
```

unless the intended access model genuinely requires public/broad access and the security implications are understood.

---

# 22. Condition

`Condition` adds constraints to a policy.

Conceptually:

```text
Allow
   +
Condition
   ↓
Allow only when condition matches
```

Example:

```json
"Condition": {
  "StringEquals": {
    "aws:RequestedRegion": "ap-south-1"
  }
}
```

This can restrict the policy based on request context.

---

# 23. Important Condition Keys

Learn these:

```text
aws:PrincipalArn

aws:PrincipalOrgId

aws:SourceIp

aws:RequestedRegion

aws:MultiFactorAuthPresent

aws:ResourceTag/*
```

These keys allow policies to make decisions based on request context.

---

# 24. `aws:PrincipalArn`

Can be used to make access decisions based on the ARN of the principal.

Example concept:

```text
Principal
   ↓
Which role/user is making request?
   ↓
aws:PrincipalArn
```

Be careful with the exact semantics and ARN forms used in a policy.

---

# 25. `aws:PrincipalOrgId`

Useful in multi-account AWS Organizations environments.

Conceptually:

```text
Request
   ↓
Principal belongs to Organization?
   ↓
aws:PrincipalOrgId
   ↓
Allow / Deny
```

This can help restrict access to principals belonging to a specific AWS Organization.

---

# 26. `aws:SourceIp`

Can restrict requests based on source IP information.

Concept:

```text
Request
   ↓
Source IP
   ↓
Condition
   ↓
Allow / Deny
```

Example use case:

```text
Allow only requests from approved network ranges.
```

Do not assume IP restrictions are always sufficient security; understand the request path and service behavior.

---

# 27. `aws:RequestedRegion`

Can restrict actions to selected AWS Regions.

Concept:

```text
Allowed:
ap-south-1

Not allowed:
us-east-1
```

Example use case:

```text
Company only operates in approved Regions.
```

---

# 28. `aws:MultiFactorAuthPresent`

Can be used in policies to require MFA-related conditions for supported request scenarios.

Concept:

```text
Request
   ↓
MFA context
   ↓
Condition
   ↓
Allow / Deny
```

Be careful when applying MFA conditions because behavior differs between long-term credentials and temporary credentials.

---

# 29. `aws:ResourceTag`

Can be used to control access based on resource tags for supported actions/services.

Concept:

```text
Resource
   ↓
Environment=dev
   ↓
Policy condition
   ↓
Developer can manage it
```

Example concept:

```text
Allow
ec2:StopInstances

Condition:
Environment=dev
```

The exact supported condition keys and actions must be checked in the relevant service documentation.

---

# 30. Policy Evaluation

This is one of the most important topics in Phase 1.

Basic mental model:

```text
Request
   ↓
Authentication
   ↓
Request Context
   ↓
Applicable Policies
   ↓
Explicit Deny?
   │
   ├── YES → DENY
   │
   └── NO
        ↓
     Explicit Allow?
        │
        ├── YES → Continue evaluation
        │
        └── NO → Implicit DENY
```

AWS states that requests are denied by default and an explicit deny overrides an allow. Other policy types can constrain an allow. citeturn0search0turn0search5

---

# 31. Explicit Deny vs Implicit Deny

This distinction is critical.

## Implicit Deny

No applicable allow exists.

Example:

```text
Developer
   ↓
s3:DeleteObject
   ↓
No Allow
   ↓
Implicit Deny
```

---

## Explicit Deny

A policy explicitly says:

```text
Deny
```

Example:

```text
Developer
   ↓
Allow s3:*
   +
Deny s3:DeleteObject
   ↓
s3:DeleteObject = DENIED
```

Remember:

```text
Explicit Deny
      ↓
Wins over Allow
```

---

# 32. Permissions Boundary

A permissions boundary defines the maximum permissions that an IAM user or role can receive from identity-based policies.

Think:

```text
Identity Policy
       ↓
   "What I request"
       │
       ▼
Permissions Boundary
       ↓
   "Maximum I can receive"
```

Effective permissions are constrained by the applicable policies, including the identity policy and permissions boundary. citeturn0search4

---

# 33. Permissions Boundary Example

Suppose:

```text
Developer Identity Policy
```

allows:

```text
ec2:*
s3:*
iam:CreateRole
```

But the permissions boundary allows only:

```text
ec2:*
s3:GetObject
s3:PutObject
```

Then the developer cannot simply use the identity policy to obtain unrestricted IAM permissions.

Mental model:

```text
Identity Policy
       ∩
Permissions Boundary
       ↓
Effective permissions
```

This is a simplified model; resource-based policies and other policy types can affect the final decision.

---

# 34. Service Control Policy — SCP

An SCP is an AWS Organizations policy that defines the maximum available permissions for principals in member accounts.

Think:

```text
AWS Organization
       ↓
OU / Account
       ↓
SCP
       ↓
Maximum permissions
```

Example:

```text
Company policy:

Deny use of
certain Regions
```

Even if a user has an identity policy allowing an action, an applicable SCP can prevent that action.

AWS documents SCPs as maximum-permission controls for IAM users and roles in organization accounts. citeturn0search1turn0search3

---

# 35. Permissions Boundary vs SCP

Remember:

```text
Permissions Boundary
        ↓
Limits an IAM user/role

SCP
        ↓
Limits permissions available
to principals in an AWS Organization
account/OU
```

Conceptually:

```text
Identity Policy
      ↓
Permissions Boundary
      ↓
SCP
      ↓
Effective permissions
```

This is a learning model, not a literal universal evaluation order.

AWS evaluates the applicable policy types according to its authorization model. citeturn0search0turn0search4

---

# 36. Session Policy

A session policy is an advanced policy that can restrict permissions for a temporary session.

Conceptually:

```text
IAM Role
   ↓
Identity Policy
   ↓
Temporary Session
   +
Session Policy
   ↓
More restricted session
```

AWS describes session policies as policies passed when creating temporary sessions for roles or federated users. citeturn0search1

---

# 37. Session Policy Mental Model

Suppose:

```text
Role permissions:

S3 Get
S3 Put
S3 Delete
```

A temporary session is created with a session policy allowing only:

```text
S3 Get
```

Conceptually:

```text
Role permissions
       ∩
Session policy
       ↓
Session permissions
```

Session policies cannot be used as a mechanism to grant more permissions than the underlying identity permissions allow.

---

# 38. STS — AWS Security Token Service

STS is used to obtain temporary AWS security credentials.

Temporary credentials contain:

```text
Access Key ID
Secret Access Key
Session Token
```

Conceptually:

```text
Principal
   ↓
STS
   ↓
Temporary Credentials
   ↓
AWS API Requests
```

STS is extremely important for:

- IAM role assumption
- cross-account access
- federation
- temporary credentials
- workload identity
- CI/CD authentication

---

# 39. AssumeRole

One of the most important STS operations is:

```text
sts:AssumeRole
```

Concept:

```text
Developer
    ↓
STS AssumeRole
    ↓
IAM Role
    ↓
Temporary Credentials
    ↓
AWS APIs
```

Two questions must be answered:

```text
1. Can this principal assume the role?
      ↓
   Trust Policy

2. What can the role do?
      ↓
   Permissions Policy
```

---

# 40. Cross-Account Role Assumption

Example:

```text
Development Account
        │
        │ Developer
        ↓
       STS
        │
        │ AssumeRole
        ↓
Production Account
        │
        ↓
ProductionRole
        │
        ↓
Temporary Credentials
```

For cross-account access, AWS evaluates the policies in the relevant accounts; the request is allowed only when the required policy evaluations allow it. citeturn0search2

This is a major company IAM pattern.

---

# 41. Cross-Account Example

Suppose:

```text
Account A
Development
```

and:

```text
Account B
Production
```

Developer exists in:

```text
Account A
```

They need temporary access to:

```text
Account B
```

You can conceptually use:

```text
Developer
    ↓
Permission to AssumeRole
    ↓
ProductionRole
    ↓
Trust policy allows Developer
    ↓
STS
    ↓
Temporary credentials
    ↓
Production resources
```

The role's permissions define what the assumed role can do.

---

# 42. Company IAM Structure

For the company simulation, create the following conceptual access groups/roles:

```text
AWS-Administrators

Developers

DevOps

Database-Admins

Security-Team

ReadOnly

Billing-Team
```

Do not automatically make all of these full IAM users.

For modern workforce access, think in terms of:

```text
Human
  ↓
IAM Identity Center
  ↓
Group
  ↓
Permission Set
  ↓
AWS Account
```

and for workloads:

```text
AWS Service / Application
  ↓
IAM Role
  ↓
Temporary Credentials
  ↓
AWS API
```

---

# 43. Developer Access Model

Developer should be able to access:

```text
EC2
ECR
CloudWatch
S3 application bucket
```

Conceptually:

```text
Developer
    ↓
Developer Permission Set / Role
    ↓
Allowed:
    EC2
    ECR
    CloudWatch
    S3 application bucket
```

Developer should not administer:

```text
Billing
IAM
Organizations
Production database deletion
```

---

# 44. Developer Policy Design

Do not give:

```text
AdministratorAccess
```

just because the person is a developer.

Instead ask:

```text
What actions does the developer actually need?
```

Example:

```text
EC2:
  describe
  start
  stop

ECR:
  pull
  push

CloudWatch:
  read logs
  read metrics

S3:
  GetObject
  PutObject
```

Then determine the exact actions and resource scope.

This is the beginning of least-privilege design.

---

# 45. Database Administrator

Database administrator access:

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

Conceptual model:

```text
DBA
 ↓
DBA Permission Set / Role
 ↓
RDS
CloudWatch
Secrets Manager
```

Avoid giving:

```text
AdministratorAccess
```

when a narrower policy can perform the job.

---

# 46. Security Team

Security team can inspect:

```text
CloudTrail
GuardDuty
Security Hub
IAM
Config
```

Conceptually:

```text
Security Team
      ↓
Security Permission Set
      ↓
Read / investigate
      ↓
Security services
```

Separate:

```text
Read / Investigate
```

from:

```text
Modify / Administer
```

where possible.

---

# 47. ReadOnly

Create a conceptual role/permission set:

```text
ReadOnly
```

Purpose:

```text
Can inspect AWS resources
Cannot modify infrastructure
```

This is useful for:

- auditors
- managers
- developers needing visibility
- support personnel
- troubleshooting

Exact permissions should match the intended use.

---

# 48. Billing Team

Create a conceptual:

```text
Billing-Team
```

Do not automatically grant:

```text
AdministratorAccess
```

Billing access should be separated from infrastructure administration when company requirements call for that separation.

The exact billing permissions depend on the account and billing configuration.

---

# 49. Least Privilege

Least privilege means giving a principal only the permissions needed to perform its job.

Bad:

```text
Developer
   ↓
AdministratorAccess
```

Better:

```text
Developer
   ↓
Required services
   ↓
Required actions
   ↓
Required resources
   ↓
Required conditions
```

Think:

```text
WHO
 ↓
WHAT ACTION
 ↓
WHICH RESOURCE
 ↓
UNDER WHICH CONDITIONS
```

---

# 50. IAM Policy Mastery

You should be able to read this:

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

and immediately explain:

```text
Effect:
Allow

Action:
s3:GetObject

Resource:
Objects inside company-data

Principal:
Not specified here because this is an identity-based policy
```

Then ask:

```text
Who has this policy?

What other policies apply?

Is there an SCP?

Is there a permissions boundary?

Is this a role session?

Is there a resource-based policy?

Are there conditions?

```

That is IAM thinking.

---

# 51. IAM Policy Exercise 1

Create a policy that conceptually allows:

```text
S3 GetObject
```

on:

```text
company-data
```

Objects only.

Expected resource:

```text
arn:aws:s3:::company-data/*
```

Do not give:

```text
s3:*
```

---

# 52. IAM Policy Exercise 2

Create a policy that allows:

```text
EC2 DescribeInstances
```

Ask:

```text
What Resource should I use?

Does this action support resource-level permissions?
```

This teaches an important IAM concept:

> Not every AWS action supports resource-level permissions in the same way.

Always check the service authorization reference.

---

# 53. IAM Policy Exercise 3

Create:

```text
Allow:
s3:*
```

Then add:

```text
Deny:
s3:DeleteObject
```

Test:

```text
GetObject
PutObject
DeleteObject
```

Expected concept:

```text
GetObject       → Allow
PutObject       → Allow
DeleteObject    → Deny
```

The explicit deny overrides the allow.

---

# 54. IAM Policy Exercise 4 — Region Restriction

Create a policy condition conceptually restricting selected actions to:

```text
ap-south-1
```

Use:

```text
aws:RequestedRegion
```

Then reason about:

```text
Request in ap-south-1
    ↓
Condition matches
    ↓
Potentially allowed

Request in another Region
    ↓
Condition does not match
    ↓
Potentially denied
```

Always check service-specific behavior before using region conditions in production.

---

# 55. IAM Policy Exercise 5 — Tag-Based Access

Concept:

```text
Environment=dev
```

Developer can manage:

```text
dev resources
```

but not:

```text
production resources
```

Mental model:

```text
Developer
   ↓
EC2 action
   ↓
Resource Tag
   ↓
Environment=dev?
   ↓
Allow / Deny
```

Learn the exact supported condition keys for each service/action before implementing this.

---

# 56. Policy Debugging

When you receive:

```text
AccessDenied
```

do not randomly add:

```text
AdministratorAccess
```

Instead investigate:

```text
Who is the principal?

What action was requested?

What resource was requested?

Which Region?

Which account?

Which policies apply?

Is there an explicit Deny?

Is there an SCP?

Is there a permissions boundary?

Is there a session policy?

Is there a resource-based policy?

Does the role trust policy allow assumption?

Are conditions matching?

Is the resource ARN correct?
```

This is how an AWS administrator should troubleshoot IAM.

---

# 57. IAM Policy Simulator / Testing

Learn to test policies before using them in production.

Look for AWS IAM policy testing/simulation capabilities in the IAM Console and documentation.

Your goal:

```text
Policy
   ↓
Test request
   ↓
Expected decision
   ↓
Allow / Deny
```

Do not rely only on reading JSON.

Test it.

---

# 58. IAM Access Analyzer

IAM Access Analyzer helps identify access issues and validate policies.

It can help with:

```text
External access
Unused access
Policy validation
Custom policy checks
Policy generation
Internal access analysis
```

AWS documents Access Analyzer as a tool for identifying resources shared with external entities, validating IAM policies, identifying unused access, and generating policies from access activity. citeturn0search13turn0search6

---

# 59. Access Analyzer Practice

Open:

```text
IAM
  ↓
Access Analyzer
```

Explore the available analyzers/features in your account.

Understand:

```text
Analyzer
   ↓
Policies / Resources
   ↓
Findings
   ↓
Investigate
   ↓
Fix unintended access
```

Do not blindly delete or modify access based on a finding.

First understand:

```text
Who needs this access?
Why does it exist?
Should it be internal?
Should it be external?
```

---

# 60. IAM Identity Center

Now move from traditional IAM identities to modern workforce access.

Mental model:

```text
IAM Identity Center
        ↓
Users / Groups
        ↓
Permission Sets
        ↓
AWS Accounts
        ↓
Temporary AWS access
```

Permission sets define the level of access users and groups receive in AWS accounts and can be provisioned to one or more accounts. citeturn0search8

---

# 61. IAM Identity Center Users

Conceptually:

```text
Company
  ↓
Employees
  ↓
IAM Identity Center Users
```

Examples:

```text
alice
bob
charlie
```

In larger organizations, identities may come from an external identity provider rather than being managed directly in IAM Identity Center.

---

# 62. IAM Identity Center Groups

Create groups such as:

```text
Developers

DevOps

Database-Admins

Security-Team

ReadOnly

Billing-Team

AWS-Administrators
```

Then:

```text
Group
   ↓
Permission Set
   ↓
AWS Account
```

This is much easier to manage than assigning individual permissions repeatedly.

---

# 63. Permission Sets

A permission set defines the access level users/groups receive when accessing an AWS account through IAM Identity Center.

Example:

```text
DeveloperPermissionSet
```

could provide permissions for:

```text
EC2
ECR
CloudWatch
Application S3
```

Another:

```text
ReadOnlyPermissionSet
```

could provide read-only access.

AWS documentation:

https://docs.aws.amazon.com/singlesignon/latest/userguide/permissionsets.html

---

# 64. Account Assignment

An account assignment connects:

```text
User / Group
      +
Permission Set
      +
AWS Account
```

Example:

```text
Developers
    +
DeveloperPermissionSet
    +
Development Account
```

AWS IAM Identity Center calls this an account assignment. citeturn0search9

---

# 65. Company IAM Identity Center Structure

Use this simulation:

```text
                     IAM Identity Center
                             │
              ┌──────────────┼──────────────┐
              │              │              │
         Developers        DevOps       Security
              │              │              │
              ↓              ↓              ↓
       Developer PS     DevOps PS      Security PS
              │              │              │
              ↓              ↓              ↓
       Development       Development      Security
         Account           Account         Account
```

If you only have one AWS account for learning, simulate the multi-account structure conceptually.

Do not create unnecessary AWS accounts just for the exercise.

---

# 66. IAM Identity Center MFA

Learn:

```text
User
 ↓
IAM Identity Center
 ↓
MFA
 ↓
AWS Account
```

MFA adds an additional authentication factor.

Your company should have a defined MFA policy for workforce access.

---

# 67. Federation

Federation allows users to authenticate through an external identity system and then obtain AWS access.

Conceptually:

```text
Employee
   ↓
Corporate Identity Provider
   ↓
Federation
   ↓
IAM Identity Center / AWS
   ↓
Permission Set
   ↓
AWS Account
```

Examples of external identity providers include enterprise identity platforms that support standards such as SAML or OIDC, depending on the integration.

For Phase 1, understand the architecture.

Deep identity-provider integration can be learned later.

---

# 68. IAM Identity Center Practice

If you have a safe learning account and are authorized to configure it:

```text
1. Open IAM Identity Center
2. Inspect its status
3. Understand identity source
4. Inspect/create a learning group
5. Create a learning permission set
6. Assign the group to an AWS account
7. Sign in using the assigned access
8. Test allowed actions
9. Test denied actions
```

Do not make changes to a company production identity system without authorization.

---

# 69. IAM Identity Center Permission Set Example

Conceptual:

```text
Permission Set:
Developer

Allowed:
  EC2
  ECR
  CloudWatch
  Application S3

Not allowed:
  IAM administration
  Organizations administration
  Billing administration
  Production DB deletion
```

The exact permissions should be implemented through carefully scoped policies.

---

# 70. Company Access Matrix

Build a matrix like this:

| Team | EC2 | ECR | S3 | RDS | CloudWatch | IAM | Organizations | Billing | Security |
|---|---|---|---|---|---|---|---|---|---|
| Developers | Limited | Yes | App bucket | Limited | Yes | No | No | No | Limited |
| DevOps | Yes | Yes | Yes | Operational | Yes | Limited | Limited | No | Limited |
| Database-Admins | No/limited | No/limited | Limited | Yes | Yes | No | No | No | Limited |
| Security-Team | Inspect | Inspect | Inspect as needed | Inspect | Yes | Inspect | Inspect | No | Yes |
| ReadOnly | Read | Read | Read | Read | Read | Read | Read | Depending on scope | Read |
| Billing-Team | No | No | No | No | No | No | No | Yes | No |
| AWS-Administrators | Broad administrative access | Broad | Broad | Broad | Broad | Broad | Broad | Broad | Broad |

**Important:** This table is a planning model, not a ready-made production policy. Each permission should be translated into specific actions/resources/conditions according to your organization's requirements.

---

# 71. IAM Design Questions

For every role or permission set, ask:

```text
Who is this for?

What job does this person perform?

Which AWS services do they need?

Which actions do they need?

Which resources do they need?

Which Regions?

Which environments?

Do they need read or write access?

Do they need delete access?

Should production be separated?

Should MFA be required?

Should access be temporary?

Should access be cross-account?

Can conditions reduce the scope?

Can tags restrict the resources?
```

This is how you move from:

```text
"I know IAM"
```

to:

```text
"I can design IAM."
```

---

# 72. IAM Security Principles

Learn these principles:

```text
Least privilege

Temporary credentials

Role-based access

Federated workforce access

MFA

No unnecessary long-lived access keys

Separate production access

Separate administrative access

Explicitly scope resources

Use conditions where appropriate

Monitor and review access

Remove unused access
```

AWS IAM security guidance emphasizes human federation, temporary credentials, MFA, workload roles, least privilege, and removing unnecessary access. Use the AWS IAM security best-practices documentation as a reference while practicing. 

---

# 73. Long-Lived Access Keys

Avoid designing company access around:

```text
Employee
   ↓
IAM User
   ↓
Access Key
   ↓
Permanent credentials
```

Prefer, where supported:

```text
Human
   ↓
IAM Identity Center / Federation
   ↓
Temporary credentials
```

For workloads:

```text
Application / AWS Service
   ↓
IAM Role
   ↓
Temporary credentials
```

Long-lived credentials create additional credential-management risk.

---

# 74. Workload IAM

IAM is not only about human users.

You also need to understand:

```text
Application
   ↓
IAM Role
   ↓
AWS API
```

Example:

```text
EC2
  ↓
EC2 Instance Role
  ↓
S3:GetObject
```

The application does not need an engineer to manually put an access key into source code.

This is a major real-world IAM pattern.

---

# 75. Bad Architecture

Avoid:

```text
Developer
   ↓
Access Key
   ↓
Source Code
   ↓
AWS
```

and:

```text
EC2
   ↓
Hard-coded AWS_ACCESS_KEY_ID
   ↓
AWS API
```

Better:

```text
Developer
   ↓
IAM Identity Center / Federation
   ↓
Temporary credentials


EC2
   ↓
IAM Role
   ↓
Temporary credentials
```

---

# 76. IAM + S3 Company Scenario

Suppose:

```text
S3 bucket:
company-application-data
```

Developer needs:

```text
GetObject
PutObject
```

but not:

```text
DeleteBucket
PutBucketPolicy
DeleteObject
```

Design:

```text
Developer
   ↓
Developer Permission Set / Role
   ↓
S3 policy
   ↓
Specific bucket
   ↓
Specific actions
```

Then test:

```text
GetObject       → should work
PutObject       → should work
DeleteObject    → should fail
PutBucketPolicy → should fail
```

---

# 77. IAM + Production Scenario

Suppose:

```text
Developer
```

can manage:

```text
Development EC2
```

but must not manage:

```text
Production EC2
```

Possible controls include:

```text
Separate AWS accounts
+
Permission Sets
+
Resource scoping
+
Conditions/tags where appropriate
+
SCPs
+
Separate production roles
```

Do not depend on one policy mechanism when a stronger account/environment boundary is appropriate.

---

# 78. IAM + Cross-Account Scenario

Company:

```text
Development Account
Staging Account
Production Account
Security Account
```

Access:

```text
Developer
   ↓
Development account
```

Production:

```text
Developer
   ↓
Request / controlled access
   ↓
Production Role
   ↓
Temporary credentials
```

Security team:

```text
Security account
   ↓
Security roles
   ↓
Inspect organization resources
```

This is a common direction for multi-account IAM architecture.

---

# 79. Troubleshooting Checklist

When AWS returns:

```text
AccessDenied
```

go through this checklist:

```text
[ ] Who is the principal?

[ ] Which AWS account?

[ ] Which Region?

[ ] Which action?

[ ] Which resource?

[ ] Is there an identity-based Allow?

[ ] Is there a resource-based Allow?

[ ] Is there an explicit Deny?

[ ] Is there an SCP?

[ ] Is there a permissions boundary?

[ ] Is there a session policy?

[ ] Is the role trust policy correct?

[ ] Are Conditions matching?

[ ] Is the ARN correct?

[ ] Is the action supported on the specified resource?

[ ] Is the request actually using the credentials you think it is?
```

This checklist should become automatic.

---

# 80. Phase 1 Practical Labs

## Lab 1 — IAM Console Exploration

Open:

```text
AWS Console
  ↓
IAM
```

Explore:

```text
Users
User groups
Roles
Policies
Access Analyzer
Account settings
Credential reports / access information where available
```

Do not create unnecessary permanent users.

---

# 81. Lab 2 — Create a Learning IAM User

Only do this in a safe personal learning account if needed.

Create a temporary test user to understand:

```text
User
 ↓
Group
 ↓
Policy
 ↓
Permission
```

Do not use this as your company's primary workforce architecture.

The purpose is learning the IAM mechanics.

---

# 82. Lab 3 — Group Permissions

Create:

```text
Developers
```

Create a test user.

Add the user to:

```text
Developers
```

Attach a narrowly scoped learning policy.

Test:

```text
Allowed action
Denied action
```

Then remove the test setup when finished if it is no longer needed.

---

# 83. Lab 4 — IAM Role

Create:

```text
DeveloperLearningRole
```

Understand:

```text
Trust Policy
+
Permissions Policy
```

Test assuming the role.

Then inspect the resulting identity/session.

---

# 84. Lab 5 — STS AssumeRole

Practice:

```text
Principal
   ↓
sts:AssumeRole
   ↓
IAM Role
   ↓
Temporary credentials
   ↓
AWS API
```

Understand the difference between:

```text
Permanent identity
```

and:

```text
Temporary role session
```

---

# 85. Lab 6 — Explicit Deny

Create a controlled test policy:

```text
Allow:
s3:GetObject
s3:PutObject

Deny:
s3:PutObject
```

Test:

```text
GetObject
PutObject
```

Observe:

```text
GetObject → Allow
PutObject → Deny
```

This teaches the explicit-deny rule.

---

# 86. Lab 7 — Resource-Based Policy

Use a test S3 bucket.

Understand:

```text
IAM Role
      +
S3 Bucket Policy
      ↓
Access decision
```

Create a controlled bucket policy.

Then test access.

Remove the test policy after learning.

---

# 87. Lab 8 — Permissions Boundary

Create a test IAM role or user.

Attach:

```text
Identity Policy
```

Then attach:

```text
Permissions Boundary
```

Make the boundary narrower than the identity policy.

Test an action outside the boundary.

Observe the resulting denial.

This is one of the best ways to understand the difference between:

```text
Policy says I can
```

and:

```text
Boundary says I cannot exceed this maximum
```

---

# 88. Lab 9 — IAM Policy Simulator / Validation

Take a policy and test different actions:

```text
s3:GetObject
s3:PutObject
s3:DeleteObject
```

Change:

```text
Action
Resource
Condition
```

and observe the authorization result.

---

# 89. Lab 10 — Access Analyzer

Create or inspect a test resource.

Use:

```text
IAM Access Analyzer
```

Investigate:

```text
External access
Unused access
Policy validation
```

Learn to answer:

```text
Why does this principal have this access?
```

---

# 90. Lab 11 — IAM Identity Center

If you have a safe account and appropriate authorization:

```text
IAM Identity Center
       ↓
Create / inspect user
       ↓
Create group
       ↓
Create permission set
       ↓
Assign group
       ↓
AWS account
       ↓
Sign in
       ↓
Test permissions
```

Permission sets are the core mechanism used to define account access for users/groups in IAM Identity Center. citeturn0search8turn0search11

---

# 91. Lab 12 — Developer Permission Set

Create a conceptual or test:

```text
Developer
```

Permission set.

Allow only the services/actions needed for:

```text
EC2
ECR
CloudWatch
Application S3
```

Test:

```text
EC2 operation       → expected result
ECR operation       → expected result
CloudWatch operation → expected result
S3 application      → expected result
IAM admin           → should fail
Organizations       → should fail
Billing admin       → should fail
```

---

# 92. Lab 13 — Database Administrator Permission Set

Create:

```text
Database-Admins
```

with controlled access to:

```text
RDS
CloudWatch
Secrets Manager
```

Test:

```text
RDS access
CloudWatch access
Secrets Manager access
IAM administration
Organizations administration
Billing administration
```

Verify that the permissions match your intended design.

---

# 93. Lab 14 — Security Team Permission Set

Create:

```text
Security-Team
```

with controlled access to inspect:

```text
CloudTrail
GuardDuty
Security Hub
IAM
Config
```

Separate:

```text
Read
```

from:

```text
Modify
```

where possible.

---

# 94. Lab 15 — Cross-Account Access

If you have multiple safe learning accounts:

```text
Account A
Development

Account B
Test / Production simulation
```

Create:

```text
Role in Account B
```

Trust:

```text
Account A / approved principal
```

Allow:

```text
Specific permissions
```

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

Cross-account requests require the relevant policy evaluations in the trusted/requesting and trusting/resource accounts to allow the request. citeturn0search2

---

# 95. IAM Policy Reading Exercise

For every policy you encounter, write this:

```text
WHO?
 ↓
Principal / identity

WHAT?
 ↓
Action

WHICH RESOURCE?
 ↓
Resource / ARN

WHEN?
 ↓
Condition

WHAT DECISION?
 ↓
Allow / Deny

WHAT LIMITS IT?
 ↓
Boundary / SCP / Session Policy /
Resource Policy / other applicable controls
```

This is your IAM analysis framework.

---

# 96. IAM Policy Design Framework

When creating a new permission:

```text
1. Identify the principal
        ↓
2. Identify the required action
        ↓
3. Identify the exact resource
        ↓
4. Identify required conditions
        ↓
5. Check for cross-account requirements
        ↓
6. Check boundaries/SCPs/session policies
        ↓
7. Test the policy
        ↓
8. Monitor the access
        ↓
9. Remove unused permissions
```

---

# 97. What NOT to Do

Avoid these learning habits:

```text
❌ Give AdministratorAccess to everything

❌ Use root for daily work

❌ Put AWS access keys in source code

❌ Put AWS secrets in Git

❌ Give developers IAM:* without a reason

❌ Use Resource="*" when a specific ARN is available

❌ Use Principal="*" casually

❌ Solve every AccessDenied error with AdministratorAccess

❌ Create permanent access keys for every employee

❌ Ignore explicit Deny

❌ Ignore SCPs

❌ Ignore permissions boundaries

❌ Ignore role trust policies
```

---

# 98. Phase 1 Learning Order

Follow this order:

```text
1. Authentication vs Authorization
          ↓
2. Principal
          ↓
3. User
          ↓
4. Group
          ↓
5. Role
          ↓
6. Trust Policy
          ↓
7. Permissions Policy
          ↓
8. ARN
          ↓
9. Identity-Based Policy
          ↓
10. Resource-Based Policy
          ↓
11. Policy Elements
          ↓
12. Conditions
          ↓
13. Policy Evaluation
          ↓
14. Explicit vs Implicit Deny
          ↓
15. Permissions Boundary
          ↓
16. SCP
          ↓
17. Session Policy
          ↓
18. STS
          ↓
19. AssumeRole
          ↓
20. Cross-Account Access
          ↓
21. Access Analyzer
          ↓
22. IAM Identity Center
          ↓
23. Permission Sets
          ↓
24. Account Assignments
          ↓
25. Federation
          ↓
26. Company IAM Architecture
```

---

# 99. Official AWS Documentation

## IAM Documentation

https://docs.aws.amazon.com/iam/

---

## IAM Security Best Practices

https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html

Use this as the security reference while learning IAM.

---

## IAM Policies and Permissions

https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html

Learn:

```text
Identity policies
Resource policies
Permissions boundaries
SCPs
Policy concepts
```

AWS documentation covers these policy types and their relationships. citeturn0search3

---

## IAM Policy Evaluation Logic

https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html

This should be one of your main Phase 1 references.

Learn:

```text
Implicit deny
Explicit deny
Allow
Identity policies
Resource policies
Permissions boundaries
SCPs
Session policies
```

AWS's authorization engine evaluates all applicable policy types and request context; explicit deny overrides allow. citeturn0search0

---

## Request Context

https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic_policy-eval-reqcontext.html

Learn how AWS evaluates information from the request context.

---

## Cross-Account Policy Evaluation

https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic-cross-account.html

Use this when learning:

```text
Account A
   ↓
Account B
   ↓
AssumeRole / resource access
```

---

## Permissions Boundaries

https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html

Learn:

```text
Identity Policy
       ∩
Permissions Boundary
       ↓
Effective permissions
```

---

## IAM Access Analyzer

https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html

Learn:

```text
External access
Unused access
Policy validation
Policy generation
Custom policy checks
```

AWS documents Access Analyzer as a way to identify unintended external access and validate/analyze IAM policies. citeturn0search13

---

## IAM Identity Center

https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html

Learn:

```text
Users
Groups
Permission Sets
Account Assignments
Federation
```

---

## IAM Identity Center Permission Sets

https://docs.aws.amazon.com/singlesignon/latest/userguide/permissionsets.html

Permission sets define the access level users/groups receive in AWS accounts. citeturn0search8

---

## Create Permission Set

https://docs.aws.amazon.com/singlesignon/latest/userguide/howtocreatepermissionset.html

Use this for practical console exercises.

---

# 100. Phase 1 Completion Checklist

Before moving to the next phase, you should be able to explain:

```text
IAM
[ ] What IAM does
[ ] Authentication vs Authorization
[ ] Principal
[ ] User
[ ] Group
[ ] Role
[ ] Policy
[ ] Permission


ROLES
[ ] What an IAM role is
[ ] Trust policy
[ ] Permissions policy
[ ] AssumeRole
[ ] Temporary credentials


POLICIES
[ ] Identity-based policy
[ ] Resource-based policy
[ ] Effect
[ ] Action
[ ] Resource
[ ] Principal
[ ] Condition
[ ] ARN


CONDITIONS
[ ] aws:PrincipalArn
[ ] aws:PrincipalOrgId
[ ] aws:SourceIp
[ ] aws:RequestedRegion
[ ] aws:MultiFactorAuthPresent
[ ] aws:ResourceTag


POLICY EVALUATION
[ ] Implicit deny
[ ] Explicit deny
[ ] Explicit allow
[ ] Explicit Deny overrides Allow
[ ] Permissions boundary
[ ] SCP
[ ] Session policy
[ ] Resource-based policy interactions
[ ] Cross-account evaluation


STS
[ ] What STS does
[ ] Temporary credentials
[ ] AssumeRole
[ ] Cross-account roles


ACCESS ANALYZER
[ ] External access findings
[ ] Unused access
[ ] Policy validation
[ ] Policy generation


IAM IDENTITY CENTER
[ ] Users
[ ] Groups
[ ] Permission Sets
[ ] Account Assignments
[ ] MFA
[ ] Federation
```

---

# 101. Final IAM Test

Do not move forward until you can answer these without copying from documentation:

### Question 1

A developer gets:

```text
AccessDenied
```

What do you check?

Answer should include:

```text
Principal
Action
Resource
Region
Identity policy
Resource policy
Explicit Deny
SCP
Permissions boundary
Session policy
Conditions
Trust policy if role assumption is involved
```

---

### Question 2

What is the difference between:

```text
Trust Policy
```

and:

```text
Permissions Policy
```

Expected mental model:

```text
Trust Policy
    ↓
Who can assume the role?

Permissions Policy
    ↓
What can the role do?
```

---

### Question 3

Why is this dangerous?

```json
{
  "Effect": "Allow",
  "Action": "*",
  "Resource": "*"
}
```

Because it is extremely broad and can grant access to many actions/resources.

You should understand why least privilege is preferable.

---

### Question 4

What happens when:

```text
Allow s3:*
```

and:

```text
Deny s3:DeleteObject
```

both apply?

```text
s3:DeleteObject
       ↓
Explicit Deny
       ↓
DENIED
```

---

### Question 5

What is the difference between:

```text
IAM User
```

and:

```text
IAM Role
```

You should understand:

```text
User
 ↓
Long-lived IAM identity

Role
 ↓
Assumable identity
 ↓
Temporary credentials
```

while also understanding that modern human workforce access is generally better handled through centralized/federated access such as IAM Identity Center.

---

### Question 6

What is the difference between:

```text
Permissions Boundary
```

and:

```text
SCP
```

Expected:

```text
Permissions Boundary
 ↓
Maximum permissions for an IAM user/role

SCP
 ↓
Maximum available permissions for principals
within applicable AWS Organization accounts/OUs
```

---

### Question 7

What is STS?

Expected:

```text
AWS Security Token Service

Used to obtain temporary AWS credentials,
including through role assumption and federation-related flows.
```

---

### Question 8

What is IAM Identity Center?

Expected:

```text
Centralized workforce access
       ↓
Users / Groups
       ↓
Permission Sets
       ↓
AWS Accounts
```

---

# 102. Final Phase 1 Mental Model

You should be able to visualize IAM like this:

```text
                         HUMAN / WORKLOAD
                                │
                                ↓
                             PRINCIPAL
                                │
                                ↓
                              REQUEST
                                │
                ┌───────────────┼────────────────┐
                │               │                │
              Action          Resource         Context
                │               │                │
                │               │         Region / IP /
                │               │         MFA / Tags /
                │               │         Org / etc.
                └───────────────┼────────────────┘
                                ↓
                       POLICY EVALUATION
                                │
          ┌─────────────────────┼─────────────────────┐
          │                     │                     │
   Identity Policy      Resource Policy       Trust Policy
          │                     │                     │
          ├───────────────┬─────┴──────┬──────────────┤
          │               │            │
     Permissions       SCP          Session
      Boundary                       Policy
          │               │            │
          └───────────────┼────────────┘
                          ↓
                    EXPLICIT DENY?
                       │
              ┌────────┴────────┐
             YES                NO
              ↓                  ↓
            DENY          Is there Allow?
                              │
                    ┌─────────┴─────────┐
                   NO                   YES
                    ↓                    ↓
             IMPLICIT DENY       ALLOW if all
                                 applicable limits
                                 are satisfied
```

And for modern company workforce access:

```text
                 EMPLOYEE
                    │
                    ↓
          IAM Identity Center
                    │
              ┌─────┴─────┐
              │           │
            User        Group
                          │
                          ↓
                  Permission Set
                          │
                          ↓
                     AWS Account
                          │
                          ↓
                   Temporary Access
```

For AWS workloads:

```text
              APPLICATION
                   │
                   ↓
               IAM ROLE
                   │
                   ↓
                  STS
                   │
                   ↓
        Temporary Credentials
                   │
                   ↓
              AWS Services
```

---

# 103. Phase 1 Final Goal

At the end of Phase 1, you should be able to look at an AWS authorization problem and reason about it systematically:

```text
WHO?
 ↓
Principal

WHAT?
 ↓
Action

WHERE?
 ↓
Resource

WHEN / UNDER WHAT CONDITIONS?
 ↓
Condition

WHICH POLICIES APPLY?
 ↓
Identity
Resource
Boundary
SCP
Session
Trust

WHAT IS THE FINAL RESULT?
 ↓
Allow / Explicit Deny / Implicit Deny
```

And you should be able to design a company access model such as:

```text
                    COMPANY AWS ACCESS
                            │
              ┌─────────────┴─────────────┐
              │                           │
           HUMAN                       WORKLOAD
              │                           │
              ↓                           ↓
     IAM Identity Center             IAM Roles
              │                           │
           Groups                       STS
              │                           │
       Permission Sets           Temporary Credentials
              │                           │
              └─────────────┬─────────────┘
                            ↓
                       AWS Accounts
                            │
                            ↓
                     AWS Resources
                            │
                            ↓
                  Policy Evaluation
                            │
                            ↓
                       Allow / Deny
```

**Do not move to the next phase until this model is comfortable.**
