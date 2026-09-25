# PHASE 9 — AWS SECURITY

## Goal

This phase is about understanding the security services that answer different questions about an AWS environment.

The most important thing is **not** memorizing service names.

Instead, learn to ask:

```text
What question am I trying to answer?
```

A useful security mental model is:

```text
                         AWS Environment
                               |
          +--------------------+--------------------+
          |                    |                    |
          v                    v                    v
       PREVENT               DETECT              RESPOND
          |                    |                    |
     IAM / KMS /          CloudTrail /         Security Hub
     Secrets / WAF        GuardDuty /          findings
                          Inspector / Macie
          |
          v
       GOVERN
          |
      AWS Config
```

---

# 1. Security Service Map

| Service | Main Question | Main Job |
|---|---|---|
| AWS CloudTrail | Who did what? | API/activity auditing |
| Amazon CloudWatch | What is happening? | Metrics, logs, alarms, observability |
| AWS Config | Is this resource configured correctly? | Configuration/compliance tracking |
| Amazon GuardDuty | Is there suspicious activity? | Threat detection |
| AWS Security Hub | What security findings/posture need attention? | Central security findings and controls |
| Amazon Inspector | Are workloads/images vulnerable? | Vulnerability management |
| Amazon Macie | Is sensitive data exposed or present in S3? | S3 data security/privacy discovery |
| AWS KMS | How are encryption keys controlled? | Key management and cryptographic controls |
| AWS Secrets Manager | Where should application secrets live? | Secret storage and rotation |
| AWS WAF | Which web requests should be allowed/blocked? | Layer 7 web protection |
| AWS Shield | Is my internet-facing application protected from DDoS? | DDoS protection |

The services overlap in a useful way, but they are not interchangeable.

---

# 2. The Five Security Questions

When something goes wrong, think in this order:

```text
1. WHO?
   |
   +-- CloudTrail

2. WHAT IS HAPPENING?
   |
   +-- CloudWatch

3. WHAT IS THE RESOURCE CONFIGURED LIKE?
   |
   +-- AWS Config

4. IS THERE SUSPICIOUS ACTIVITY?
   |
   +-- GuardDuty

5. WHAT SECURITY FINDINGS NEED ATTENTION?
   |
   +-- Security Hub
```

Then:

```text
Is software vulnerable?
    -> Inspector

Is sensitive data in S3?
    -> Macie

How is encryption controlled?
    -> KMS

Where are application secrets?
    -> Secrets Manager

Can malicious web requests be blocked?
    -> WAF

Is the application protected against DDoS?
    -> Shield
```

---

# 3. Security Is Not One Service

A production security architecture is layered.

Example:

```text
Internet
   |
   v
Route 53
   |
   v
CloudFront
   |
   +---- AWS WAF
   |
   +---- AWS Shield
   |
   v
ALB
   |
   v
EC2 / ECS
   |
   +---- CloudWatch
   |
   +---- Inspector
   |
   v
RDS
   |
   +---- KMS encryption
   |
   +---- CloudWatch monitoring

AWS account
   |
   +---- CloudTrail
   +---- AWS Config
   +---- GuardDuty
   +---- Security Hub
   +---- Macie
   +---- KMS
   +---- Secrets Manager
```

The services solve different problems.

---

# PART 1 — CLOUDTRAIL

# 4. CloudTrail

The primary question is:

> **Who did what?**

Examples:

```text
Who deleted this S3 object?

Who changed this Security Group?

Who created this IAM resource?

Who stopped this EC2 instance?

Who changed a bucket policy?

Who modified a KMS key policy?
```

CloudTrail records AWS activity as events.

Conceptually:

```text
AWS API request
      |
      v
CloudTrail
      |
      v
Event
```

An event can contain information such as:

```text
Who
What action
When
From where
Which resource
Whether the request succeeded
```

---

# 5. CloudTrail Event Mental Model

Imagine:

```text
Alice
 |
 | DeleteSecurityGroup
 v
EC2
```

CloudTrail can provide an event showing that API activity.

Think:

```text
Principal
   |
   v
API action
   |
   v
AWS service
   |
   v
Resource
   |
   v
CloudTrail event
```

---

# 6. CloudTrail Management Events

Management events cover control-plane activity.

Examples:

```text
CreateBucket
CreateRole
CreateSecurityGroup
RunInstances
StopInstances
ModifyDBInstance
PutBucketPolicy
```

These events are useful for answering:

```text
Who changed my infrastructure?
```

---

# 7. CloudTrail Data Events

Data events concern operations on data-plane resources.

Examples can include object-level activity such as:

```text
S3 GetObject
S3 PutObject
S3 DeleteObject
```

Data events can be high-volume.

Therefore, production logging design should consider:

```text
What should be logged?
How much data?
How long?
What will it cost?
```

---

# 8. CloudTrail Event History

CloudTrail provides an event history view for recent management events.

You can filter/search by values such as:

```text
Event name
Username / identity
Resource
Time
AWS service
```

Exercise:

Search for:

```text
StopInstances
```

Then inspect:

```text
Who performed it?
When?
Which instance?
Source IP?
AWS Region?
```

---

# 9. CloudTrail Trails

A trail can deliver CloudTrail events to durable storage such as S3.

Architecture:

```text
AWS activity
     |
     v
CloudTrail
     |
     v
Trail
     |
     v
S3
```

This is important for longer-term auditing and investigation.

A company should design retention deliberately.

---

# 10. CloudTrail + S3

A common security architecture is:

```text
Management Account
       |
       v
CloudTrail
       |
       v
Central Log Archive
       |
       v
S3
```

In a multi-account organization:

```text
Account A ----\
Account B -----+--> Organization / centralized audit logging
Account C ----/
```

The goal is to prevent an administrator of one workload from easily erasing the evidence of their own activity.

---

# 11. CloudTrail + CloudWatch

CloudTrail answers:

```text
Who performed the API operation?
```

CloudWatch can help you:

```text
Monitor
Search
Alert
Dashboard
Automate responses
```

Example:

```text
CloudTrail event
     |
     v
CloudWatch / EventBridge
     |
     v
SNS / Lambda / automation
```

---

# 12. CloudTrail Investigation Lab

Perform:

```text
1. Create a test security group.
2. Add a rule.
3. Remove the rule.
4. Stop an EC2 instance.
5. Change an S3 bucket setting.
6. Open CloudTrail.
7. Search for the API events.
```

For each event identify:

```text
Identity
Action
Time
Region
Resource
Source IP
Result
```

---

# PART 2 — CLOUDWATCH

# 13. CloudWatch

The main question is:

> **What is happening to my infrastructure and application?**

CloudWatch provides observability capabilities such as:

```text
Metrics
Logs
Alarms
Dashboards
Log Groups
Logs Insights
Events / event-driven integrations
```

CloudWatch is different from CloudTrail.

```text
CloudTrail
    =
API activity auditing

CloudWatch
    =
operational observability
```

---

# 14. Metrics

A metric is a time-series measurement.

Examples:

```text
CPUUtilization
NetworkIn
NetworkOut
RequestCount
Latency
5XXError
DatabaseConnections
FreeStorageSpace
```

Conceptually:

```text
Metric
 |
 +-- Name
 +-- Value
 +-- Timestamp
 +-- Dimensions
```

---

# 15. Dimensions

Dimensions help identify what a metric belongs to.

Example:

```text
CPUUtilization
InstanceId = i-123456
```

Another:

```text
RequestCount
LoadBalancer = my-alb
TargetGroup = backend
```

Dimensions are important when building alarms and dashboards.

---

# 16. EC2 Monitoring

For EC2 you can inspect metrics such as:

```text
CPU
Network
Disk-related metrics depending on monitoring/agent setup
Status checks
```

Exercise:

Launch an EC2 instance and observe:

```text
CPUUtilization
NetworkIn
NetworkOut
StatusCheckFailed
```

Then generate CPU load and watch the metric change.

---

# 17. Logs

CloudWatch Logs stores log events in log groups and log streams.

Mental model:

```text
Application
   |
   v
Log stream
   |
   v
Log group
```

Example:

```text
/production/api
```

could contain streams for individual application instances/containers.

---

# 18. Application Logs

Your application might produce:

```text
INFO User logged in
WARN Database latency high
ERROR Payment request failed
```

CloudWatch Logs can centralize those logs.

Architecture:

```text
EC2 / ECS / Lambda
       |
       v
CloudWatch Logs
       |
       v
Log Group
```

---

# 19. Log Groups

A log group is a logical container for log streams.

Examples:

```text
/production/api
/production/worker
/production/frontend
```

You should design naming and retention intentionally.

Do not automatically keep everything forever.

---

# 20. Log Retention

Production logging should have a retention strategy.

Example:

```text
Development:
7–30 days

Production:
30–365+ days
```

The correct value depends on:

```text
Security requirements
Compliance
Incident response
Cost
Business requirements
```

---

# 21. CloudWatch Logs Insights

Logs Insights lets you query CloudWatch Logs.

Typical investigation:

```text
Find all 500 errors
Find requests slower than 2 seconds
Find a particular request ID
Find authentication failures
```

Conceptual flow:

```text
Logs
 |
 v
Logs Insights
 |
 v
Query
 |
 v
Results
```

---

# 22. Example Logs Insights Investigation

Suppose your application logs:

```text
status=500
latency=2400
requestId=abc123
```

You can query for:

```text
status = 500
```

and investigate:

```text
timestamp
requestId
endpoint
latency
error
```

This is extremely useful during incidents.

---

# 23. Alarms

An alarm watches a metric against a threshold or other evaluation condition.

Example:

```text
CPU > 80%
```

for a defined evaluation period.

Conceptually:

```text
Metric
  |
  v
Alarm
  |
  +---- OK
  |
  +---- ALARM
  |
  +---- INSUFFICIENT_DATA
```

---

# 24. Alarm Actions

An alarm can integrate with actions/services such as:

```text
SNS notifications
Auto Scaling
other AWS automation
```

Example:

```text
CPU > 80%
     |
     v
CloudWatch Alarm
     |
     v
SNS
     |
     v
Notification
```

---

# 25. Dashboards

A dashboard gives you a visual operational view.

Example:

```text
Production Dashboard

CPU
Memory
Request Count
Latency
5XX
Healthy Targets
Database Connections
Free Storage
```

The goal is:

```text
One screen
   |
   v
Current system health
```

---

# 26. CloudWatch Events / EventBridge

AWS event-driven architectures commonly use Amazon EventBridge.

Conceptually:

```text
AWS event
   |
   v
EventBridge
   |
   +--> Lambda
   +--> SNS
   +--> SQS
   +--> Step Functions
   +--> other targets
```

Examples:

```text
EC2 state changed
Security finding created
Scheduled event occurred
AWS service event occurred
```

For modern AWS architectures, learn EventBridge alongside CloudWatch because event routing is a major operational pattern.

---

# PART 3 — AWS CONFIG

# 27. AWS Config

The question is:

> **Are AWS resources configured according to our rules?**

CloudTrail says:

```text
Who changed the resource?
```

Config says:

```text
What is the resource configuration?
Was it compliant?
How did the configuration change?
```

---

# 28. Config Resource Recording

AWS Config records configuration information about supported resources.

Examples:

```text
EC2
Security Groups
S3
RDS
IAM resources
VPC resources
CloudFront
many other AWS resource types
```

The exact supported resource types depend on AWS service/Region and current Config support.

---

# 29. Config Timeline

Config can help you understand resource configuration over time.

Example:

```text
Security Group
     |
     +-- Monday: SSH from office IP
     |
     +-- Tuesday: SSH from 0.0.0.0/0
     |
     +-- Wednesday: fixed
```

This is valuable for compliance and investigations.

---

# 30. Config Rules

A Config rule evaluates whether a resource meets a condition.

Example:

```text
Rule:
Security groups should not allow unrestricted SSH
```

Possible result:

```text
COMPLIANT
NON_COMPLIANT
```

---

# 31. Example Config Rules

Useful examples:

```text
S3 buckets should block public access

Security groups should not allow unrestricted SSH

RDS should not be publicly accessible

Required tags should exist

Encryption should be enabled

CloudTrail should be enabled
```

Rules can be AWS managed or custom, depending on the use case.

---

# 32. Config vs CloudTrail

This distinction is essential.

### CloudTrail

```text
Who changed the security group?
```

### Config

```text
What does the security group look like now?
Was it compliant?
How did its configuration change?
```

Together:

```text
CloudTrail + Config
```

provide a much stronger investigation capability.

---

# 33. Config + Security Hub

Security Hub can use security controls and findings across the environment.

Config can provide configuration/compliance information needed by many controls.

Conceptually:

```text
AWS Config
    |
    v
Configuration/compliance information
    |
    v
Security Hub
```

Security Hub also integrates with multiple AWS security services.

---

# PART 4 — GUARDDUTY

# 34. GuardDuty

The question is:

> **Are there signs of suspicious or potentially malicious activity?**

GuardDuty is a threat detection service.

It analyzes security-relevant data sources and generates findings.

Current GuardDuty uses foundational data sources such as:

```text
CloudTrail management events
VPC Flow Logs
Route 53 Resolver DNS query logs
```

and additional protection features can analyze other sources when enabled.

---

# 35. GuardDuty Finding

A finding is an indication of potentially suspicious activity.

Conceptual example:

```text
Suspicious API activity
       |
       v
GuardDuty
       |
       v
Finding
```

A finding can contain:

```text
Severity
Resource
Account
Region
Activity
Threat information
Time
```

---

# 36. GuardDuty Is Not CloudTrail

CloudTrail:

```text
"An API call happened."
```

GuardDuty:

```text
"This activity may indicate a security threat."
```

Example:

```text
CloudTrail:
CreateAccessKey happened.

GuardDuty:
The observed activity matches a suspicious pattern.
```

GuardDuty uses security analytics rather than merely showing raw audit events.

---

# 37. GuardDuty Protection Areas

GuardDuty has protection capabilities for different AWS resource/data types.

Depending on Region and configuration, these can include areas such as:

```text
AWS accounts
EC2
S3
EKS
ECS
RDS
Lambda
```

Do not assume every protection capability is automatically enabled in every environment.

Check the GuardDuty console for the protections available to your account and Region.

---

# 38. GuardDuty Multi-Account Architecture

For the company organization from Phase 2:

```text
AWS Organization
       |
       v
Security Account
       |
       v
GuardDuty delegated administrator
       |
       +---- Development
       +---- Staging
       +---- Production
       +---- other accounts
```

This gives the security team centralized visibility.

---

# 39. GuardDuty Lab

Enable GuardDuty in your learning environment.

Study:

```text
Dashboard
Findings
Severity
Resource
Finding type
Suppression/filter concepts
Protection plans
```

Do not intentionally create real malicious activity.

Instead, learn how to use supported sample findings or AWS-provided test mechanisms where available.

---

# PART 5 — SECURITY HUB

# 40. Security Hub

The question is:

> **What is the security posture across my AWS environment?**

Security Hub provides a central view of security findings and security controls.

It can integrate findings from services such as:

```text
GuardDuty
Inspector
Macie
IAM Access Analyzer
AWS Config
and other integrated services
```

---

# 41. Security Hub Mental Model

Think:

```text
GuardDuty
      \
Inspector ----\
Macie ---------+--> Security Hub
Config --------/
IAM Analyzer --/
```

Security Hub is not a replacement for those services.

It helps centralize and organize security findings and posture information.

---

# 42. Security Controls

Security Hub CSPM includes security controls that evaluate AWS resources against security best practices and supported standards.

Example concept:

```text
Control:
S3 buckets should block public access

Result:
PASSED / FAILED
```

The exact controls available depend on the service, Region, enabled standards, and current AWS capabilities.

---

# 43. Security Hub Finding Workflow

Example:

```text
Inspector
   |
   v
Vulnerable EC2 package
   |
   v
Security Hub
   |
   v
Security Team
   |
   v
Remediation
```

Another:

```text
GuardDuty
   |
   v
Suspicious activity
   |
   v
Security Hub
   |
   v
Investigation
```

---

# 44. Security Hub Is Not SIEM

Do not think:

```text
Security Hub = complete SIEM
```

Security Hub is primarily an AWS security posture/finding aggregation and management service.

A company may integrate AWS security findings with a broader SIEM/SOAR system when needed.

---

# PART 6 — AMAZON INSPECTOR

# 45. Amazon Inspector

The question is:

> **Are my workloads or software artifacts vulnerable?**

Inspector is a vulnerability management service.

Current capabilities include scanning areas such as:

```text
EC2
ECR container images
Lambda
```

depending on enabled scanning/configuration.

---

# 46. Inspector vs GuardDuty

This distinction is important.

### Inspector

```text
Vulnerability management
```

Example:

```text
Package has known CVE
```

### GuardDuty

```text
Threat detection
```

Example:

```text
Suspicious activity detected
```

So:

```text
Vulnerability != active threat
```

---

# 47. ECR + Inspector

From Phase 7:

```text
Developer
   |
   v
Docker image
   |
   v
ECR
   |
   v
Inspector
   |
   v
Vulnerability findings
```

This creates a useful security gate before deployment.

Example:

```text
myapp:1.0
   |
   +-- vulnerable OpenSSL package
```

---

# 48. EC2 + Inspector

Inspector can identify vulnerabilities in supported EC2 workloads.

Conceptually:

```text
EC2
 |
 v
Inspector
 |
 v
Package vulnerability
 |
 v
Security Hub
```

---

# PART 7 — AMAZON MACIE

# 49. Amazon Macie

The question is:

> **Where is sensitive data in my S3 environment, and what data-security risks should I investigate?**

Macie focuses on Amazon S3.

It can discover sensitive data using:

```text
Managed data identifiers
Custom data identifiers
Sensitive data discovery
Automated sensitive data discovery
```

---

# 50. Example Sensitive Data

Depending on the configured detection capabilities, sensitive data can include categories such as:

```text
Credentials
Financial information
Personal information
Identity-related information
Secrets
```

Do not treat the finding as proof of a legal classification.

Your organization's policies and applicable laws determine how data must be classified and handled.

---

# 51. Macie Example

Suppose:

```text
S3:
company-data/
```

contains:

```text
customers.csv
employees.csv
backup.csv
```

Macie can help identify objects that appear to contain sensitive information.

Conceptually:

```text
S3
 |
 v
Macie
 |
 +-- sensitive data discovery
 |
 v
Finding
```

---

# 52. Macie vs GuardDuty

### GuardDuty

```text
Threat activity
```

### Macie

```text
Sensitive data in S3
```

Example:

```text
GuardDuty:
Suspicious access pattern

Macie:
Object appears to contain sensitive information
```

---

# PART 8 — AWS KMS

# 53. AWS KMS

The question is:

> **How do we control and audit cryptographic keys used to protect data?**

KMS is a managed key management service.

It is used by many AWS services for encryption.

Examples:

```text
S3
EBS
RDS
Secrets Manager
CloudTrail
and many other AWS services
```

---

# 54. Encryption Mental Model

Think:

```text
Plaintext
   |
   v
Encryption
   |
   v
Ciphertext
```

A cryptographic key controls the encryption/decryption operation.

KMS helps manage the keys and access controls around them.

---

# 55. KMS Key Types

For this phase, understand:

```text
AWS owned keys
AWS managed keys
Customer managed keys
```

The exact capabilities, control, and billing differ.

Customer managed KMS keys give you additional control over:

```text
Key policy
Permissions
Rotation configuration
Lifecycle
Auditing
Cross-account usage design
```

---

# 56. KMS Key Policy

This is extremely important.

KMS keys have key policies.

The key policy is a resource policy and is a primary mechanism for controlling access to a KMS key.

Conceptually:

```text
IAM Policy
    +
KMS Key Policy
    +
Grant where applicable
    |
    v
Can this principal use the KMS key?
```

Unlike ordinary IAM permissions, KMS access cannot be understood correctly by looking only at an IAM policy.

---

# 57. KMS Encryption Example

S3:

```text
Application
    |
    v
S3
    |
    v
KMS
    |
    v
Encrypted object
```

RDS:

```text
Application
    |
    v
RDS
    |
    v
KMS encryption
```

EBS:

```text
EC2
 |
 v
Encrypted EBS
 |
 v
KMS
```

---

# 58. Envelope Encryption

A core concept:

```text
KMS key
   |
   v
Data key
   |
   v
Large data
```

You generally do not send every byte of a large file directly through KMS.

Instead, AWS services commonly use envelope encryption.

Conceptually:

```text
KMS key
   |
   +--> protects data key
              |
              v
         encrypts data
```

This is a critical cryptography concept to understand.

---

# 59. KMS Grants

AWS services can use KMS grants to obtain limited permissions for cryptographic operations.

Think:

```text
KMS key
   |
   +-- Key policy
   |
   +-- IAM policy where applicable
   |
   +-- Grants
```

Do not modify KMS permissions casually.

A bad key-policy change can prevent applications or AWS services from decrypting data.

---

# 60. KMS Region

KMS keys are Regional.

A key in:

```text
ap-south-1
```

is not the same key as a key in:

```text
us-east-1
```

Multi-Region KMS keys are a specialized capability and should be learned separately.

---

# 61. KMS Audit

KMS API activity can be logged through CloudTrail.

Example:

```text
Who used kms:Decrypt?
Who changed the key policy?
Who scheduled deletion?
```

Mental model:

```text
KMS
 |
 v
CloudTrail
 |
 v
Audit
```

---

# PART 9 — SECRETS MANAGER

# 62. AWS Secrets Manager

The question is:

> **Where should applications securely store and retrieve secrets?**

Examples:

```text
Database passwords
API keys
Third-party credentials
Application secrets
Tokens
```

Do not put secrets in:

```text
Git
Dockerfile
source code
AMI
plaintext environment files
public S3 objects
```

---

# 63. Secrets Manager Mental Model

```text
Application
    |
    | IAM role
    v
Secrets Manager
    |
    v
Secret value
```

The application retrieves the secret at runtime.

---

# 64. Example Database Secret

Instead of:

```text
DB_PASSWORD=MyPassword123
```

inside source code:

```text
Application
   |
   v
Secrets Manager
   |
   v
database credential
```

Then:

```text
Application
   |
   v
RDS
```

---

# 65. Secrets Manager Encryption

Secrets Manager encrypts secret values using envelope encryption with AWS KMS keys.

You can use:

```text
aws/secretsmanager
```

or, when appropriate, a customer managed KMS key.

Use customer managed keys when you specifically need additional key-policy/control requirements.

---

# 66. Secrets Rotation

Rotation means:

```text
Old secret
   |
   v
New secret
```

The credential should be updated in both:

```text
Secrets Manager
```

and the system that uses the credential.

Secrets Manager supports managed rotation for supported secrets and Lambda-based rotation for other use cases.

---

# 67. Secrets Manager + RDS

A common architecture:

```text
Application
    |
    v
Secrets Manager
    |
    v
DB credentials
    |
    v
RDS
```

IAM:

```text
EC2/ECS task role
       |
       +-- secretsmanager:GetSecretValue
```

Do not give an application:

```text
secretsmanager:*
```

unless there is a very specific administrative reason.

Use least privilege.

---

# 68. Parameter Store vs Secrets Manager

You learned Parameter Store in Phase 4.1.

Think:

```text
Parameter Store
    |
    +-- configuration
    +-- parameters
    +-- some encrypted values

Secrets Manager
    |
    +-- secrets
    +-- credentials
    +-- rotation
```

There is overlap.

Choose based on:

```text
Secret lifecycle
Rotation requirements
Integration
Cost
Access model
Application requirements
```

---

# PART 10 — AWS WAF

# 69. AWS WAF

The question is:

> **Which web requests should be allowed, blocked, counted, or challenged?**

WAF is a web application firewall.

It operates at the HTTP/HTTPS request layer.

Common AWS integrations include:

```text
CloudFront
Application Load Balancer
API Gateway
other supported resources
```

---

# 70. WAF Mental Model

```text
Internet
   |
   v
CloudFront / ALB
   |
   v
WAF
   |
   +---- Allow
   |
   +---- Block
   |
   +---- Count
   |
   +---- other supported actions
```

The exact processing order depends on the Web ACL/rule configuration.

---

# 71. WAF Web ACL

A Web ACL is the main policy object.

Example:

```text
Web ACL
 |
 +-- Rule 1
 +-- Rule 2
 +-- Rule 3
 +-- Default action
```

---

# 72. WAF Rules

Rules can inspect request properties such as:

```text
IP address
URI path
Headers
Query strings
HTTP method
labels
rate-based conditions
```

Rules can also use managed rule groups.

---

# 73. WAF Managed Rules

AWS and AWS Marketplace providers offer managed rule groups for common attack patterns.

They can help detect/block patterns associated with threats such as:

```text
SQL injection
Cross-site scripting
known malicious request patterns
```

Managed rules still require testing and tuning.

Do not blindly enable every rule and assume the application is secure.

---

# 74. WAF Rate-Based Rules

A rate-based rule can help control excessive request rates from sources.

Conceptually:

```text
Client
 |
 | thousands of requests
 v
WAF
 |
 +-- threshold exceeded
 |
 v
Block / challenge / count
```

This can help against abusive request patterns, but WAF rate limiting is not a replacement for complete DDoS protection or application-level abuse controls.

---

# 75. WAF vs Security Group

Very important:

### Security Group

Works at network/instance interface level.

Example:

```text
TCP 443
```

### WAF

Understands HTTP/HTTPS requests.

Example:

```text
/block suspicious URI
```

So:

```text
Security Group
    =
network access control

WAF
    =
web request filtering
```

---

# 76. WAF vs Shield

### WAF

```text
Application/web request protection
```

### Shield

```text
DDoS protection
```

They can work together.

Example:

```text
Internet
   |
   v
CloudFront
   |
   +-- Shield
   |
   +-- WAF
   |
   v
Application
```

---

# PART 11 — AWS SHIELD

# 77. AWS Shield

The question is:

> **How is my internet-facing AWS application protected against DDoS attacks?**

Shield provides DDoS protection.

There are two important service levels:

```text
Shield Standard
Shield Advanced
```

---

# 78. Shield Standard

Shield Standard provides automatic protection against common network and transport-layer DDoS attacks for AWS customers.

It is integrated into AWS services and provides particular benefits for resources such as:

```text
Route 53
CloudFront
Global Accelerator
```

---

# 79. Shield Advanced

Shield Advanced provides additional DDoS detection, mitigation, and response capabilities.

It is intended for organizations with stronger DDoS protection requirements.

Learn:

```text
Protected resources
DDoS response
Visibility
Support
Cost considerations
Integration with WAF
```

Do not assume Shield Advanced is required for every small application.

---

# 80. Shield + WAF + CloudFront

A common internet-facing architecture is:

```text
User
 |
 v
Route 53
 |
 v
CloudFront
 |
 +---- Shield
 |
 +---- WAF
 |
 v
ALB
 |
 v
Application
```

Each layer has a different purpose:

```text
Route 53
    -> DNS

Shield
    -> DDoS protection

WAF
    -> HTTP request filtering

CloudFront
    -> edge delivery / CDN

ALB
    -> load balancing
```

---

# PART 12 — HOW THE SERVICES WORK TOGETHER

# 81. Complete Security Architecture

```text
                           Internet
                              |
                              v
                           Route 53
                              |
                              v
                         CloudFront
                         /         \
                        /           \
                    Shield           WAF
                                      |
                                      v
                                     ALB
                                      |
                          +-----------+-----------+
                          |                       |
                          v                       v
                        EC2                     ECS
                          |                       |
                          +-----------+-----------+
                                      |
                                      v
                                     RDS
```

Observability/security:

```text
CloudTrail
    |
    +--> API audit

CloudWatch
    |
    +--> metrics
    +--> logs
    +--> alarms

AWS Config
    |
    +--> configuration/compliance

GuardDuty
    |
    +--> threat detection

Inspector
    |
    +--> vulnerability findings

Macie
    |
    +--> S3 sensitive-data findings

Security Hub
    |
    +--> centralized findings/posture
```

Data protection:

```text
KMS
 |
 +--> S3
 +--> EBS
 +--> RDS
 +--> Secrets Manager
```

Secrets:

```text
EC2 / ECS
    |
    v
Secrets Manager
    |
    v
DB password / API key
```

---

# 82. Security Service Relationship Map

Think:

```text
                    AWS ENVIRONMENT
                           |
          +----------------+----------------+
          |                |                |
          v                v                v
        AUDIT           OBSERVE           CONFIG
          |                |                |
      CloudTrail        CloudWatch       AWS Config
          |                |                |
          +----------------+----------------+
                           |
                           v
                       SECURITY
                           |
       +-------------------+-------------------+
       |                   |                   |
       v                   v                   v
   GuardDuty           Inspector            Macie
   threats             vulnerabilities      S3 data
       |                   |                   |
       +-------------------+-------------------+
                           |
                           v
                     Security Hub
                           |
                           v
                       RESPONSE
```

Protection:

```text
Internet
   |
   v
Shield
   |
   v
WAF
   |
   v
Application
```

Encryption/secrets:

```text
KMS
 |
 +---- encryption keys

Secrets Manager
 |
 +---- application secrets
```

---

# PART 13 — SECURITY INVESTIGATION SCENARIOS

# 83. Scenario 1 — Someone Deleted an S3 Object

Question:

```text
Who deleted this object?
```

Start:

```text
CloudTrail
```

Investigate:

```text
DeleteObject
Identity
Time
Bucket
Object key
Source
```

Then:

```text
CloudTrail
    |
    v
Who?
    |
    v
IAM identity / role
```

If object-level data events were not configured/available for the relevant trail setup, you may not have the desired object-level audit record.

This is why logging design must be done before an incident.

---

# 84. Scenario 2 — Security Group Suddenly Changed

Question:

```text
Who changed my security group?
```

Use:

```text
CloudTrail
```

Then inspect the current state/history with:

```text
AWS Config
```

Mental model:

```text
CloudTrail
    |
    +-- Who changed it?

Config
    |
    +-- What changed / what is current?
```

---

# 85. Scenario 3 — EC2 CPU Is 100%

Question:

```text
What is happening?
```

Use:

```text
CloudWatch
```

Check:

```text
CPU
Network
Application logs
Process behavior if available through host tooling
```

If the activity appears suspicious:

```text
GuardDuty
```

can provide a separate threat-detection perspective.

---

# 86. Scenario 4 — EC2 Has Vulnerable Packages

Question:

```text
Does this workload contain known vulnerabilities?
```

Use:

```text
Inspector
```

Then:

```text
Security Hub
```

can help centralize the resulting security finding.

---

# 87. Scenario 5 — S3 Contains Customer Data

Question:

```text
Does this S3 estate contain sensitive data?
```

Use:

```text
Macie
```

Then investigate:

```text
Bucket
Object
Data type
Finding
Access controls
Encryption
Retention
```

---

# 88. Scenario 6 — API Is Being Attacked

Possible layers:

```text
CloudFront
Shield
WAF
ALB
Application
CloudWatch
GuardDuty
```

Investigate:

```text
1. Is traffic reaching the edge?
2. Is WAF blocking requests?
3. Is this DDoS-like traffic?
4. Are backend targets overloaded?
5. Are application errors increasing?
6. Is there suspicious AWS account activity?
```

Use the correct service for each question.

---

# 89. Scenario 7 — Secret Leaked

If an API key is exposed:

```text
1. Revoke/rotate the credential.
2. Update Secrets Manager.
3. Identify where the secret was exposed.
4. Search CloudTrail for relevant use.
5. Review application access.
6. Investigate whether unauthorized activity occurred.
```

Do not simply delete the secret and assume the incident is resolved.

---

# PART 14 — CLOUDTRAIL + CLOUDWATCH + CONFIG

# 90. The Three-Service Investigation Pattern

This pattern is extremely important.

```text
CloudTrail
    |
    | Who performed an action?
    v
AWS Config
    |
    | What is/was the resource configuration?
    v
CloudWatch
    |
    | What was the operational impact?
    v
Incident picture
```

Example:

```text
Security Group changed
       |
       +--> CloudTrail
       |      Who changed it?
       |
       +--> Config
       |      What was the old/new configuration?
       |
       +--> CloudWatch
              Did traffic/errors/latency change?
```

---

# PART 15 — SECURITY HUB + DETECTION SERVICES

# 91. Findings Pipeline

A useful architecture:

```text
GuardDuty --------\
Inspector ---------\
Macie --------------> Security Hub
AWS Config ---------/
IAM Access Analyzer/
```

Then:

```text
Security Hub
      |
      v
Security Team
      |
      v
Investigation
      |
      v
Remediation
```

Security Hub supports integrations with many AWS security services.

---

# 92. Finding Is Not Automatically an Incident

Important distinction:

```text
Finding
   !=
Confirmed security incident
```

A finding is a signal requiring assessment.

For example:

```text
Inspector:
Known vulnerable package
```

does not necessarily mean:

```text
Attacker compromised the server
```

Likewise:

```text
GuardDuty:
Suspicious activity
```

requires investigation and context.

---

# PART 16 — KMS + SECRETS + DATA

# 93. Data Protection Architecture

Example:

```text
Application
    |
    +-------------------+
    |                   |
    v                   v
Secrets Manager       S3
    |                   |
    |                   v
    |                  KMS
    v
   KMS
```

For RDS:

```text
Application
    |
    v
Secrets Manager
    |
    v
Credentials
    |
    v
RDS
    |
    v
KMS encryption
```

The distinction:

```text
Secrets Manager
    =
protect secret values

KMS
    =
manage cryptographic keys
```

---

# PART 17 — WAF + SHIELD + SECURITY GROUP

# 94. Network/Application Security Layers

Think:

```text
Internet
   |
   v
Shield
   |
   v
WAF
   |
   v
CloudFront / ALB
   |
   v
Security Group
   |
   v
EC2 / ECS
```

Different layers:

```text
Shield
    DDoS

WAF
    HTTP request filtering

Security Group
    network connection filtering

Application
    authentication/authorization/business security
```

No single layer replaces the others.

---

# PART 18 — HANDS-ON LABS

# 95. Lab 1 — CloudTrail Investigation

Create a test EC2 instance.

Then:

```text
Stop instance
Start instance
Terminate instance
```

Use CloudTrail to find:

```text
StopInstances
StartInstances
TerminateInstances
```

For each event identify:

```text
Who
When
Where
What
Resource
```

---

# 96. Lab 2 — CloudTrail S3 Activity

Use a test S3 bucket.

Perform:

```text
PutObject
GetObject
DeleteObject
```

Understand the distinction between:

```text
management events
```

and:

```text
data events
```

Study how object-level logging is configured and what its cost implications are.

---

# 97. Lab 3 — CloudWatch EC2

Create an EC2 instance.

Monitor:

```text
CPU
Network
Status checks
```

Create:

```text
CPU > 70%
```

alarm.

Send the alarm to SNS or another notification target.

---

# 98. Lab 4 — CloudWatch Application Logs

Configure an application to send logs to:

```text
CloudWatch Logs
```

Create:

```text
/phase9/app
```

Then query:

```text
ERROR
WARN
500
```

using Logs Insights.

---

# 99. Lab 5 — CloudWatch Dashboard

Create a dashboard:

```text
Phase9-Production
```

Add:

```text
EC2 CPU
ALB RequestCount
ALB 5XX
ALB TargetResponseTime
RDS CPU
RDS DatabaseConnections
```

---

# 100. Lab 6 — AWS Config

Enable AWS Config in the learning Region.

Record selected resource types.

Create/study rules such as:

```text
Required tags
S3 public access
Security group unrestricted SSH
RDS public accessibility
```

Observe:

```text
COMPLIANT
NON_COMPLIANT
```

---

# 101. Lab 7 — Config Timeline

Take a test Security Group.

Change:

```text
SSH source
```

from:

```text
your IP
```

to another test value.

Then restore it.

Use Config to inspect the configuration timeline.

Use CloudTrail to identify the API operation.

---

# 102. Lab 8 — GuardDuty

Enable GuardDuty in the learning Region.

Study:

```text
Dashboard
Findings
Severity
Resource
Finding type
Protection plans
```

Use AWS-supported sample/test finding functionality where available.

Do not generate real attacks.

---

# 103. Lab 9 — Security Hub

Enable Security Hub CSPM.

Study:

```text
Security standards
Controls
Findings
Account coverage
Integrations
```

Observe findings from:

```text
GuardDuty
Inspector
Macie
Config
```

where the relevant integrations are enabled.

---

# 104. Lab 10 — Inspector + ECR

Use the container image from Phase 7.

Push an image to:

```text
ECR
```

Enable appropriate Inspector scanning.

Inspect:

```text
Image
Package
Vulnerability
Severity
Recommendation
```

---

# 105. Lab 11 — Macie

Create a test S3 bucket containing synthetic sample data.

For example:

```text
customers.csv
```

Use only fake/non-production data.

Enable Macie and study:

```text
S3 inventory
Automated sensitive data discovery
Sensitive data findings
```

Do not upload real personal information for a learning lab.

---

# 106. Lab 12 — KMS

Create a customer managed KMS key.

Study:

```text
Key policy
Key administrators
Key users
Key ARN
Key rotation settings
CloudTrail events
```

Use the key to encrypt a test resource.

Do not experiment with production encryption keys.

---

# 107. Lab 13 — KMS + S3

Create:

```text
S3 bucket
```

Configure:

```text
SSE-KMS
```

using the test KMS key.

Then investigate:

```text
Who can PutObject?
Who can GetObject?
Who can use kms:Encrypt?
Who can use kms:Decrypt?
```

Understand the interaction between:

```text
S3 permissions
KMS key policy
IAM permissions
```

---

# 108. Lab 14 — Secrets Manager

Create:

```text
phase9/test/db
```

with a fake credential.

Then configure an EC2/ECS role with only the required read permission.

Application flow:

```text
EC2/ECS
   |
   v
Secrets Manager
   |
   v
Secret
```

Do not put the secret directly into source code.

---

# 109. Lab 15 — WAF

Create or use:

```text
CloudFront
```

or:

```text
ALB
```

Create a Web ACL.

Start with a simple rule such as:

```text
Block a test IP
```

or:

```text
Count a matching request
```

Observe:

```text
Allowed
Blocked
Counted
```

Use WAF logs/metrics where configured.

---

# 110. Lab 16 — WAF Rate-Based Rule

Configure a rate-based rule in a safe test environment.

Generate normal test traffic.

Observe the rule's behavior.

Understand:

```text
Threshold
Evaluation window
Aggregation
Action
```

Do not use aggressive traffic generation.

---

# 111. Lab 17 — Full Security Architecture

Build:

```text
Route 53
   |
   v
CloudFront
   |
   +-- Shield
   |
   +-- WAF
   |
   v
ALB
   |
   v
ECS / EC2
   |
   v
RDS
```

Then enable:

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
```

where appropriate for your learning environment.

---

# PART 19 — TROUBLESHOOTING

# 112. "Who Changed This?"

Start:

```text
CloudTrail
```

Search:

```text
Event name
Resource
Identity
Time
```

Then use Config when you need configuration history/state.

---

# 113. "Why Is the Server Slow?"

Start:

```text
CloudWatch
```

Check:

```text
CPU
Memory if collected
Network
Disk/application metrics
Logs
ALB latency
Database metrics
```

Then investigate:

```text
EC2
ECS
RDS
application
```

---

# 114. "Is This an Attack?"

Look at:

```text
GuardDuty
WAF
Shield
CloudWatch
CloudTrail
```

Ask:

```text
Is there suspicious AWS activity?
Is there abnormal web traffic?
Is there a DDoS signal?
Are application errors increasing?
```

---

# 115. "Is the Resource Misconfigured?"

Use:

```text
AWS Config
```

Examples:

```text
Public S3
Open SSH
Public RDS
Missing encryption
Missing required tags
```

---

# 116. "Is the Software Vulnerable?"

Use:

```text
Inspector
```

For:

```text
EC2
ECR
Lambda
```

where supported/enabled.

---

# 117. "Does S3 Contain Sensitive Data?"

Use:

```text
Macie
```

Then review:

```text
Bucket
Object
Sensitive data type
Access
Encryption
Business need
Retention
```

---

# 118. "Application Cannot Read Secret"

Check:

```text
1. IAM role
2. secretsmanager:GetSecretValue
3. Secret ARN
4. KMS permissions if customer managed key is used
5. KMS key policy
6. Region
7. Secret version
```

---

# 119. "Application Cannot Decrypt Data"

Check:

```text
KMS key
Key policy
IAM permission
Grant if applicable
Region
Encryption context if relevant
```

Remember:

```text
KMS authorization is not simply ordinary IAM authorization.
```

---

# 120. "WAF Is Blocking Valid Requests"

Check:

```text
Web ACL
Rule
Managed rule group
Match statement
Action
Logs
Labels
```

Start suspicious rules in:

```text
Count
```

mode when appropriate for safe testing, then tune before blocking production traffic.

---

# PART 20 — SECURITY OPERATIONS

# 121. Daily Security Questions

A security engineer should routinely ask:

```text
Any suspicious findings?

Any critical vulnerabilities?

Any public resources?

Any configuration violations?

Any unusual API activity?

Any secrets exposed?

Any sensitive S3 data exposed?

Any WAF spikes?

Any DDoS signals?

Any failed security controls?
```

---

# 122. Weekly Security Review

Example:

```text
CloudTrail
    |
    +-- unusual administrative actions

GuardDuty
    |
    +-- findings

Security Hub
    |
    +-- critical/high findings

Inspector
    |
    +-- critical vulnerabilities

Macie
    |
    +-- sensitive data findings

Config
    |
    +-- non-compliant resources

WAF
    |
    +-- blocked requests / patterns

CloudWatch
    |
    +-- operational anomalies
```

---

# 123. Security Incident Workflow

A basic incident workflow:

```text
Detect
  |
  v
Triage
  |
  v
Investigate
  |
  v
Contain
  |
  v
Eradicate
  |
  v
Recover
  |
  v
Lessons learned
```

AWS services contribute different evidence.

---

# 124. Detection

Possible sources:

```text
GuardDuty
Security Hub
CloudWatch
WAF
Inspector
Macie
CloudTrail
```

---

# 125. Investigation

Use:

```text
CloudTrail
AWS Config
CloudWatch Logs
CloudWatch Metrics
GuardDuty
Security Hub
service logs
```

Question:

```text
What happened?
When?
Who?
What resource?
What changed?
What was affected?
```

---

# 126. Containment

Depending on the incident, possible actions may include:

```text
Disable compromised credentials
Rotate secrets
Restrict Security Groups
Block malicious IPs
Update WAF
Isolate instance
Remove compromised workload
```

Containment should follow the company's incident-response procedures.

Do not make destructive changes blindly because forensic evidence may be needed.

---

# 127. Recovery

After containment:

```text
Patch
Rotate
Restore
Redeploy
Validate
Monitor
```

Then confirm:

```text
Finding resolved
Resource compliant
Application healthy
Logs available
No continuing suspicious activity
```

---

# PART 21 — COMPANY SECURITY ARCHITECTURE

# 128. Multi-Account Security

From Phase 2:

```text
Organization
|
+-- Management
|
+-- Security OU
|   |
|   +-- Security Account
|   +-- Log Archive
|
+-- Infrastructure OU
|   |
|   +-- Network Account
|
+-- Workloads OU
    |
    +-- Development
    +-- Staging
    +-- Production
```

Security services can be centrally administered/aggregated where supported.

---

# 129. Security Account

A dedicated security account can host centralized security operations.

Possible responsibilities:

```text
GuardDuty administration
Security Hub administration
Central security findings
Security investigations
Delegated administration
Security tooling
```

The exact design depends on the organization's operating model.

---

# 130. Log Archive

A log archive account can store security/audit logs.

Example:

```text
Accounts
   |
   +-- CloudTrail
   |
   v
Central logging
   |
   v
Log Archive S3
```

Protect the log archive strongly.

---

# 131. Security Team Access

Security team should receive appropriate access through:

```text
IAM Identity Center
```

rather than distributing long-lived access keys unnecessarily.

Example permission set:

```text
Security-ReadOnly
```

or a controlled incident-response role.

Use least privilege.

---

# PART 22 — IMPORTANT COMPARISONS

# 132. CloudTrail vs CloudWatch

```text
CloudTrail
    Who did what?

CloudWatch
    What is happening?
```

Example:

```text
CloudTrail:
Alice stopped EC2.

CloudWatch:
EC2 CPU dropped to zero and application traffic changed.
```

---

# 133. CloudTrail vs Config

```text
CloudTrail
    Activity/audit trail

Config
    Resource configuration/compliance
```

---

# 134. GuardDuty vs Inspector

```text
GuardDuty
    Threat detection

Inspector
    Vulnerability management
```

---

# 135. GuardDuty vs Macie

```text
GuardDuty
    Suspicious activity

Macie
    Sensitive data in S3
```

---

# 136. Security Hub vs GuardDuty

```text
GuardDuty
    Generates threat findings

Security Hub
    Centralizes/organizes findings and security posture
```

Security Hub does not replace GuardDuty.

---

# 137. KMS vs Secrets Manager

```text
KMS
    Cryptographic key management

Secrets Manager
    Secret value management
```

They commonly work together.

---

# 138. WAF vs Shield

```text
WAF
    Web request filtering

Shield
    DDoS protection
```

---

# 139. WAF vs Security Group

```text
Security Group
    Network connection rules

WAF
    HTTP/HTTPS request rules
```

---

# 140. Config vs Security Hub

```text
Config
    Configuration/compliance evaluation

Security Hub
    Security posture/findings aggregation
```

They complement each other.

---

# PART 23 — PRODUCTION SECURITY CHECKLIST

# 141. Identity

```text
[ ] IAM Identity Center used for human access where appropriate
[ ] MFA enabled
[ ] Least privilege
[ ] No unnecessary long-lived access keys
[ ] Separate admin/security roles
[ ] Access Analyzer reviewed
```

---

# 142. Audit

```text
[ ] CloudTrail enabled appropriately
[ ] Central audit logging designed
[ ] Log retention defined
[ ] CloudTrail integrity/monitoring requirements reviewed
[ ] Important data events enabled where required
```

---

# 143. Monitoring

```text
[ ] CloudWatch metrics
[ ] CloudWatch logs
[ ] Alarms
[ ] Dashboards
[ ] Log retention
[ ] Incident notifications
```

---

# 144. Configuration

```text
[ ] AWS Config enabled where required
[ ] Rules defined
[ ] Required tags
[ ] Public exposure checks
[ ] Encryption checks
[ ] Compliance review
```

---

# 145. Threat Detection

```text
[ ] GuardDuty enabled
[ ] Appropriate protection plans reviewed
[ ] Security Hub enabled
[ ] Findings routed to security team
```

AWS currently recommends considering GuardDuty coverage across supported Regions in an organization, because global-service activity such as IAM can otherwise have reduced detection coverage in Regions where GuardDuty is not enabled.

---

# 146. Vulnerability Management

```text
[ ] Inspector enabled where appropriate
[ ] ECR images scanned
[ ] EC2 workloads scanned
[ ] Critical vulnerabilities reviewed
[ ] Patch process defined
```

---

# 147. Data Security

```text
[ ] S3 Block Public Access
[ ] Encryption
[ ] KMS controls
[ ] Macie for sensitive S3 data where appropriate
[ ] Data classification
[ ] Retention rules
```

---

# 148. Secrets

```text
[ ] Secrets Manager
[ ] No secrets in Git
[ ] No secrets in Dockerfiles
[ ] Rotation where appropriate
[ ] Least-privilege access
[ ] KMS permissions reviewed
```

---

# 149. Application Protection

```text
[ ] WAF
[ ] Shield
[ ] CloudFront
[ ] HTTPS
[ ] Secure headers/application controls
[ ] Rate limiting
[ ] DDoS strategy
```

---

# PART 24 — FINAL ARCHITECTURE

# 150. Full Security Architecture

```text
                                      INTERNET
                                          |
                                          v
                                      Route 53
                                          |
                                          v
                                      CloudFront
                                      /        \
                                     /          \
                                Shield           WAF
                                                  |
                                                  v
                                                 ALB
                                                  |
                                    +-------------+-------------+
                                    |                           |
                                    v                           v
                                  ECS                         EC2
                                    |                           |
                                    +-------------+-------------+
                                                  |
                                                  v
                                                 RDS
```

Security/observability:

```text
                    +---------------- AWS ----------------+
                    |                                     |
                    | CloudTrail  -> Audit                |
                    | CloudWatch  -> Observability        |
                    | Config      -> Configuration        |
                    | GuardDuty   -> Threat Detection     |
                    | Security Hub-> Security Posture     |
                    | Inspector   -> Vulnerabilities      |
                    | Macie       -> S3 Sensitive Data    |
                    | KMS         -> Encryption Keys      |
                    | Secrets     -> Application Secrets  |
                    | WAF         -> Web Protection       |
                    | Shield      -> DDoS Protection      |
                    +-------------------------------------+
```

---

# 151. Complete Incident Example

Imagine:

```text
Production EC2
```

suddenly has:

```text
High CPU
```

Step 1:

```text
CloudWatch
```

shows:

```text
CPU = 99%
```

Step 2:

Check:

```text
Application logs
```

Step 3:

Check:

```text
GuardDuty
```

for suspicious activity.

Step 4:

Check:

```text
CloudTrail
```

for unusual IAM/API activity.

Step 5:

Check:

```text
Inspector
```

for vulnerabilities.

Step 6:

Check:

```text
AWS Config
```

for configuration changes.

Step 7:

If the application is receiving malicious HTTP requests:

```text
WAF
```

and:

```text
Shield
```

become relevant.

Step 8:

Use:

```text
Security Hub
```

to consolidate relevant findings.

This is how the services work together rather than as isolated products.

---

# 152. Final Mental Model

Memorize this:

```text
CloudTrail
    "Who did what?"

CloudWatch
    "What is happening?"

AWS Config
    "What is configured, and is it compliant?"

GuardDuty
    "Is there suspicious activity?"

Security Hub
    "What security findings/posture need attention?"

Inspector
    "Are my workloads/software vulnerable?"

Macie
    "Where is sensitive data in S3?"

KMS
    "How are encryption keys controlled?"

Secrets Manager
    "Where do application secrets live?"

WAF
    "Which web requests should be allowed or blocked?"

Shield
    "How is my application protected against DDoS?"
```

The most important architecture is:

```text
             PREVENT
                |
       +--------+--------+
       |                 |
      KMS              WAF
       |                 |
 Secrets Manager       Shield
       |
       v
      DATA

             DETECT
                |
    +-----------+-----------+
    |           |           |
CloudTrail   GuardDuty   Inspector
    |           |           |
    +-----------+-----------+
                |
                v
          Security Hub

            GOVERN
                |
                v
           AWS Config

          OBSERVE
                |
                v
          CloudWatch

        DATA SECURITY
                |
                v
              Macie
```

---

# 153. Phase 9 Completion Checklist

## CloudTrail

```text
[ ] I understand CloudTrail
[ ] I understand management events
[ ] I understand data events
[ ] I understand Event History
[ ] I understand Trails
[ ] I can investigate an API action
[ ] I understand CloudTrail + S3
```

## CloudWatch

```text
[ ] I understand metrics
[ ] I understand dimensions
[ ] I understand logs
[ ] I understand log groups
[ ] I understand log streams
[ ] I understand Logs Insights
[ ] I understand alarms
[ ] I understand dashboards
[ ] I understand EventBridge relationship
```

## AWS Config

```text
[ ] I understand configuration recording
[ ] I understand resource history
[ ] I understand Config Rules
[ ] I understand compliance
[ ] I understand Config vs CloudTrail
```

## GuardDuty

```text
[ ] I understand threat detection
[ ] I understand findings
[ ] I understand protection plans
[ ] I understand GuardDuty vs CloudTrail
[ ] I understand multi-account administration
```

## Security Hub

```text
[ ] I understand findings aggregation
[ ] I understand security controls
[ ] I understand security standards
[ ] I understand integrations
[ ] I understand account/Region coverage
```

## Inspector

```text
[ ] I understand vulnerability management
[ ] I understand EC2 scanning
[ ] I understand ECR image scanning
[ ] I understand Inspector vs GuardDuty
```

## Macie

```text
[ ] I understand S3 data discovery
[ ] I understand sensitive data findings
[ ] I understand automated discovery
[ ] I understand Macie vs GuardDuty
```

## KMS

```text
[ ] I understand encryption keys
[ ] I understand key policies
[ ] I understand IAM + KMS authorization
[ ] I understand envelope encryption
[ ] I understand grants
[ ] I understand Regional keys
[ ] I understand CloudTrail + KMS
```

## Secrets Manager

```text
[ ] I understand secret storage
[ ] I understand GetSecretValue
[ ] I understand rotation
[ ] I understand KMS integration
[ ] I understand Secrets Manager vs Parameter Store
```

## WAF

```text
[ ] I understand Web ACLs
[ ] I understand rules
[ ] I understand managed rules
[ ] I understand rate-based rules
[ ] I understand WAF vs Security Group
```

## Shield

```text
[ ] I understand DDoS
[ ] I understand Shield Standard
[ ] I understand Shield Advanced
[ ] I understand Shield + WAF
[ ] I understand Shield + CloudFront
```

## Architecture

```text
[ ] CloudTrail + CloudWatch
[ ] CloudTrail + Config
[ ] GuardDuty + Security Hub
[ ] Inspector + Security Hub
[ ] Macie + Security Hub
[ ] KMS + S3
[ ] KMS + RDS
[ ] Secrets Manager + RDS
[ ] WAF + CloudFront
[ ] Shield + CloudFront
[ ] Full security architecture
```

---

# 154. Official AWS Documentation

## CloudTrail

https://docs.aws.amazon.com/awscloudtrail/latest/userguide/

## CloudWatch

https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/

## AWS Config

https://docs.aws.amazon.com/config/latest/developerguide/

## GuardDuty

https://docs.aws.amazon.com/guardduty/latest/ug/

## Security Hub

https://docs.aws.amazon.com/securityhub/

## Amazon Inspector

https://docs.aws.amazon.com/inspector/

## Amazon Macie

https://docs.aws.amazon.com/macie/

## AWS KMS

https://docs.aws.amazon.com/kms/latest/developerguide/

## Secrets Manager

https://docs.aws.amazon.com/secretsmanager/latest/userguide/

## AWS WAF

https://docs.aws.amazon.com/waf/latest/developerguide/

## AWS Shield

https://docs.aws.amazon.com/waf/latest/developerguide/shield-chapter.html

---

# 155. Final Phase 9 Principle

Do not memorize:

```text
CloudTrail = logs
CloudWatch = metrics
GuardDuty = security
```

Instead, learn to ask the correct question:

```text
WHO?
    -> CloudTrail

WHAT IS HAPPENING?
    -> CloudWatch

WHAT IS CONFIGURED?
    -> AWS Config

IS THIS SUSPICIOUS?
    -> GuardDuty

WHAT SECURITY FINDINGS NEED ATTENTION?
    -> Security Hub

IS THE SOFTWARE VULNERABLE?
    -> Inspector

IS SENSITIVE DATA IN S3?
    -> Macie

HOW ARE KEYS CONTROLLED?
    -> KMS

WHERE ARE APPLICATION SECRETS?
    -> Secrets Manager

SHOULD THIS WEB REQUEST BE BLOCKED?
    -> WAF

IS THE APPLICATION PROTECTED AGAINST DDoS?
    -> Shield
```

That question-based mental model is the foundation for operating AWS security professionally.

# End of Phase 9
