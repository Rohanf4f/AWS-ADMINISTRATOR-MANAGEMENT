# PHASE 6 — AMAZON RDS
## Deep, Practical AWS RDS Mastery

> Goal: understand how to design, deploy, secure, operate, monitor, back up, troubleshoot, and scale relational databases on AWS.
>
> Main build:
>
> ```text
> Internet
>     ↓
>   ALB
>     ↓
>    EC2
>     ↓
> RDS PostgreSQL
>
> RDS:
> Private Subnet
> Public Access = No
>
> RDS Security Group:
> PostgreSQL 5432
> Source = Application/EC2 Security Group
> ```
>
> The central security principle is:
>
> **The application should be allowed to reach the database; the public internet should not be allowed to reach the database.**

---

# 1. What RDS Actually Is

Amazon RDS is a managed relational database service.

Instead of manually installing PostgreSQL/MySQL on an EC2 server and managing:

- OS patches
- database installation
- storage configuration
- backups
- failover infrastructure
- monitoring
- maintenance
- replication setup

AWS manages much of the underlying infrastructure for you.

You still manage the database-level concerns such as:

- schemas
- tables
- indexes
- SQL
- users/roles
- queries
- application connection behavior
- database configuration
- data modeling

Think:

```text
You manage
    ↓
Database + schema + SQL + users + application behavior

AWS manages
    ↓
Managed DB infrastructure + storage + backups + HA features
```

Official documentation:

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/

---

# 2. RDS Mental Model

Do not think of RDS as simply:

```text
"EC2 but with SQL"
```

Instead:

```text
VPC
 │
 ├── Public Subnets
 │      └── ALB
 │
 ├── Private Application Subnets
 │      └── EC2 / ECS / application
 │
 └── Private Database Subnets
        └── RDS
```

Traffic:

```text
Internet
   │
   ▼
Internet Gateway
   │
   ▼
ALB
   │
   ▼
EC2
   │
   │ TCP 5432
   ▼
RDS PostgreSQL
```

The database should normally have:

```text
Publicly accessible = No
```

and should be reachable through controlled private networking.

AWS documents the private web-server/database pattern as a public web tier communicating with a private DB tier. citeturn1search18turn0search13

---

# 3. What You Need to Understand Before RDS

Before building RDS, make sure you understand:

```text
VPC
Subnet
Route Table
Security Group
Availability Zone
Private Subnet
DNS
EC2
IAM
KMS
CloudWatch
```

Especially:

```text
Security Group
```

because most RDS connectivity problems are ultimately networking/security configuration problems.

---

# 4. RDS Database Engines

RDS supports multiple relational engines.

For this phase focus on:

```text
PostgreSQL
MySQL
Aurora PostgreSQL
Aurora MySQL
```

You do not need to master every RDS engine immediately.

---

# 5. PostgreSQL

PostgreSQL is a powerful open-source relational database.

Example:

```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255) UNIQUE
);
```

Application:

```text
Node.js / Python / Java / Go
            │
            ▼
       PostgreSQL
```

Default PostgreSQL port:

```text
5432
```

---

# 6. MySQL

MySQL is another popular relational database.

Default MySQL port:

```text
3306
```

Example:

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    email VARCHAR(255) UNIQUE
);
```

Application:

```text
Backend
   │
   ▼
MySQL
```

---

# 7. PostgreSQL vs MySQL

For AWS learning, understand the operational concepts rather than trying to memorize every SQL difference.

| Topic | PostgreSQL | MySQL |
|---|---|---|
| Default port | 5432 | 3306 |
| Type | Relational | Relational |
| RDS supported | Yes | Yes |
| Aurora compatible | Aurora PostgreSQL | Aurora MySQL |
| Strong ecosystem | Backend/data-heavy workloads | Web/backend workloads |
| Configuration | Parameter groups | Parameter groups |
| Read replicas | Yes | Yes |
| Multi-AZ | Yes | Yes |

Your main Phase 6 project uses:

```text
RDS PostgreSQL
```

---

# 8. What Is Aurora?

Amazon Aurora is a managed relational database engine that is part of Amazon RDS.

Aurora is compatible with:

```text
Aurora PostgreSQL
Aurora MySQL
```

The architecture is different from a normal RDS DB instance.

Normal RDS mental model:

```text
DB Instance
    │
    └── Database storage
```

Aurora:

```text
              Aurora Cluster
                    │
          ┌─────────┴─────────┐
          │                   │
       Writer               Reader
          │                   │
          └─────────┬─────────┘
                    │
             Shared cluster
                storage
```

Aurora separates database compute instances from its distributed storage architecture.

AWS documents the Aurora cluster as a cluster volume plus DB instances. The cluster storage spans multiple Availability Zones. citeturn0search0turn0search1

---

# 9. Aurora Writer and Reader

Aurora normally has:

```text
Writer
  ↓
Read/Write

Reader
  ↓
Read
```

Example:

```text
Application
    │
    ├── INSERT ──→ Writer
    │
    ├── UPDATE ──→ Writer
    │
    └── SELECT ──→ Reader
```

Aurora Replicas can be used for read scaling and availability.

AWS currently documents up to 15 Aurora Replicas per Aurora DB cluster. citeturn0search0turn0search2

---

# 10. RDS DB Instance vs Aurora Cluster

Important distinction:

## Standard RDS

```text
RDS PostgreSQL
      │
      └── DB Instance
```

## Aurora

```text
Aurora PostgreSQL Cluster
      │
      ├── Writer instance
      ├── Reader instance
      └── Cluster storage
```

Do not use the terms interchangeably.

---

# 11. RDS Networking

RDS runs inside a VPC.

Example:

```text
VPC
10.0.0.0/16

Public
10.0.1.0/24
10.0.2.0/24

Private App
10.0.11.0/24
10.0.12.0/24

Private DB
10.0.21.0/24
10.0.22.0/24
```

Architecture:

```text
                Internet
                   │
                   ▼
             Internet Gateway
                   │
                   ▼
             Public Subnets
                   │
                   ▼
                  ALB
                   │
                   ▼
          Private App Subnets
                   │
                   ▼
                 EC2
                   │
             TCP 5432
                   │
                   ▼
          Private DB Subnets
                   │
                   ▼
            RDS PostgreSQL
```

---

# 12. Public Subnet vs Private Subnet

A subnet is not automatically public because you named it:

```text
Public-Subnet
```

It is public because its route table provides a route to an Internet Gateway.

Example:

```text
0.0.0.0/0
    ↓
Internet Gateway
```

Private database subnet:

```text
No direct route to Internet Gateway
```

This is why naming alone is not security.

---

# 13. DB Subnet Group

A DB subnet group is a collection of subnets that RDS can use for database placement.

Example:

```text
DB Subnet Group
│
├── Private-DB-A
│   AZ-a
│
└── Private-DB-B
    AZ-b
```

AWS requires a DB subnet group to cover subnets in at least two Availability Zones. AWS recommends including a subnet in every AZ in the Region where practical. citeturn1search8turn1search4

Why?

Because the database infrastructure may need to use another AZ for:

- Multi-AZ
- failover
- maintenance
- high availability

---

# 14. DB Subnet Group Mental Model

Think:

```text
VPC
 │
 ├── Public Subnet A
 ├── Public Subnet B
 │
 ├── Private App A
 ├── Private App B
 │
 ├── Private DB A ──┐
 │                  │
 └── Private DB B ──┤
                    ▼
              DB Subnet Group
                    │
                    ▼
                  RDS
```

The DB subnet group does not itself provide access control.

It tells RDS:

```text
"These are the subnets where my database networking can be placed."
```

Security Groups control network access.

---

# 15. Security Group for RDS

Create:

```text
RDS-Postgres-SG
```

Inbound:

```text
Type: PostgreSQL
Protocol: TCP
Port: 5432
Source: Application SG
```

Do NOT normally use:

```text
0.0.0.0/0
```

for database access.

---

# 16. Why Security Group Referencing Is Better

Suppose:

```text
EC2 SG:
app-server-sg

RDS SG:
database-sg
```

RDS inbound rule:

```text
TCP 5432
Source = app-server-sg
```

This means:

```text
Instances associated with app-server-sg
             │
             ▼
       allowed to connect
             │
             ▼
     RDS database on 5432
```

This is better than:

```text
0.0.0.0/0
```

because you are expressing the actual trust relationship:

```text
Application → Database
```

rather than:

```text
Entire Internet → Database
```

AWS documents security groups as controlling traffic to and from DB instances, with rules that can specify another security group as the source. citeturn0search13turn1search11

---

# 17. The Most Important RDS Security Rule

Your database security group should normally look conceptually like:

```text
Inbound

PostgreSQL
TCP
5432
Source:
Application-SG
```

Not:

```text
PostgreSQL
TCP
5432
Source:
0.0.0.0/0
```

The application should be the trusted network identity.

---

# 18. Why Port 5432 Is Not Enough

A security group rule:

```text
5432 from Application-SG
```

does not mean:

```text
"Anyone can connect to PostgreSQL."
```

It means:

```text
Traffic arriving from resources matching Application-SG
can reach TCP port 5432.
```

You still need:

```text
PostgreSQL running
+
correct hostname
+
correct port
+
correct database
+
valid credentials
+
correct PostgreSQL permissions
```

Therefore database access has multiple layers.

---

# 19. RDS Access Layers

Think about:

```text
Layer 1
VPC routing

Layer 2
Security Group

Layer 3
RDS public accessibility

Layer 4
TLS/network encryption

Layer 5
Database authentication

Layer 6
Database authorization

Layer 7
Application-level permissions
```

For example:

```text
EC2
 │
 │ Route?
 ▼
VPC
 │
 │ SG?
 ▼
RDS
 │
 │ PostgreSQL authentication?
 ▼
Database user
 │
 │ SQL permissions?
 ▼
Table
```

This is extremely important for troubleshooting.

---

# 20. Public Accessibility

When creating RDS you will see:

```text
Public access
```

For this project:

```text
No
```

Conceptually:

```text
Internet
   X
   │
   ▼
RDS
```

Instead:

```text
Internet
   │
   ▼
ALB
   │
   ▼
EC2
   │
   ▼
RDS
```

AWS exposes the Public accessibility setting for RDS DB instances in a VPC. citeturn0search13

---

# 21. Why Database Should Be Private

Suppose you configure:

```text
RDS public
+
5432
+
0.0.0.0/0
```

Now the database endpoint may be reachable from the public internet.

That creates unnecessary attack surface.

A better architecture:

```text
Internet
    │
    ▼
   ALB
    │
    ▼
  EC2
    │
    ▼
Private RDS
```

Only the application tier needs database access.

---

# 22. Multi-AZ

Multi-AZ means RDS maintains database infrastructure across Availability Zones for high availability.

For the traditional Multi-AZ DB instance deployment:

```text
AZ-A
  │
  └── Primary

AZ-B
  │
  └── Standby
```

The standby is not normally used to serve application read traffic.

AWS describes Multi-AZ DB instance deployments as maintaining a standby in another AZ for redundancy and failover. citeturn0search8

---

# 23. Multi-AZ Is Mainly About Availability

Think:

```text
Primary fails
     │
     ▼
RDS failover
     │
     ▼
Standby becomes primary
```

It is not the same thing as:

```text
Read scaling
```

This distinction is critical.

---

# 24. Multi-AZ vs Read Replica

## Multi-AZ

Primary purpose:

```text
High availability
Failover
```

## Read Replica

Primary purpose:

```text
Read scaling
```

Mental model:

```text
Multi-AZ

Primary
   │
   └── Standby
        ↑
     failover
```

Read replica:

```text
Primary
   │
   └── Read Replica
          ↑
       SELECT
```

RDS PostgreSQL read replicas use asynchronous PostgreSQL streaming replication. citeturn0search10

---

# 25. Multi-AZ DB Cluster

AWS also provides a Multi-AZ DB cluster deployment.

Conceptually:

```text
AZ-A
 Writer

AZ-B
 Reader

AZ-C
 Reader
```

A Multi-AZ DB cluster has one writer and two readable replicas across three AZs. citeturn0search7

This differs from the traditional:

```text
Primary + standby
```

model.

For your learning path, understand both:

```text
Multi-AZ DB instance
Multi-AZ DB cluster
```

before choosing one for a production workload.

---

# 26. Read Replicas

A read replica is a separate DB instance that receives replicated changes from a source.

Example:

```text
Primary
  │
  ├────→ Read Replica 1
  │
  ├────→ Read Replica 2
  │
  └────→ Read Replica 3
```

Application:

```text
Writes
   ↓
Primary

Reads
   ↓
Reader/Replica
```

The application must understand which connection endpoint should receive writes and which can receive read traffic.

---

# 27. Read Replica Is Not Backup

This is a very important distinction.

Read replica:

```text
Replication
```

Backup:

```text
Recovery
```

If bad application code executes:

```sql
DELETE FROM users;
```

the delete can propagate to a replica.

So:

```text
Replica ≠ backup
```

You need:

```text
Automated backups
+
Snapshots
```

for recovery.

---

# 28. Automated Backups

RDS automated backups support recovery to a point in time within the backup retention period.

Example:

```text
09:00
10:00
11:00
12:00
13:00
```

Suppose bad data was introduced at:

```text
12:37
```

You may restore to a point before the bad change, within the available retention window.

AWS allows an RDS DB instance to be restored to a point in time within its backup retention period. citeturn1search12

---

# 29. Backup Retention

For a normal RDS DB instance:

```text
0–35 days
```

is the documented backup retention range.

Console-created DB instances default to seven days unless changed.

Setting:

```text
0
```

disables automated backups.

AWS documents these retention settings and their operational implications. citeturn1search2

For production:

```text
Do not blindly accept defaults.
```

Choose retention based on:

```text
RPO
Compliance
Recovery requirements
Cost
Business requirements
```

---

# 30. RPO

RPO:

```text
Recovery Point Objective
```

Question:

> How much data can the business afford to lose?

Example:

```text
RPO = 15 minutes
```

means your recovery strategy should aim to limit data loss to roughly that window.

---

# 31. RTO

RTO:

```text
Recovery Time Objective
```

Question:

> How quickly must the service be restored?

Example:

```text
RTO = 30 minutes
```

means the recovery process should target restoring service within that time.

---

# 32. RPO vs RTO

```text
RPO
↓
How much data can we lose?

RTO
↓
How much downtime can we tolerate?
```

Database architecture should be designed around these business requirements.

---

# 33. Manual Snapshot

A snapshot is a point-in-time backup you explicitly create.

Example:

```text
RDS
 │
 └── Create snapshot
          │
          ▼
      DB Snapshot
```

Use snapshots before significant operations such as:

```text
Major migration
Major configuration change
Database upgrade
Experimental change
```

But do not rely only on manual snapshots for operational recovery.

---

# 34. Automated Backup vs Snapshot

| Feature | Automated Backup | Manual Snapshot |
|---|---|---|
| Purpose | Continuous recovery capability | Explicit backup |
| Point-in-time restore | Yes | No, snapshot point |
| User initiated | No | Yes |
| Retention | Configurable | Until deleted |
| Useful before major changes | Sometimes | Yes |
| Disaster recovery | Yes | Yes |

---

# 35. Encryption

RDS encryption protects data at rest.

Encrypted data can include:

```text
Database storage
Automated backups
Snapshots
Read replicas
Logs
```

RDS encryption uses AWS KMS. AWS documents encryption at rest for RDS resources and AES-256 server-side encryption. citeturn0search14

---

# 36. Encryption at Rest

Conceptually:

```text
Application
     │
     ▼
RDS
     │
     ▼
Encrypted storage
     │
     ▼
KMS key
```

For company environments, understand:

```text
AWS-owned/default encryption
vs
customer managed KMS key
```

Customer-managed keys provide additional control over:

- key policy
- permissions
- rotation configuration
- auditing
- cross-account usage patterns

---

# 37. Encryption in Transit

Encryption at rest is not enough.

You should also understand:

```text
EC2
 │
 │ TLS
 ▼
RDS PostgreSQL
```

The PostgreSQL client/application can be configured to use SSL/TLS.

Think:

```text
At rest
   ↓
Data stored on infrastructure

In transit
   ↓
Data moving between application and database
```

---

# 38. RDS Parameter Groups

A DB parameter group contains engine configuration parameters.

Example categories:

```text
Connection settings
Memory-related settings
Logging settings
Query behavior
Engine-specific settings
```

AWS describes DB parameter groups as containers for engine configuration values applied to DB instances. citeturn0search3

---

# 39. Default vs Custom Parameter Group

Do not blindly modify production databases using random settings.

Create a custom parameter group:

```text
company-postgres-prod
```

instead of treating the default group as your permanent application configuration.

Mental model:

```text
PostgreSQL
     │
     ▼
Parameter Group
     │
     ▼
Database engine configuration
```

---

# 40. Dynamic vs Static Parameters

Some database parameters can be changed dynamically.

Others require a restart.

Conceptually:

```text
Dynamic
   ↓
Apply without restart

Static
   ↓
Requires restart/reboot
```

Always check the parameter's apply type before changing production configuration.

---

# 41. Parameter Group Example

Imagine:

```text
log_statement
log_min_duration_statement
```

These can affect database logging and troubleshooting.

You might have:

```text
dev-postgres-params
staging-postgres-params
prod-postgres-params
```

with different operational requirements.

Do not copy production tuning values into every environment without testing.

---

# 42. Option Groups

Option groups are engine-specific configuration containers used by some RDS engines.

Important:

```text
PostgreSQL does not use option groups.
```

PostgreSQL uses extensions/modules for additional functionality.

AWS explicitly documents that PostgreSQL does not use option groups. citeturn0search15

Therefore:

```text
RDS PostgreSQL
    ↓
Parameter Group
    ↓
Extensions/modules
```

not:

```text
PostgreSQL
    ↓
Option Group
```

You should still learn Option Groups because other RDS engines use them.

---

# 43. Monitoring RDS

You need to monitor:

```text
CPU
Memory-related pressure
Storage
Connections
IOPS
Latency
Database load
Replica lag
Free storage
Read/write throughput
Database events
Logs
```

CloudWatch receives RDS metrics automatically, generally at one-minute intervals for standard RDS metrics. citeturn1search1

---

# 44. Important RDS CloudWatch Metrics

Examples:

```text
CPUUtilization
DatabaseConnections
FreeStorageSpace
FreeableMemory
ReadIOPS
WriteIOPS
ReadLatency
WriteLatency
NetworkReceiveThroughput
NetworkTransmitThroughput
ReplicaLag
```

Do not monitor only:

```text
CPU
```

A database can have:

```text
CPU = 30%
```

while still suffering from:

```text
slow queries
locks
connection exhaustion
I/O pressure
storage pressure
```

---

# 45. Enhanced Monitoring

Enhanced Monitoring gives deeper operating-system-level visibility for RDS DB instances.

It can provide:

```text
CPU
Memory
Processes
Disk activity
Filesystem information
OS-level metrics
```

AWS documents Enhanced Monitoring as providing real-time OS-level metrics, with granularity down to one second. citeturn1search6turn1search7

---

# 46. Performance Insights — Important Current Change

Your Phase 6 syllabus includes:

```text
Performance Insights
```

You should learn the concept, but note an important current AWS change:

AWS announced the end of life of the Performance Insights feature on:

```text
July 31, 2026
```

AWS has migrated this functionality into:

```text
CloudWatch Database Insights
```

As of this curriculum's current date, you should therefore learn:

```text
Performance Insights concepts
        ↓
Database Insights implementation
```

AWS documents Database Insights as the current monitoring/troubleshooting experience for RDS and Aurora. citeturn1search0turn1search9

---

# 47. Database Load

One of the most useful database performance concepts is:

```text
DB Load
```

Think:

```text
How much active database work is happening?
```

Database Insights lets you investigate load by dimensions such as:

```text
Waits
SQL statements
Hosts
Users
```

This is more useful than looking only at CPU.

AWS documents Database Insights as the current way to analyze database load and troubleshoot RDS/Aurora at scale. citeturn1search9

---

# 48. Example Performance Problem

Suppose:

```text
CPU = 35%
```

but users complain:

```text
API is slow
```

Do not immediately increase:

```text
DB instance size
```

Investigate:

```text
DB Load
   ↓
Wait events
   ↓
SQL statements
   ↓
Slow query
   ↓
Missing index
```

Possible solution:

```text
CREATE INDEX ...
```

rather than:

```text
Buy a bigger DB
```

This is an important database-engineering mindset.

---

# 49. RDS Logs

Depending on engine and configuration, RDS can expose database logs such as:

```text
Error logs
Slow query logs
General logs
PostgreSQL logs
```

You can integrate relevant logs with CloudWatch Logs.

Use logs to answer:

```text
Why did the database reject the connection?

Why is this query slow?

Why did the database restart?

Why is authentication failing?
```

AWS's RDS monitoring documentation covers CloudWatch metrics, logs, Enhanced Monitoring, and Database Insights. citeturn1search10

---

# 50. RDS Events

RDS generates events for important database operations and conditions.

Examples:

```text
Maintenance
Backup
Failover
Configuration changes
Storage events
Availability events
```

Monitor important events using:

```text
RDS
+
CloudWatch
+
SNS/EventBridge where appropriate
```

---

# 51. RDS Architecture for This Phase

Use:

```text
                         INTERNET
                            │
                            ▼
                    Internet Gateway
                            │
                            ▼
                     Public Subnets
                            │
                            ▼
                          ALB
                            │
                            ▼
                Private Application Subnets
                            │
                       ┌────┴────┐
                       │         │
                      EC2       EC2
                       │         │
                       └────┬────┘
                            │
                       TCP 5432
                            │
                            ▼
                 Private Database Subnets
                       ┌────┴────┐
                       │         │
                     AZ-A       AZ-B
                       │         │
                       └────┬────┘
                            │
                       RDS PostgreSQL
```

Security:

```text
ALB SG
  ↓
EC2 SG
  ↓
RDS SG
```

---

# 52. Three-Tier Security Group Design

Create three security groups.

## ALB SG

```text
Inbound:
80
443
Source:
Internet
```

## Application SG

```text
Inbound:
80
443
Source:
ALB SG
```

## RDS SG

```text
Inbound:
5432
Source:
Application SG
```

Mental model:

```text
Internet
   │
   ▼
ALB SG
   │
   ▼
Application SG
   │
   ▼
RDS SG
```

Each layer trusts only the layer immediately above it.

---

# 53. Why Not Allow ALB → RDS?

The ALB should not talk directly to PostgreSQL.

Correct:

```text
ALB
 ↓
EC2
 ↓
RDS
```

Incorrect for this application architecture:

```text
ALB
 ↓
RDS
```

The application logic belongs on the application tier.

---

# 54. Application Connection

Your application will eventually have something like:

```text
DB_HOST=my-postgres.xxxxxx.ap-south-1.rds.amazonaws.com
DB_PORT=5432
DB_NAME=companydb
DB_USER=app_user
DB_PASSWORD=********
```

Do not hard-code:

```text
DB_PASSWORD
```

inside source code.

Use:

```text
Secrets Manager
```

or an appropriate secure configuration mechanism.

---

# 55. RDS Endpoint

RDS gives you a DNS endpoint.

Example:

```text
my-postgres.xxxxxx.ap-south-1.rds.amazonaws.com
```

Your EC2 application connects to:

```text
hostname
+
port
```

not directly to a hard-coded database IP.

Why?

Because RDS manages the underlying infrastructure and failover behavior.

Your application should use the database endpoint provided by AWS.

---

# 56. RDS Connection Flow

Example:

```text
User
 │
 ▼
ALB
 │
 ▼
EC2
 │
 │ DNS lookup
 ▼
RDS endpoint
 │
 │ TCP 5432
 ▼
RDS
 │
 │ PostgreSQL authentication
 ▼
Database
```

---

# 57. DNS Is Important

When debugging:

```text
Can EC2 resolve the RDS hostname?
```

Test from EC2:

```bash
nslookup my-postgres.xxxxxx.ap-south-1.rds.amazonaws.com
```

or:

```bash
dig my-postgres.xxxxxx.ap-south-1.rds.amazonaws.com
```

Then test TCP connectivity:

```bash
nc -vz my-postgres.xxxxxx.ap-south-1.rds.amazonaws.com 5432
```

For PostgreSQL:

```bash
psql -h my-postgres.xxxxxx.ap-south-1.rds.amazonaws.com \
     -p 5432 \
     -U app_user \
     -d companydb
```

---

# 58. RDS Troubleshooting Framework

When EC2 cannot connect to RDS, troubleshoot in this order.

## Step 1 — DNS

```text
Can EC2 resolve the RDS endpoint?
```

## Step 2 — Network

```text
Are EC2 and RDS in reachable subnets/VPC?
```

## Step 3 — Security Group

```text
Does RDS SG allow TCP 5432 from EC2 SG?
```

## Step 4 — Public Accessibility

For this architecture:

```text
No
```

is expected.

## Step 5 — Port

PostgreSQL:

```text
5432
```

MySQL:

```text
3306
```

## Step 6 — Credentials

Check:

```text
username
password
database name
```

## Step 7 — PostgreSQL permissions

The network connection may work while the database user still lacks permissions.

---

# 59. The Difference Between Network and Database Authorization

Suppose:

```text
EC2 → RDS
```

is allowed by the security group.

That means:

```text
Network access allowed
```

It does NOT mean:

```text
PostgreSQL user authorized
```

You can have:

```text
Network = OK
Database authentication = FAIL
```

or:

```text
Network = FAIL
Database authentication = never reached
```

This distinction is essential.

---

# 60. RDS Deletion Protection

For important environments, consider:

```text
Deletion protection = Enabled
```

This helps prevent accidental deletion through normal deletion workflows.

For production database infrastructure, deletion protection is an important safety control.

---

# 61. Storage

RDS storage configuration affects:

```text
capacity
performance
cost
IOPS
throughput
```

Do not choose storage only by:

```text
"How many GB do I need?"
```

Also consider:

```text
How many IOPS?
What latency?
How much throughput?
How fast will data grow?
```

---

# 62. Storage Autoscaling

RDS can automatically increase storage when configured appropriately.

Conceptually:

```text
Current:
100 GB

Usage increases

RDS detects threshold

        ↓

Storage increases

        ↓

Database continues operating
```

This can reduce the risk of running out of storage.

Still monitor:

```text
FreeStorageSpace
```

---

# 63. Database Instance Class

RDS instance class determines compute resources.

Think:

```text
CPU
Memory
Network
Database workload
```

Example:

```text
db.t* family
db.m* family
db.r* family
```

Do not memorize every instance class.

Learn how to evaluate:

```text
CPU requirement
Memory requirement
Network requirement
I/O requirement
Cost
```

---

# 64. Scaling RDS

There are multiple scaling dimensions.

## Vertical scaling

Increase instance size:

```text
db.m6g.large
        ↓
db.m6g.xlarge
```

More:

```text
CPU
Memory
```

## Read scaling

Use:

```text
Read Replicas
```

## Application scaling

Use:

```text
Connection pooling
RDS Proxy
```

## Storage scaling

Use:

```text
Storage autoscaling
```

---

# 65. Connection Management

One common database problem:

```text
Too many connections
```

Suppose:

```text
100 EC2 instances
```

and each opens:

```text
100 DB connections
```

Potentially:

```text
10,000 connections
```

The database may struggle even when CPU is not extremely high.

Therefore:

```text
Connection pooling
```

is important.

For some architectures, AWS RDS Proxy can help pool and reuse database connections.

---

# 66. Maintenance

RDS handles much of the infrastructure maintenance, but you still need to understand:

```text
Maintenance windows
Engine versions
Minor upgrades
Major upgrades
Parameter compatibility
Application compatibility
Failover
Backups
```

Never treat:

```text
"Managed service"
```

as:

```text
"No operations required."
```

---

# 67. Version Upgrades

Database upgrades can affect:

```text
SQL behavior
Extensions
Parameters
Performance
Application drivers
Query plans
```

Before production upgrades:

```text
Backup
 ↓
Test
 ↓
Staging upgrade
 ↓
Application tests
 ↓
Production plan
```

---

# 68. Snapshot Before Major Changes

A useful operational pattern:

```text
Before migration
       ↓
Create snapshot
       ↓
Perform change
       ↓
Validate
       ↓
Keep rollback plan
```

Do not assume:

```text
"RDS is managed, so rollback is automatic."
```

---

# 69. Security Checklist

For a private PostgreSQL deployment:

```text
[ ] Public accessibility = No
[ ] Private DB subnets
[ ] DB subnet group spans at least 2 AZs
[ ] RDS SG created
[ ] Port 5432 only
[ ] Source = Application SG
[ ] No 0.0.0.0/0
[ ] Encryption enabled
[ ] KMS understood
[ ] Automated backups enabled
[ ] Backup retention selected
[ ] Deletion protection considered
[ ] Monitoring enabled
[ ] Database logs considered
[ ] Secrets not hard-coded
[ ] TLS considered
[ ] Parameter group reviewed
```

---

# 70. BUILD — RDS PostgreSQL

Now build the actual architecture.

## Target

```text
Internet
    ↓
   ALB
    ↓
   EC2
    ↓
RDS PostgreSQL
```

Database:

```text
Private
No Public Access
Encrypted
Backups enabled
```

---

# 71. Build Prerequisites

You should already have:

```text
VPC
Public subnets
Private app subnets
Private DB subnets
Route tables
Internet Gateway
Security groups
EC2
ALB
```

If not, use your Phase 3 and Phase 4 architecture.

---

# 72. Step 1 — Create DB Subnets

Use two private database subnets:

```text
Private-DB-A
AZ-A

Private-DB-B
AZ-B
```

Example:

```text
10.0.21.0/24
10.0.22.0/24
```

Ensure they belong to the same VPC.

---

# 73. Step 2 — Create DB Subnet Group

Open:

```text
AWS Console
    ↓
RDS
    ↓
Subnet groups
```

Choose:

```text
Create DB subnet group
```

Example:

```text
Name:
company-db-subnet-group

Description:
Private subnet group for company databases

VPC:
company-vpc
```

Select:

```text
AZ-A
Private-DB-A

AZ-B
Private-DB-B
```

AWS requires at least two AZs in the DB subnet group. citeturn1search8

---

# 74. Step 3 — Create RDS Security Group

Go to:

```text
VPC
 ↓
Security Groups
 ↓
Create security group
```

Name:

```text
company-rds-postgres-sg
```

VPC:

```text
company-vpc
```

Inbound:

```text
Type:
PostgreSQL

Port:
5432

Source:
Application Security Group
```

Do not enter:

```text
0.0.0.0/0
```

---

# 75. Step 4 — Understand the SG Rule

Suppose:

```text
EC2:
company-app-sg
```

RDS:

```text
company-rds-postgres-sg
```

RDS rule:

```text
5432
Source:
company-app-sg
```

Traffic:

```text
EC2
 │
 │ TCP 5432
 ▼
RDS SG
 │
 ▼
RDS PostgreSQL
```

This creates an explicit application-to-database trust relationship.

---

# 76. Step 5 — Create PostgreSQL

Open:

```text
RDS
 ↓
Databases
 ↓
Create database
```

Choose:

```text
Standard create
```

Engine:

```text
PostgreSQL
```

---

# 77. Step 6 — Choose Template

For learning:

```text
Dev/Test
```

or the smallest reasonable configuration available for your lab.

For production:

```text
Production
```

does not automatically mean:

```text
Correct architecture
```

You must still configure:

```text
network
security
backup
HA
monitoring
encryption
```

---

# 78. Step 7 — Credentials

Set:

```text
Master username
```

Use a strong password.

Do not put credentials into:

```text
Git
GitHub
source code
Docker image
frontend code
Slack
README
```

For production, prefer a secrets-management workflow.

---

# 79. Step 8 — Connectivity

Choose:

```text
VPC:
company-vpc
```

DB subnet group:

```text
company-db-subnet-group
```

Public access:

```text
No
```

Security group:

```text
company-rds-postgres-sg
```

Port:

```text
5432
```

---

# 80. Step 9 — Availability

For the first lab:

```text
Single-AZ
```

can be acceptable to understand basic connectivity and cost.

Then build a second lab:

```text
Multi-AZ
```

to understand failover.

Do not confuse:

```text
learning lab
```

with:

```text
production architecture
```

---

# 81. Step 10 — Encryption

Enable:

```text
Encryption
```

Choose the appropriate KMS key.

For learning:

```text
AWS managed KMS key
```

can be sufficient.

For company environments, understand when a customer-managed KMS key is required.

---

# 82. Step 11 — Backup

Configure:

```text
Automated backups
```

Choose an appropriate retention period.

For your lab:

```text
7 days
```

is a reasonable starting point.

For production:

```text
RPO
RTO
compliance
business requirements
```

should determine the strategy.

---

# 83. Step 12 — Deletion Protection

For an important environment:

```text
Deletion protection:
Enabled
```

For temporary learning resources, you may intentionally leave it disabled so cleanup is easier.

Always know which state you selected.

---

# 84. Step 13 — Monitoring

Review:

```text
CloudWatch
Database Insights
Enhanced Monitoring
Logs
Events
```

Do not enable expensive monitoring features without understanding their pricing and retention.

---

# 85. Step 14 — Create Database

Review:

```text
Engine
Version
Instance class
Storage
VPC
Subnet group
Public access
Security group
Port
Encryption
Backup
Monitoring
Deletion protection
```

Then:

```text
Create database
```

---

# 86. Step 15 — Find the Endpoint

Open:

```text
RDS
 ↓
Databases
 ↓
company-postgres
```

Find:

```text
Endpoint
Port
```

Example:

```text
Endpoint:
company-postgres.xxxxx.ap-south-1.rds.amazonaws.com

Port:
5432
```

---

# 87. Step 16 — Test From EC2

SSH or Session Manager into your EC2 application server.

Check DNS:

```bash
nslookup company-postgres.xxxxx.ap-south-1.rds.amazonaws.com
```

Test port:

```bash
nc -vz company-postgres.xxxxx.ap-south-1.rds.amazonaws.com 5432
```

If successful:

```text
TCP connection works
```

Then test PostgreSQL:

```bash
psql \
  -h company-postgres.xxxxx.ap-south-1.rds.amazonaws.com \
  -p 5432 \
  -U app_user \
  -d companydb
```

---

# 88. What If Connection Fails?

Do not randomly change settings.

Use:

```text
DNS
 ↓
VPC
 ↓
Subnet
 ↓
Route
 ↓
Security Group
 ↓
RDS status
 ↓
Port
 ↓
Credentials
 ↓
Database permissions
```

This is your debugging sequence.

---

# 89. Intentionally Break the Security Group

This is an important lab.

Change:

```text
RDS SG
```

from:

```text
5432
Source = Application SG
```

to:

```text
5432
Source = some incorrect SG
```

Then test:

```bash
nc -vz RDS_ENDPOINT 5432
```

It should fail.

Now restore:

```text
5432
Source = Application SG
```

Test again.

This teaches you the actual effect of a security group rule.

---

# 90. Intentionally Break the Port

Change the RDS security group:

```text
5432
```

to:

```text
3306
```

while the database remains PostgreSQL.

Test:

```bash
nc -vz RDS_ENDPOINT 5432
```

The connection should fail.

Restore:

```text
5432
```

This teaches:

```text
Database engine
+
port
+
SG
```

must agree.

---

# 91. Intentionally Test Public Access

For learning, understand the difference between:

```text
Publicly accessible = No
```

and:

```text
Publicly accessible = Yes
```

Do not expose a real company database to the internet just to experiment.

Use an isolated lab if you need to understand the setting.

---

# 92. Multi-AZ Lab

After the basic RDS project works:

```text
RDS
 ↓
Modify
 ↓
Multi-AZ
```

Observe:

```text
Primary
+
Standby
```

Then learn what happens during:

```text
planned failover
unplanned failure
maintenance
```

The application should use the RDS endpoint rather than hard-coded infrastructure IPs.

---

# 93. Read Replica Lab

Create:

```text
Primary
   │
   └── Read Replica
```

Observe:

```text
replication
replica lag
read-only behavior
```

Test:

```text
SELECT
```

against the replica.

Understand why writes should remain on the primary.

---

# 94. Backup Lab

Create:

```text
Manual snapshot
```

Then restore it as a separate DB instance.

Do not restore over your only production database as a learning exercise.

Test:

```text
Snapshot
 ↓
Restore
 ↓
New DB
 ↓
Connect
 ↓
Verify data
```

---

# 95. Point-in-Time Recovery Lab

Use automated backups.

Create some data:

```sql
INSERT INTO users(name)
VALUES ('Alice');
```

Wait/perform changes.

Then:

```sql
INSERT INTO users(name)
VALUES ('Bob');
```

Then simulate a bad operation:

```sql
DELETE FROM users;
```

Use:

```text
Restore to point in time
```

to create a new DB from a time before the destructive operation.

AWS supports restoring an RDS DB instance to a chosen point in time within the backup retention window. citeturn1search12

---

# 96. Monitoring Lab

Open:

```text
RDS
 ↓
Database
 ↓
Monitoring / Database Insights
```

Observe:

```text
CPU
Connections
Storage
IOPS
Latency
Database load
```

Then create a CloudWatch alarm for an appropriate metric.

Example:

```text
CPUUtilization > 80%
```

But remember:

```text
CPU alarm
```

is not a complete database health strategy.

---

# 97. Database Insights Lab

Use:

```text
CloudWatch
 ↓
Database Insights
```

Investigate:

```text
DB Load
Waits
SQL statements
Hosts
Users
```

Try to answer:

```text
What is consuming database time?
```

This is much closer to real production database troubleshooting than simply checking CPU.

AWS currently documents Database Insights as the replacement/current experience following the Performance Insights transition. citeturn1search0turn1search15

---

# 98. Parameter Group Lab

Create:

```text
custom-postgres-parameters
```

Inspect parameters.

Choose one safe, non-disruptive parameter to understand.

Check:

```text
Dynamic?
Static?
Apply immediately?
Requires reboot?
```

Do not randomly modify production parameters.

---

# 99. Secrets Lab

Your application should not use:

```python
DB_PASSWORD = "mypassword"
```

Instead:

```text
Application
   │
   ▼
Secrets Manager
   │
   ▼
Database credentials
   │
   ▼
RDS
```

The application IAM role gets permission to retrieve the required secret.

This connects Phase 1 IAM with Phase 6 RDS.

---

# 100. RDS IAM Connection

Your architecture now combines:

```text
IAM
+
VPC
+
EC2
+
Security Groups
+
RDS
+
KMS
+
CloudWatch
+
Secrets Manager
```

This is why RDS is an important real-world AWS service.

---

# 101. Company Architecture

A company architecture might look like:

```text
                    Internet
                       │
                       ▼
                 CloudFront
                       │
                       ▼
                     ALB
                       │
              ┌────────┴────────┐
              │                 │
             EC2               EC2
              │                 │
              └────────┬────────┘
                       │
                    App SG
                       │
                    5432
                       │
                       ▼
              RDS PostgreSQL
                 Private DB
                       │
             ┌─────────┴─────────┐
             │                   │
            AZ-A                AZ-B
             │                   │
          Primary              Standby
```

Security:

```text
Internet
   ↓
ALB
   ↓
EC2
   ↓
RDS
```

Not:

```text
Internet
   ↓
RDS
```

---

# 102. RDS Security Layers

Use defense in depth:

```text
Layer 1
Private subnet

Layer 2
No public accessibility

Layer 3
Security Group

Layer 4
Encryption at rest

Layer 5
TLS in transit

Layer 6
Database authentication

Layer 7
Database authorization

Layer 8
Secrets management

Layer 9
CloudTrail / auditing

Layer 10
Monitoring
```

No single control is sufficient.

---

# 103. Common Mistakes

## Mistake 1

```text
RDS SG:
5432
0.0.0.0/0
```

Avoid unless there is a very specific, documented, controlled requirement.

---

## Mistake 2

```text
Publicly accessible = Yes
```

when the application is already inside the same VPC.

Usually unnecessary.

---

## Mistake 3

Thinking:

```text
Multi-AZ = backup
```

Incorrect.

Multi-AZ is primarily an availability/failover mechanism.

---

## Mistake 4

Thinking:

```text
Read Replica = backup
```

Incorrect.

Replication can reproduce unwanted changes.

---

## Mistake 5

Hard-coding:

```text
DB password
```

in source code.

---

## Mistake 6

Using:

```text
IP address
```

instead of the RDS endpoint.

---

## Mistake 7

Monitoring only:

```text
CPU
```

---

## Mistake 8

Changing production parameters without understanding:

```text
static vs dynamic
```

---

## Mistake 9

Deleting a database without checking:

```text
snapshot
backup
deletion protection
```

---

## Mistake 10

Assuming:

```text
AWS manages RDS
=
AWS manages your database operations
```

You still own:

```text
schema
queries
indexes
users
permissions
application behavior
data
```

---

# 104. RDS vs EC2 PostgreSQL

## PostgreSQL on EC2

You manage:

```text
EC2
OS
PostgreSQL installation
patching
storage
backup
replication
failover
monitoring
```

## RDS PostgreSQL

AWS manages much of:

```text
Infrastructure
Managed database service
Backups
HA features
Maintenance mechanisms
Monitoring integration
```

You still manage:

```text
Database
Schema
Queries
Indexes
Users
Application
Permissions
```

---

# 105. RDS vs Aurora

Think:

```text
RDS PostgreSQL
    ↓
Managed PostgreSQL instance

Aurora PostgreSQL
    ↓
Cloud-optimized PostgreSQL-compatible
cluster architecture
```

Aurora is useful to study when you need to understand:

```text
shared/distributed storage
writer
readers
cluster endpoints
read scaling
high availability
```

Aurora's storage is replicated across multiple AZs independently of the number of DB instances. citeturn0search1

---

# 106. Endpoints You Should Know

For standard RDS:

```text
DB endpoint
```

For Aurora:

```text
Cluster endpoint
Reader endpoint
Custom endpoints
```

Conceptually:

```text
Writes
  ↓
Writer / Cluster endpoint

Reads
  ↓
Reader endpoint
```

Do not randomly distribute write traffic to reader endpoints.

---

# 107. Read Scaling Architecture

Example:

```text
                  Application
                       │
              ┌────────┴────────┐
              │                 │
            Writes             Reads
              │                 │
              ▼                 ▼
           Writer          Reader endpoint
                                │
                       ┌────────┼────────┐
                       ▼        ▼        ▼
                     Reader   Reader   Reader
```

This architecture is particularly relevant to Aurora.

---

# 108. Production Database Checklist

Before calling an RDS database production-ready, review:

```text
NETWORK
[ ] Private subnets
[ ] DB subnet group
[ ] Multiple AZs where required
[ ] Correct route tables

ACCESS
[ ] Public access disabled unless specifically required
[ ] RDS SG created
[ ] DB port restricted
[ ] Source SG rather than broad CIDR where possible

ENCRYPTION
[ ] Encryption at rest
[ ] KMS understood
[ ] TLS in transit

BACKUP
[ ] Automated backups
[ ] Retention configured
[ ] PITR understood
[ ] Snapshot strategy
[ ] Restore tested

AVAILABILITY
[ ] Multi-AZ strategy
[ ] Failover tested
[ ] Application uses endpoint

PERFORMANCE
[ ] CloudWatch metrics
[ ] Database Insights
[ ] Logs
[ ] Slow query analysis
[ ] Connection monitoring

OPERATIONS
[ ] Parameter groups
[ ] Maintenance window
[ ] Upgrade strategy
[ ] Deletion protection
[ ] Secrets management

SECURITY
[ ] IAM permissions
[ ] KMS permissions
[ ] Secrets access restricted
[ ] Audit/logging strategy
```

---

# 109. Practical Troubleshooting Decision Tree

If application says:

```text
connection timeout
```

think:

```text
DNS?
  ↓
VPC?
  ↓
Subnet?
  ↓
Route?
  ↓
Security Group?
  ↓
NACL?
  ↓
RDS available?
  ↓
Correct endpoint?
  ↓
Correct port?
```

If application says:

```text
password authentication failed
```

think:

```text
Network probably works
        ↓
Username?
Password?
Database?
PostgreSQL user?
```

If application says:

```text
too many connections
```

think:

```text
Connection pooling?
Application connection leak?
Too many EC2 instances?
RDS capacity?
RDS Proxy?
```

If database is:

```text
slow
```

think:

```text
CPU
Memory
IOPS
Latency
DB Load
Waits
Locks
Slow queries
Indexes
Connections
```

---

# 110. RDS Mental Model — Final

Remember this:

```text
RDS
│
├── Engine
│     ├── PostgreSQL
│     └── MySQL
│
├── Networking
│     ├── VPC
│     ├── DB Subnet Group
│     ├── Private Subnets
│     └── Security Group
│
├── Availability
│     ├── Multi-AZ
│     └── Failover
│
├── Scaling
│     ├── Vertical scaling
│     ├── Read replicas
│     ├── Aurora readers
│     └── Connection management
│
├── Recovery
│     ├── Automated backups
│     ├── PITR
│     └── Snapshots
│
├── Security
│     ├── Private access
│     ├── Security Groups
│     ├── Encryption
│     ├── KMS
│     ├── TLS
│     └── Secrets
│
├── Configuration
│     ├── Parameter Groups
│     └── Option Groups
│
└── Operations
      ├── CloudWatch
      ├── Database Insights
      ├── Enhanced Monitoring
      ├── Logs
      └── Events
```

---

# 111. Final Architecture to Memorize

```text
                         INTERNET
                            │
                            ▼
                         ALB
                            │
                            │
                    ┌───────▼────────┐
                    │       EC2      │
                    │ Application SG │
                    └───────┬────────┘
                            │
                        TCP 5432
                            │
                    ┌───────▼────────┐
                    │    RDS SG      │
                    │ Source=App SG  │
                    └───────┬────────┘
                            │
                    ┌───────▼────────┐
                    │ RDS PostgreSQL │
                    │ Private        │
                    │ No Public      │
                    └────────────────┘
```

The most important relationship is:

```text
ALB
 ↓
Application
 ↓
Database
```

and the most important database security rule is:

```text
RDS 5432
Source = Application Security Group
```

rather than:

```text
RDS 5432
Source = 0.0.0.0/0
```

---

# 112. Phase 6 Hands-On Labs

Complete these in order.

## Lab 1 — Basic RDS

```text
Create PostgreSQL
Private
No Public Access
```

---

## Lab 2 — DB Subnet Group

```text
Private DB subnet A
Private DB subnet B
```

---

## Lab 3 — Security Group

```text
5432
Source = Application SG
```

---

## Lab 4 — EC2 → RDS

From EC2:

```bash
nc -vz RDS_ENDPOINT 5432
```

Then:

```bash
psql ...
```

---

## Lab 5 — Break SG

Remove the correct source SG.

Observe failure.

Restore.

Observe success.

---

## Lab 6 — Backups

Create:

```text
manual snapshot
```

Restore:

```text
new RDS instance
```

---

## Lab 7 — PITR

Create data.

Delete data.

Restore to a point before deletion.

---

## Lab 8 — Multi-AZ

Enable/configure Multi-AZ.

Understand:

```text
primary
standby
failover
```

---

## Lab 9 — Read Replica

Create:

```text
Primary
   ↓
Read Replica
```

Observe:

```text
replication
lag
read-only behavior
```

---

## Lab 10 — Monitoring

Observe:

```text
CloudWatch
Database Insights
Enhanced Monitoring
Logs
Events
```

---

## Lab 11 — Parameter Group

Create:

```text
custom parameter group
```

Understand:

```text
dynamic
static
reboot
```

---

## Lab 12 — Aurora

Create a small Aurora PostgreSQL lab if your budget allows.

Learn:

```text
Cluster
Writer
Reader
Cluster endpoint
Reader endpoint
Shared storage
Aurora Replica
```

---

# 113. Cost Safety

RDS can generate charges while running.

After labs:

```text
Delete test DB instances
Delete read replicas
Delete Aurora clusters
Delete snapshots you no longer need
Remove unused DB subnet groups
Remove unused security groups
```

But remember:

```text
Deleting a DB
≠
automatically deleting every backup/snapshot
```

Review retained resources before considering the lab fully cleaned up.

---

# 114. Phase 6 Completion Checklist

You are ready to move forward when you can explain without notes:

```text
[ ] What is RDS?
[ ] What is Aurora?
[ ] PostgreSQL port?
[ ] MySQL port?
[ ] What is a DB subnet group?
[ ] Why at least two AZs?
[ ] What is a private DB subnet?
[ ] What does Public Access = No mean?
[ ] How does an EC2 SG reach an RDS SG?
[ ] Why use SG-to-SG instead of 0.0.0.0/0?
[ ] What is Multi-AZ?
[ ] What is a standby?
[ ] What is a read replica?
[ ] Multi-AZ vs read replica?
[ ] Why is a read replica not a backup?
[ ] What is automated backup?
[ ] What is PITR?
[ ] What is a snapshot?
[ ] What is encryption at rest?
[ ] What is TLS in transit?
[ ] What is KMS?
[ ] What is a parameter group?
[ ] What is an option group?
[ ] Why doesn't PostgreSQL use option groups?
[ ] What is Enhanced Monitoring?
[ ] What is Database Insights?
[ ] What happened to Performance Insights?
[ ] How do you troubleshoot an RDS timeout?
[ ] How do you troubleshoot authentication failure?
[ ] Why should DB passwords not be hard-coded?
[ ] Why should RDS normally be private?
```

---

# 115. Official AWS Documentation

## RDS Documentation

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/

## RDS Documentation Hub

https://docs.aws.amazon.com/rds/

## RDS VPC / DB Subnet Groups

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.WorkingWithRDSInstanceinaVPC.html

## RDS Security

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/infrastructure-security.html

## RDS Encryption

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.Encryption.html

## RDS Backups

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithAutomatedBackups.html

## Point-in-Time Restore

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIT.html

## RDS Multi-AZ

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html

## Multi-AZ DB Clusters

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/multi-az-db-clusters-concepts.html

## Read Replicas

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html

## PostgreSQL Read Replicas

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PostgreSQL.Replication.ReadReplicas.Configuration.html

## Parameter Groups

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/parameter-groups-overview.html

## Option Groups

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithOptionGroups.html

## RDS Monitoring

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/monitoring-cloudwatch.html

## Database Insights

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_DatabaseInsights.html

## CloudWatch Database Insights

https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Database-Insights.html

## Aurora Overview

https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.html

## Aurora Storage

https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.StorageReliability.html

## Aurora Replication

https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Replication.html

---

# 116. Final Rule

If you remember only one RDS architecture from this phase, remember:

```text
                         INTERNET
                            │
                            ▼
                           ALB
                            │
                            ▼
                           EC2
                            │
                    Application SG
                            │
                         5432
                            │
                            ▼
                    RDS Security Group
                    Source = App SG
                            │
                            ▼
                    Private RDS PostgreSQL
                            │
                    Public Access = No
```

And remember:

```text
Multi-AZ
    =
High availability / failover

Read Replica
    =
Read scaling / replication

Backup
    =
Recovery

Snapshot
    =
Explicit point-in-time backup

Security Group
    =
Network access control

Parameter Group
    =
Database engine configuration

Database Insights
    =
Database performance/load investigation

KMS
    =
Encryption key management
```

This mental model will carry forward into later AWS architecture work.
