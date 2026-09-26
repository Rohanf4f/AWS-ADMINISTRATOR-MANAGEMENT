# AWS IAM Actions Cheat Sheet

AWS IAM permissions generally follow:

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

# 1. S3 — `s3:*`

S3 is used for buckets and objects/files.

## Object Operations

```text
s3:GetObject
s3:GetObjectVersion
s3:PutObject
s3:PutObjectAcl
s3:DeleteObject
s3:DeleteObjectVersion
s3:RestoreObject
s3:AbortMultipartUpload
s3:ListMultipartUploadParts
```

## Bucket Operations

```text
s3:ListBucket
s3:ListBucketVersions
s3:ListBucketMultipartUploads
s3:GetBucketLocation
s3:GetBucketVersioning
s3:PutBucketVersioning
s3:GetBucketAcl
s3:PutBucketAcl
s3:GetBucketPolicy
s3:PutBucketPolicy
s3:DeleteBucketPolicy
s3:GetBucketCors
s3:PutBucketCors
s3:DeleteBucketCors
s3:GetBucketLogging
s3:PutBucketLogging
s3:GetBucketEncryption
s3:PutBucketEncryption
s3:GetBucketNotification
s3:PutBucketNotification
```

## Common S3 Permissions

Read:

```text
s3:GetObject
s3:ListBucket
```

Upload:

```text
s3:PutObject
s3:ListBucket
```

Read + Write + Delete:

```text
s3:GetObject
s3:PutObject
s3:DeleteObject
s3:ListBucket
```

All S3 actions:

```text
s3:*
```

---

# 2. EC2 — `ec2:*`

EC2 is used for virtual servers, networking, volumes, security groups, and AMIs.

## Instance Operations

```text
ec2:RunInstances
ec2:StartInstances
ec2:StopInstances
ec2:RebootInstances
ec2:TerminateInstances
ec2:DescribeInstances
ec2:DescribeInstanceStatus
ec2:DescribeInstanceTypes
```

## AMI Operations

```text
ec2:DescribeImages
ec2:CreateImage
ec2:DeregisterImage
ec2:CreateTags
```

## Security Groups

```text
ec2:DescribeSecurityGroups
ec2:CreateSecurityGroup
ec2:DeleteSecurityGroup
ec2:AuthorizeSecurityGroupIngress
ec2:AuthorizeSecurityGroupEgress
ec2:RevokeSecurityGroupIngress
ec2:RevokeSecurityGroupEgress
```

## EBS Volumes

```text
ec2:DescribeVolumes
ec2:CreateVolume
ec2:DeleteVolume
ec2:AttachVolume
ec2:DetachVolume
```

## Networking

```text
ec2:DescribeVpcs
ec2:DescribeSubnets
ec2:DescribeRouteTables
ec2:DescribeInternetGateways
ec2:DescribeNatGateways
ec2:DescribeNetworkInterfaces
ec2:DescribeAddresses
```

## Common Read-Only Pattern

```text
ec2:Describe*
```

---

# 3. ECR — `ecr:*`

ECR (Elastic Container Registry) stores Docker/container images.

## Authentication

```text
ecr:GetAuthorizationToken
```

## Pull Images

```text
ecr:BatchCheckLayerAvailability
ecr:BatchGetImage
ecr:GetDownloadUrlForLayer
```

## Push Images

```text
ecr:BatchCheckLayerAvailability
ecr:CompleteLayerUpload
ecr:InitiateLayerUpload
ecr:UploadLayerPart
ecr:PutImage
```

## Repository Operations

```text
ecr:CreateRepository
ecr:DeleteRepository
ecr:DescribeRepositories
ecr:ListImages
ecr:DescribeImages
ecr:BatchDeleteImage
```

## Lifecycle Policy

```text
ecr:GetLifecyclePolicy
ecr:PutLifecyclePolicy
ecr:DeleteLifecyclePolicy
```

## Repository Policy

```text
ecr:GetRepositoryPolicy
ecr:SetRepositoryPolicy
ecr:DeleteRepositoryPolicy
```

## Typical ECR Push Permissions

```text
ecr:GetAuthorizationToken
ecr:BatchCheckLayerAvailability
ecr:CompleteLayerUpload
ecr:InitiateLayerUpload
ecr:UploadLayerPart
ecr:PutImage
```

---

# 4. RDS — `rds:*`

RDS is used for managed relational databases.

## Database Information

```text
rds:DescribeDBInstances
rds:DescribeDBClusters
rds:DescribeDBSnapshots
rds:DescribeDBClusterSnapshots
```

## Database Lifecycle

```text
rds:CreateDBInstance
rds:DeleteDBInstance
rds:ModifyDBInstance
rds:RebootDBInstance
rds:StartDBInstance
rds:StopDBInstance
```

## Snapshots

```text
rds:CreateDBSnapshot
rds:DeleteDBSnapshot
rds:RestoreDBInstanceFromDBSnapshot
```

## Aurora / Cluster Operations

```text
rds:CreateDBCluster
rds:DeleteDBCluster
rds:ModifyDBCluster
rds:StartDBCluster
rds:StopDBCluster
rds:RebootDBCluster
```

## Parameter Groups

```text
rds:DescribeDBParameterGroups
rds:CreateDBParameterGroup
rds:ModifyDBParameterGroup
rds:DeleteDBParameterGroup
```

## Common Read-Only Pattern

```text
rds:Describe*
```

---

# 5. SageMaker — `sagemaker:*`

SageMaker is used for ML models, training, processing, and inference endpoints.

## Endpoint Operations

```text
sagemaker:CreateEndpoint
sagemaker:UpdateEndpoint
sagemaker:DeleteEndpoint
sagemaker:DescribeEndpoint
sagemaker:InvokeEndpoint
sagemaker:InvokeEndpointAsync
sagemaker:ListEndpoints
```

## Endpoint Configuration

```text
sagemaker:CreateEndpointConfig
sagemaker:DeleteEndpointConfig
sagemaker:DescribeEndpointConfig
sagemaker:ListEndpointConfigs
```

## Models

```text
sagemaker:CreateModel
sagemaker:DeleteModel
sagemaker:DescribeModel
sagemaker:ListModels
```

## Training

```text
sagemaker:CreateTrainingJob
sagemaker:DescribeTrainingJob
sagemaker:StopTrainingJob
sagemaker:ListTrainingJobs
```

## Processing

```text
sagemaker:CreateProcessingJob
sagemaker:DescribeProcessingJob
sagemaker:StopProcessingJob
sagemaker:ListProcessingJobs
```

## Batch Transform

```text
sagemaker:CreateTransformJob
sagemaker:DescribeTransformJob
sagemaker:StopTransformJob
sagemaker:ListTransformJobs
```

## Async Inference

```text
sagemaker:InvokeEndpointAsync
```

## SageMaker Domains / Studio

```text
sagemaker:CreateDomain
sagemaker:DescribeDomain
sagemaker:DeleteDomain
sagemaker:ListDomains
```

---

# 6. SNS — `sns:*`

SNS is used for notifications, topics, publishing, and subscriptions.

## Publish

```text
sns:Publish
```

## Topics

```text
sns:CreateTopic
sns:DeleteTopic
sns:GetTopicAttributes
sns:SetTopicAttributes
sns:ListTopics
```

## Subscriptions

```text
sns:Subscribe
sns:Unsubscribe
sns:ListSubscriptions
sns:ListSubscriptionsByTopic
sns:GetSubscriptionAttributes
sns:SetSubscriptionAttributes
```

## Common Application Permission

If an application only needs to send notifications:

```text
sns:Publish
```

---

# 7. Lambda — `lambda:*`

Lambda is used for serverless functions.

## Invoke

```text
lambda:InvokeFunction
```

## Function Information

```text
lambda:GetFunction
lambda:GetFunctionConfiguration
lambda:ListFunctions
```

## Create / Update / Delete

```text
lambda:CreateFunction
lambda:UpdateFunctionCode
lambda:UpdateFunctionConfiguration
lambda:DeleteFunction
```

## Versions and Aliases

```text
lambda:PublishVersion
lambda:ListVersionsByFunction
lambda:CreateAlias
lambda:UpdateAlias
lambda:DeleteAlias
lambda:GetAlias
```

## Concurrency

```text
lambda:GetFunctionConcurrency
lambda:PutFunctionConcurrency
lambda:DeleteFunctionConcurrency
```

## Event Source Mappings

```text
lambda:CreateEventSourceMapping
lambda:DeleteEventSourceMapping
lambda:GetEventSourceMapping
lambda:ListEventSourceMappings
lambda:UpdateEventSourceMapping
```

## Function Permissions

```text
lambda:AddPermission
lambda:RemovePermission
lambda:GetPolicy
```

---

# Quick Reference Table

| Service | Action | Meaning |
|---|---|---|
| S3 | `s3:GetObject` | Read/download an object |
| S3 | `s3:PutObject` | Upload an object |
| S3 | `s3:DeleteObject` | Delete an object |
| S3 | `s3:ListBucket` | List objects in a bucket |
| EC2 | `ec2:StartInstances` | Start EC2 instance |
| EC2 | `ec2:StopInstances` | Stop EC2 instance |
| EC2 | `ec2:TerminateInstances` | Terminate EC2 instance |
| EC2 | `ec2:DescribeInstances` | Get EC2 instance information |
| ECR | `ecr:GetAuthorizationToken` | Authenticate Docker with ECR |
| ECR | `ecr:PutImage` | Push image to ECR |
| ECR | `ecr:BatchGetImage` | Retrieve image from ECR |
| RDS | `rds:DescribeDBInstances` | Get database information |
| RDS | `rds:CreateDBInstance` | Create database instance |
| RDS | `rds:ModifyDBInstance` | Modify database instance |
| RDS | `rds:DeleteDBInstance` | Delete database instance |
| SageMaker | `sagemaker:InvokeEndpoint` | Run model inference |
| SageMaker | `sagemaker:CreateEndpoint` | Create inference endpoint |
| SageMaker | `sagemaker:UpdateEndpoint` | Update inference endpoint |
| SageMaker | `sagemaker:DeleteEndpoint` | Delete inference endpoint |
| SNS | `sns:Publish` | Send notification/message |
| SNS | `sns:CreateTopic` | Create SNS topic |
| SNS | `sns:Subscribe` | Subscribe to a topic |
| Lambda | `lambda:InvokeFunction` | Execute Lambda function |
| Lambda | `lambda:CreateFunction` | Create Lambda function |
| Lambda | `lambda:UpdateFunctionCode` | Deploy/update Lambda code |
| Lambda | `lambda:DeleteFunction` | Delete Lambda function |

---

# Understanding IAM Wildcards

## All actions

```json
"Action": "s3:*"
```

Means all S3 actions.

## Multiple specific actions

```json
"Action": [
    "s3:GetObject",
    "s3:PutObject",
    "s3:DeleteObject"
]
```

Means only those three S3 actions.

## Action prefix wildcard

```text
ec2:Describe*
```

Matches actions such as:

```text
ec2:DescribeInstances
ec2:DescribeVolumes
ec2:DescribeSecurityGroups
ec2:DescribeSubnets
```

Other examples:

```text
sagemaker:Describe*
rds:Describe*
lambda:Get*
```

---

# Mental Map

```text
S3
└── Files / Objects / Buckets

EC2
└── Virtual Servers / Networking / Volumes

ECR
└── Docker / Container Images

RDS
└── Managed Databases

SageMaker
└── ML Models / Training / Inference Endpoints

SNS
└── Notifications / Messages / Topics

Lambda
└── Serverless Functions
```

## How to Read an IAM Action

```text
s3:GetObject
│  │
│  └── Action = GetObject
└───── Service = S3
```

Think:

```text
s3:GetObject
→ Read a file from S3

s3:PutObject
→ Upload a file to S3

ecr:PutImage
→ Push a Docker image to ECR

sagemaker:InvokeEndpoint
→ Send an inference request to SageMaker

sns:Publish
→ Send a notification/message

lambda:InvokeFunction
→ Execute a Lambda function

rds:DescribeDBInstances
→ Get information about RDS databases

ec2:StartInstances
→ Start an EC2 instance
```

---

# Important IAM Principle

Prefer the minimum permissions required by the application.

For example, if a backend only needs to invoke a SageMaker endpoint, prefer:

```json
{
  "Effect": "Allow",
  "Action": "sagemaker:InvokeEndpoint",
  "Resource": "arn:aws:sagemaker:REGION:ACCOUNT_ID:endpoint/ENDPOINT_NAME"
}
```

instead of:

```json
{
  "Effect": "Allow",
  "Action": "sagemaker:*",
  "Resource": "*"
}
```

Similarly, if an application only needs to publish to SNS:

```json
{
  "Effect": "Allow",
  "Action": "sns:Publish",
  "Resource": "SNS_TOPIC_ARN"
}
```

This approach follows the IAM principle of least privilege.

---

# Summary

The most important pattern to remember is:

```text
service:action
```

Common services:

```text
s3:*          → Storage
ec2:*         → Servers
ecr:*         → Container images
rds:*         → Databases
sagemaker:*   → Machine learning
sns:*         → Notifications
lambda:*      → Serverless functions
```

Common actions:

```text
Get*       → Read/get information
List*      → List resources
Describe*  → View resource details
Create*    → Create resource
Put*       → Upload/write/configure
Update*    → Update resource
Modify*    → Modify resource
Delete*    → Delete resource
Invoke*    → Execute/call resource
Start*     → Start resource
Stop*      → Stop resource
```

> **Note:** This is a practical cheat sheet, not an exhaustive list of every AWS IAM action. AWS services have many additional actions and new actions can be introduced over time.
