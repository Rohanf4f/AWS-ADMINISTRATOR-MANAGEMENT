# PHASE 5 — AMAZON S3

## Goal

Amazon S3 is not simply:

> "A place to upload files."

You need to understand S3 as an object-storage system with multiple layers:

```text
Bucket
   |
   +-- Objects
   |
   +-- Object keys / prefixes
   |
   +-- Versioning
   |
   +-- Encryption
   |
   +-- Access control
   |
   +-- Lifecycle
   |
   +-- Replication
   |
   +-- Storage classes
   |
   +-- Object Lock
   |
   +-- Events
   |
   +-- Logging / auditing
```

The most important S3 question is:

> Who can access this object, through which path, under which conditions?

---

# 5.1 S3 Mental Model

Think about S3 like this:

```text
AWS Account
     |
     v
   Bucket
     |
     +---------------------------+
     |                           |
     v                           v
  Objects                    Policies
     |                           |
     +-- key                     +-- IAM Policy
     +-- data                    +-- Bucket Policy
     +-- metadata                +-- Block Public Access
     +-- version                 +-- Object Ownership
```

Example:

```text
Bucket:
company-private-data

Objects:

users/1001/profile.jpg
users/1002/profile.jpg
reports/2026/09/report.pdf
logs/2026/09/26/app.log
backups/database/db.sql
```

S3 does not use traditional filesystem directories.

What looks like:

```text
users/1001/profile.jpg
```

is an object key containing a prefix:

```text
users/1001/
```

and an object name:

```text
profile.jpg
```

---

# 5.2 Bucket

A bucket is an S3 container for objects.

Example:

```text
company-private-data
```

Bucket-level configuration can include:

```text
Versioning
Encryption
Lifecycle
Object Ownership
Block Public Access
Bucket Policy
Replication
Object Lock
Event Notifications
Logging-related configuration
```

A bucket name is globally unique within the AWS partition namespace, so a name that is already taken cannot be reused for another bucket in the same namespace.

Official documentation:

https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingBucket.html

---

# 5.3 Object

An S3 object contains:

```text
Object key
Data
Metadata
Version ID (when versioning is enabled)
```

Example:

```text
Bucket:
company-private-data

Object key:
reports/2026/09/report.pdf
```

Think:

```text
Bucket
  |
  +-- Object
       |
       +-- Key
       +-- Data
       +-- Metadata
       +-- Version
```

An object is not an EC2 filesystem file.

S3 is object storage.

---

# 5.4 Prefix

A prefix is the beginning portion of an object key.

Example:

```text
company-private-data

reports/2026/09/a.pdf
reports/2026/09/b.pdf
reports/2026/10/c.pdf
```

Prefix:

```text
reports/2026/09/
```

This is useful for organizing data logically:

```text
users/
reports/
logs/
backups/
uploads/
```

Do not think of prefixes as real directories.

They are part of object keys.

---

# 5.5 S3 Storage Classes

S3 provides different storage classes for different access patterns.

Learn the major concepts:

```text
S3 Standard
S3 Intelligent-Tiering
S3 Standard-IA
S3 One Zone-IA
S3 Glacier Instant Retrieval
S3 Glacier Flexible Retrieval
S3 Glacier Deep Archive
```

Mental model:

```text
Frequently accessed
        |
        v
S3 Standard

Unknown/changing access
        |
        v
Intelligent-Tiering

Infrequent access
        |
        v
Standard-IA / One Zone-IA

Archive
        |
        v
Glacier classes
```

Do not choose a storage class only because its storage price is lower.

Consider:

```text
Access frequency
Retrieval charges
Minimum storage duration
Availability/resilience characteristics
Object size
Lifecycle transition cost
Application latency requirements
```

Official documentation:

https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html

---

# 5.6 Versioning

S3 Versioning keeps multiple versions of an object in the same bucket.

Example:

```text
report.pdf
   |
   +-- Version 1
   +-- Version 2
   +-- Version 3
```

If a user accidentally overwrites an object:

```text
Version 1
    |
    v
Version 2
```

the previous version can remain available.

Versioning is useful for:

```text
Accidental overwrite recovery
Accidental deletion recovery
Data protection
Replication workflows
Object Lock
```

Important:

Versioning is not the same as a backup strategy.

You still need to design retention, replication, recovery, and access controls appropriately.

Official documentation:

https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html

---

# 5.7 Delete Markers

In a versioning-enabled bucket, deleting the current object does not necessarily destroy all historical versions.

S3 can create a:

```text
Delete Marker
```

Example:

```text
report.pdf

Version 3
Version 2
Version 1
Delete Marker <- current version
```

The object appears deleted through normal reads, while older versions can still exist.

This becomes important when troubleshooting:

```text
"Why did my object disappear?"
```

and when designing lifecycle rules for noncurrent versions.

---

# 5.8 Encryption

S3 supports server-side encryption.

For modern S3 buckets, new objects are encrypted by default with SSE-S3 unless you configure another supported encryption option.

Important concepts:

```text
SSE-S3
SSE-KMS
DSSE-KMS
```

For learning:

```text
SSE-S3
=
S3-managed encryption

SSE-KMS
=
AWS KMS-backed encryption
```

Use SSE-KMS when you need stronger control over key policies, grants, auditing, or separation of key administration from S3 access.

Official documentation:

https://docs.aws.amazon.com/AmazonS3/latest/userguide/serv-side-encryption.html

---

# 5.9 Encryption in Transit

Encryption at rest is not the whole story.

For data moving between client and S3:

```text
Client
   |
   | HTTPS / TLS
   v
S3
```

For sensitive buckets, you can enforce HTTPS using a bucket policy condition such as:

```text
aws:SecureTransport
```

The mental model is:

```text
At rest
   -> S3 encryption

In transit
   -> TLS / HTTPS
```

---

# 5.10 IAM Policy vs Bucket Policy

This is one of the most important S3 concepts.

## IAM Policy

Attached to an identity:

```text
User
Role
Group
```

Example:

```text
DeveloperRole
     |
     v
IAM Policy
     |
     v
s3:GetObject
```

## Bucket Policy

Attached to the S3 bucket.

It is a resource-based policy.

Example:

```text
Bucket
   |
   v
Bucket Policy
   |
   v
Allow specific principal
```

Mental model:

```text
IAM Policy
=
What can this identity do?

Bucket Policy
=
Who can access this bucket/object and under what conditions?
```

Both participate in AWS authorization.

---

# 5.11 Example IAM Policy

Example read-only object access:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::company-private-data/reports/*"
    }
  ]
}
```

This means the identity can read objects under:

```text
reports/
```

but does not automatically grant:

```text
s3:PutObject
s3:DeleteObject
```

---

# 5.12 Example Bucket Policy

A bucket policy can restrict access based on conditions.

Example concept:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyInsecureTransport",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::company-private-data",
        "arn:aws:s3:::company-private-data/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
```

The exact production policy should be tested carefully before deployment.

---

# 5.13 Block Public Access

For private company data, understand:

```text
Block Public Access
```

S3 Block Public Access provides controls at account, bucket, access point, and organization levels.

The four bucket-level settings are designed to prevent public access through different policy/ACL paths.

For a private data bucket, the normal learning configuration is:

```text
Block all public access
= ON
```

Important:

Block Public Access can override policies or ACLs that would otherwise make a bucket/object public.

AWS recommends keeping Block Public Access enabled unless a documented use case requires public access.

Official documentation:

https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html

---

# 5.14 ACLs

S3 historically supported Access Control Lists.

Modern S3 design generally uses:

```text
IAM policies
+
Bucket policies
+
Object Ownership
```

rather than object ACLs.

New S3 buckets default to:

```text
Object Ownership
Bucket owner enforced
```

which disables ACLs.

This means:

```text
Bucket owner
      |
      v
Owns uploaded objects
      |
      v
Policies control access
```

Use ACLs only when you have a specific compatibility requirement.

Official documentation:

https://docs.aws.amazon.com/AmazonS3/latest/userguide/about-object-ownership.html

---

# 5.15 Object Ownership

Understand the three settings:

```text
Bucket owner enforced
Bucket owner preferred
Object writer
```

The modern default:

```text
Bucket owner enforced
```

means ACLs are disabled and the bucket owner owns objects.

This simplifies cross-account object ownership and access management.

For most new architectures:

```text
Object Ownership
= Bucket owner enforced
```

Official documentation:

https://docs.aws.amazon.com/AmazonS3/latest/userguide/about-object-ownership.html

---

# 5.16 Lifecycle Rules

Lifecycle rules automate object management.

Example:

```text
Day 0
 |
 v
S3 Standard

Day 30
 |
 v
S3 Standard-IA

Day 90
 |
 v
Glacier

Day 365
 |
 v
Delete
```

Lifecycle rules can:

```text
Transition objects
Expire objects
Manage noncurrent versions
Abort incomplete multipart uploads
```

A versioned bucket may need separate rules for:

```text
Current versions
Noncurrent versions
```

Official documentation:

https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html

---

# 5.17 Lifecycle Example

For logs:

```text
Prefix:
logs/
```

Example policy:

```text
0-30 days
S3 Standard

30-90 days
Infrequent access

90+ days
Archive

After retention period
Delete
```

Do not blindly copy these numbers into production.

Choose retention based on:

```text
Business requirement
Compliance
Recovery requirement
Access pattern
Cost
```

---

# 5.18 Replication

S3 Replication automatically copies objects between S3 locations.

Major concepts:

```text
Same-Region Replication
Cross-Region Replication
```

Example:

```text
Production Bucket
       |
       | Replication
       v
Backup / DR Bucket
```

Replication is useful for:

```text
Disaster recovery
Regional redundancy
Compliance
Data distribution
Account separation
```

Understand that replication is not automatically a complete backup strategy.

You need to consider:

```text
Versioning
Replication configuration
Delete-marker behavior
Existing objects
Replication permissions
Encryption
Destination ownership
Recovery requirements
```

Official documentation:

https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html

---

# 5.19 Object Lock

S3 Object Lock provides a WORM model:

```text
Write Once
Read Many
```

It can prevent objects from being deleted or overwritten for a defined retention period or indefinitely.

Conceptually:

```text
Object
  |
  v
Object Lock
  |
  +-- Retention
  +-- Legal Hold
```

Object Lock requires versioning.

Two important retention modes:

```text
Governance
Compliance
```

Also learn:

```text
Legal Hold
Retain Until Date
```

Be careful:

Object Lock can make data intentionally difficult or impossible to delete during retention.

Do not enable it casually on a learning bucket if you do not understand the consequences.

Official documentation:

https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html

---

# 5.20 Presigned URL

A presigned URL gives temporary access to a specific S3 operation/object.

Conceptually:

```text
Application
    |
    v
Generate Presigned URL
    |
    v
Client
    |
    v
S3
```

Example use case:

```text
User uploads profile image
```

Instead of:

```text
Browser
   |
   v
Backend
   |
   v
S3
```

you can use:

```text
Browser
   |
   | presigned PUT
   v
S3
```

The application can generate a URL that expires after a limited time.

Important:

```text
Presigned URL
!= Public bucket
```

The bucket can remain private.

Official documentation:

https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html

---

# 5.21 Multipart Upload

Multipart upload splits a large object into parts.

Conceptually:

```text
Large File
   |
   +-- Part 1
   +-- Part 2
   +-- Part 3
   +-- Part 4
   |
   v
S3
   |
   v
Complete Object
```

Benefits:

```text
Parallel upload
Better handling of large objects
Retry individual failed parts
Resume interrupted uploads
```

Important operational concept:

```text
Initiate
   |
   v
Upload parts
   |
   v
Complete
```

Incomplete multipart uploads should be cleaned up using lifecycle rules when appropriate.

Official documentation:

https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html

---

# 5.22 S3 Events

S3 can publish event notifications when certain bucket events occur.

Example:

```text
Object uploaded
     |
     v
S3 Event
     |
     +--> SQS
     +--> SNS
     +--> Lambda
     +--> EventBridge
```

Use cases:

```text
Image upload
    |
    v
Lambda
    |
    v
Thumbnail generation
```

or:

```text
CSV uploaded
    |
    v
Event
    |
    v
Processing pipeline
```

S3 event notifications are designed for at-least-once delivery, so consumers should be designed to handle duplicate events safely.

Official documentation:

https://docs.aws.amazon.com/AmazonS3/latest/userguide/EventNotifications.html

---

# 5.23 Access Logs vs CloudTrail

These are not the same thing.

## S3 Server Access Logging

Provides request/access records for a bucket.

Useful for:

```text
Request activity analysis
Access investigation
Operational troubleshooting
```

Server access logs are delivered on a best-effort basis and are not a complete accounting of every request.

## AWS CloudTrail

CloudTrail records AWS API activity.

For S3:

```text
CloudTrail
 |
 +-- Bucket-level API activity
 +-- Object-level data events when configured
```

Important:

S3 object-level data events are not the same as management events and may need to be explicitly configured.

Mental model:

```text
Server Access Logs
=
S3 request/access logging

CloudTrail
=
AWS API audit trail
```

Official documentation:

https://docs.aws.amazon.com/AmazonS3/latest/userguide/ServerLogs.html

https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html

---

# 5.24 S3 Access Control Mental Model

When access fails, think:

```text
Client
  |
  v
Identity
  |
  +-- IAM Policy
  |
  v
S3
  |
  +-- Block Public Access
  +-- Bucket Policy
  +-- Object Ownership / ACL
  +-- Encryption / KMS permissions
  |
  v
Object
```

For an AWS principal to access an object, all relevant authorization controls must permit the request and no applicable explicit deny can override it.

---

# 5.25 PRIVATE S3

Recommended company private-data architecture:

```text
Company Application
        |
        v
IAM Role
        |
        v
Private S3 Bucket
        |
        v
Objects
```

Bucket:

```text
Block Public Access = ON
Object Ownership = Bucket owner enforced
Encryption = enabled
Versioning = enabled when appropriate
```

Access:

```text
IAM
+
Bucket Policy
```

No public bucket access is required.

---

# 5.26 PUBLIC S3

A public S3 architecture intentionally allows anonymous/public access.

Conceptually:

```text
Internet
    |
    v
S3
    |
    v
Public Object
```

This should only be used when there is a documented reason.

For example, a public-content use case may require anonymous reads.

However, for modern web architectures, do not automatically make the S3 bucket public just because users need to download files.

A private S3 origin behind CloudFront is often the better architecture for content delivery.

---

# 5.27 CloudFront + Private S3

Modern architecture:

```text
User
  |
  v
CloudFront
  |
  | authenticated origin request
  v
Private S3
```

The S3 bucket stays private.

CloudFront uses:

```text
Origin Access Control
(OAC)
```

to authenticate requests to the S3 origin.

AWS currently recommends OAC over the legacy Origin Access Identity (OAI) for S3 origins.

Official documentation:

https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html

---

# 5.28 OAC Mental Model

Without OAC:

```text
User
 |
 +----> S3 directly
 |
 +----> CloudFront
```

With private S3 + OAC:

```text
User
 |
 v
CloudFront
 |
 | signed/authenticated request
 v
S3
```

The bucket policy grants the CloudFront service principal access and can restrict that access to the specific CloudFront distribution using the distribution ARN.

Example structure:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipalReadOnly",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::company-private-data/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::ACCOUNT_ID:distribution/DISTRIBUTION_ID"
        }
      }
    }
  ]
}
```

Replace:

```text
ACCOUNT_ID
DISTRIBUTION_ID
```

with your actual values.

Do not copy this policy blindly into production without checking the exact bucket, distribution, actions, and encryption requirements.

AWS OAC documentation:

https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html

---

# 5.29 OAC + SSE-KMS

If the S3 objects use SSE-KMS:

```text
User
 |
 v
CloudFront
 |
 v
S3
 |
 v
KMS
```

CloudFront also needs appropriate permission to use the KMS key.

This means there are two authorization layers to understand:

```text
S3 Bucket Policy
+
KMS Key Policy / permissions
```

AWS documents the additional KMS permissions required for CloudFront OAC with SSE-KMS.

Official documentation:

https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html

---

# 5.30 Important OAC Rule

When using OAC with a regular S3 bucket origin:

```text
Use S3 bucket origin
+
OAC
+
Private bucket
```

Do not confuse this with:

```text
S3 static website endpoint
```

The S3 website endpoint is a different origin model and does not support OAC in the same way.

For a private S3 + CloudFront architecture, use the regular S3 origin rather than configuring the bucket as a public website endpoint.

---

# 5.31 PROJECT — company-private-data

Now build the requested S3 bucket.

Create:

```text
company-private-data
```

Use a unique bucket name if AWS requires one that is not already taken.

---

# 5.32 Step 1 — Create Bucket

Go to:

```text
AWS Console
 ->
S3
 ->
Buckets
 ->
Create bucket
```

Choose the intended Region.

Configure:

```text
Bucket name:
company-private-data-<unique-suffix>
```

For a private data bucket:

```text
Block Public Access:
ON
```

Use:

```text
Object Ownership:
Bucket owner enforced
```

which keeps ACLs disabled for the normal modern configuration.

---

# 5.33 Step 2 — Enable Versioning

Open:

```text
Bucket
 ->
Properties
 ->
Bucket Versioning
 ->
Enable
```

Test by uploading:

```text
test.txt
```

Then modify:

```text
test.txt
```

Upload it again.

Observe the object versions.

Mental model:

```text
test.txt
 |
 +-- Version A
 +-- Version B
```

---

# 5.34 Step 3 — Encryption

Open:

```text
Bucket
 ->
Properties
 ->
Default encryption
```

Understand:

```text
SSE-S3
```

first.

Then separately study:

```text
SSE-KMS
```

for cases where centralized key management and key-policy controls are required.

Do not assume KMS is automatically "more secure" for every workload; it introduces key-management configuration and permissions that must be operated correctly.

---

# 5.35 Step 4 — Block Public Access

Verify all four bucket-level public-access-block controls are enabled for this private bucket.

Your target:

```text
BlockPublicAcls = ON
IgnorePublicAcls = ON
BlockPublicPolicy = ON
RestrictPublicBuckets = ON
```

The effective result should be:

```text
Anonymous public access
        |
        X
      BLOCKED
```

Official documentation:

https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html

---

# 5.36 Step 5 — Object Ownership

Verify:

```text
Object Ownership
=
Bucket owner enforced
```

Understand the consequence:

```text
ACLs disabled
        |
        v
Bucket policies / IAM policies
        |
        v
Access control
```

---

# 5.37 Step 6 — Bucket Policy

Add a policy that denies insecure transport.

Concept:

```text
HTTP
 |
 X
S3

HTTPS
 |
 v
S3
```

Example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyInsecureTransport",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::company-private-data-<unique-suffix>",
        "arn:aws:s3:::company-private-data-<unique-suffix>/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
```

This is a security guardrail, not the main mechanism for granting application access.

---

# 5.38 Step 7 — Create Prefixes

Upload test objects using logical prefixes:

```text
documents/
documents/company-policy.pdf

reports/
reports/2026/report.pdf

logs/
logs/2026/09/app.log

backups/
backups/test/db.sql
```

Remember:

```text
Prefix != physical directory
```

---

# 5.39 Step 8 — Lifecycle Rule

Create a lifecycle rule for a test prefix:

```text
logs/
```

For the learning exercise, configure a short transition/expiration policy only if the current S3 console permits the chosen transition timing and you understand the associated costs/constraints.

For a production design, define lifecycle based on:

```text
Retention requirement
Access pattern
Recovery requirement
Compliance
Cost
```

Also learn how versioning changes lifecycle behavior for:

```text
Current versions
Noncurrent versions
Delete markers
```

---

# 5.40 Step 9 — Test IAM Access

Create or use an IAM role for testing.

Give it only the permissions needed for the exercise.

Example:

```text
s3:ListBucket
s3:GetObject
```

for the relevant bucket/prefix.

Test:

```text
List bucket
Download object
```

Then remove:

```text
s3:GetObject
```

Test again.

Expected:

```text
AccessDenied
```

This connects S3 directly to your Phase 1 IAM policy knowledge.

---

# 5.41 Step 10 — Test Private Access

Try opening the object's direct S3 URL without authentication.

Expected:

```text
Access denied
```

The important result is:

```text
Bucket remains private
```

while authorized IAM principals can access it.

---

# 5.42 Step 11 — Presigned URL Exercise

Generate a presigned URL for a private object.

Conceptually:

```text
Private Object
      |
      v
Presigned URL
      |
      v
Temporary access
```

Test:

```text
URL works before expiration
URL stops working after expiration
```

Understand:

```text
Private bucket
+
temporary delegated access
```

---

# 5.43 Step 12 — Multipart Upload

Use a sufficiently large test object and learn:

```text
Initiate multipart upload
Upload parts
Complete multipart upload
```

Then intentionally interrupt an upload and learn how incomplete multipart uploads can be identified and cleaned up.

Add a lifecycle rule to abort incomplete multipart uploads after an appropriate number of days in real systems.

---

# 5.44 Step 13 — S3 Event

Create a test event:

```text
Object Created
```

Target:

```text
SQS
```

or another supported destination.

Test:

```text
Upload object
     |
     v
S3
     |
     v
Event
     |
     v
Destination
```

Remember that S3 event notifications are designed for at-least-once delivery.

Therefore consumers should tolerate duplicate processing.

---

# 5.45 Step 14 — CloudTrail

Learn the difference:

```text
S3 API activity
       |
       v
CloudTrail
```

and:

```text
S3 request/access logging
       |
       v
Server access logs
```

For a serious audit requirement, configure CloudTrail appropriately, including the relevant S3 data events when object-level API activity needs to be recorded.

Do not assume default CloudTrail management events automatically give you a complete object-level audit trail.

---

# 5.46 Step 15 — Object Lock Exercise

Do this in a separate test bucket.

Create:

```text
company-object-lock-lab
```

Enable:

```text
Versioning
Object Lock
```

Then upload a test object.

Learn:

```text
Retention
Legal Hold
Governance Mode
Compliance Mode
```

Do not experiment with irreversible retention settings on important company data.

Official documentation:

https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html

---

# 5.47 Step 16 — CloudFront + Private S3 Project

Create the architecture:

```text
                 User
                   |
                   v
              CloudFront
                   |
                   | OAC
                   v
            Private S3 Bucket
                   |
                   v
                Object
```

Requirements:

```text
S3 Block Public Access = ON
Object Ownership = Bucket owner enforced
S3 bucket = private
CloudFront = S3 origin
OAC = enabled
Bucket policy = CloudFront distribution access
```

Do not make the bucket public just to make CloudFront work.

---

# 5.48 CloudFront OAC Console Exercise

Conceptual workflow:

```text
CloudFront
 ->
Create distribution
 ->
S3 origin
 ->
Origin access
 ->
Origin Access Control
 ->
Create OAC
 ->
Sign requests
```

Then update the S3 bucket policy so the CloudFront service principal can access objects only for the intended distribution.

AWS recommends OAC for S3 origins over legacy OAI.

Official documentation:

https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html

---

# 5.49 Test Direct S3 vs CloudFront

After OAC is configured:

## Direct S3

```text
Browser
   |
   v
S3 URL
   |
   X
Access denied
```

## CloudFront

```text
Browser
   |
   v
CloudFront URL
   |
   v
OAC
   |
   v
Private S3
   |
   v
Object
```

Expected:

```text
CloudFront -> works
Direct S3 -> denied
```

This is one of the most important S3 + CloudFront exercises.

---

# 5.50 S3 Troubleshooting Framework

When S3 returns:

```text
AccessDenied
```

do not randomly modify policies.

Check:

```text
1. Which principal is making the request?
2. IAM identity policy?
3. Bucket policy?
4. Explicit Deny?
5. Block Public Access?
6. Object Ownership / ACL?
7. Correct bucket ARN?
8. Correct object ARN?
9. Correct Region?
10. KMS permissions?
11. VPC endpoint policy, if using a VPC endpoint?
12. Presigned URL still valid?
13. CloudFront OAC configured?
14. CloudFront bucket policy SourceArn correct?
```

---

# 5.51 Common S3 Mistakes

## Mistake 1 — Making a private bucket public

Do not use:

```text
Principal: "*"
Effect: Allow
Action: s3:GetObject
```

unless the bucket is intentionally public.

## Mistake 2 — Treating S3 prefixes as directories

They are key prefixes.

## Mistake 3 — Using ACLs everywhere

Modern S3 generally uses:

```text
Object Ownership
+
IAM
+
Bucket Policies
```

## Mistake 4 — Assuming versioning is backup

Versioning helps recover from accidental changes but is not automatically a complete backup/DR design.

## Mistake 5 — Exposing S3 because CloudFront needs access

Use:

```text
CloudFront OAC
+
Private S3
```

instead.

## Mistake 6 — Forgetting KMS permissions

SSE-KMS introduces KMS authorization into the access path.

## Mistake 7 — Ignoring lifecycle costs

Transitions and retrieval can have costs and timing constraints.

## Mistake 8 — Assuming event delivery is exactly once

Design consumers to tolerate duplicates.

---

# 5.52 Company S3 Architecture

A realistic private-data architecture:

```text
                    Company Users
                         |
                         v
                  Application / API
                         |
                         v
                    IAM Role
                         |
                         v
                 Private S3 Bucket
                         |
       +-----------------+------------------+
       |                 |                  |
       v                 v                  v
   Documents           Reports            Logs
       |                 |                  |
       +-----------------+------------------+
                         |
                    Lifecycle
                         |
             +-----------+-----------+
             |                       |
             v                       v
       Storage Classes          Expiration
```

For web-delivered content:

```text
User
 |
 v
CloudFront
 |
 | OAC
 v
Private S3
```

For uploads:

```text
Application
 |
 v
Presigned URL
 |
 v
Private S3
```

For processing:

```text
S3 Object Created
       |
       v
Event
       |
       v
SQS / Lambda / EventBridge
```

---

# 5.53 S3 Security Checklist

For a company-private bucket:

```text
[ ] Block Public Access enabled
[ ] Object Ownership = Bucket owner enforced
[ ] ACLs disabled unless required
[ ] Default encryption configured
[ ] IAM access least-privilege
[ ] Bucket policy reviewed
[ ] HTTPS enforced where appropriate
[ ] Versioning enabled where appropriate
[ ] Lifecycle rules reviewed
[ ] Noncurrent versions considered
[ ] Replication/DR requirement evaluated
[ ] Object Lock evaluated for compliance workloads
[ ] CloudTrail logging evaluated
[ ] Access logging evaluated where useful
[ ] Sensitive data not placed in object keys/tags unnecessarily
[ ] CloudFront uses OAC for private S3 origins
[ ] KMS permissions reviewed when SSE-KMS is used
```

---

# 5.54 Completion Checklist

You should be able to explain:

- Bucket
- Object
- Object key
- Prefix
- Versioning
- Delete markers
- Encryption
- SSE-S3
- SSE-KMS
- IAM policy
- Bucket policy
- ACL
- Object Ownership
- Block Public Access
- Lifecycle
- Storage classes
- Replication
- Object Lock
- Presigned URL
- Multipart upload
- S3 events
- Server access logging
- CloudTrail
- CloudFront OAC

You should be able to build:

```text
Private S3
```

with:

```text
Block Public Access
Versioning
Encryption
Lifecycle
Bucket Policy
Object Ownership
```

And then build:

```text
CloudFront
    |
    v
OAC
    |
    v
Private S3
```

without making the bucket public.

---

# 5.55 Final Mental Model

```text
                         S3
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
       Objects        Security       Lifecycle
          |              |              |
          |        +-----+-----+        |
          |        |     |     |        |
          |        v     v     v        |
          |      IAM   Bucket  B.P.A.   |
          |            Policy           |
          |                             |
          +-----------------------------+
                         |
                  Versioning / Lock
                         |
                  Storage Classes
                         |
                    Replication
                         |
                 Events / Logging
```

Modern private-content architecture:

```text
                         User
                           |
                           v
                       CloudFront
                           |
                           | OAC
                           v
                    PRIVATE S3 BUCKET
                           |
              +------------+------------+
              |            |            |
              v            v            v
          Versioning   Encryption   Lifecycle
```

The most important S3 skill is:

> Keep data private by default, grant the minimum required access, understand exactly which policy allows access, and be able to trace every request from the client to the object.

---

# Official AWS Documentation

## S3

https://docs.aws.amazon.com/AmazonS3/latest/userguide/

## Buckets

https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingBucket.html

## Versioning

https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html

## Encryption

https://docs.aws.amazon.com/AmazonS3/latest/userguide/serv-side-encryption.html

## Access Control

https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-management.html

## Object Ownership / ACLs

https://docs.aws.amazon.com/AmazonS3/latest/userguide/about-object-ownership.html

## Block Public Access

https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html

## Lifecycle

https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html

## Storage Classes

https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html

## Replication

https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html

## Object Lock

https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html

## Presigned URLs

https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html

## Multipart Upload

https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html

## Event Notifications

https://docs.aws.amazon.com/AmazonS3/latest/userguide/EventNotifications.html

## Server Access Logging

https://docs.aws.amazon.com/AmazonS3/latest/userguide/ServerLogs.html

## AWS CloudTrail

https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html

## CloudFront + S3 OAC

https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html
