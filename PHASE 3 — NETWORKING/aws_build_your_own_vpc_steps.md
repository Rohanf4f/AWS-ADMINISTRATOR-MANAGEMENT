# PHASE 3.1 — BUILD YOUR OWN AWS VPC

## Goal

Build a production-style learning VPC manually in the AWS Console.

You will create:

```text
VPC
10.0.0.0/16
```

with six subnets across two Availability Zones:

```text
AZ-A
 |
 +-- Public-A
 |   10.0.1.0/24
 |
 +-- Private-App-A
 |   10.0.11.0/24
 |
 +-- Private-DB-A
     10.0.21.0/24

AZ-B
 |
 +-- Public-B
 |   10.0.2.0/24
 |
 +-- Private-App-B
 |   10.0.12.0/24
 |
 +-- Private-DB-B
     10.0.22.0/24
```

The final logical design:

```text
                         Internet
                            |
                            v
                    Internet Gateway
                       /         \
                      v           v
                 Public-A      Public-B
                   AZ-A          AZ-B
                      |           |
                 NAT Gateway  NAT Gateway
                      |           |
                      v           v
              Private-App-A  Private-App-B
                    |             |
                    +------+------+
                           |
                    Private-DB-A/B
```

---

# 3.1.1 Before You Start

Use a dedicated learning environment/account if possible.

Networking resources can incur charges, especially NAT Gateways and data transfer.

Before creating resources:

1. Open AWS Console.
2. Select the intended Region.
3. Confirm the account.
4. Confirm your permissions.
5. Check the AWS pricing information for the resources you will create.
6. Decide how you will clean everything up after the lab.

Official VPC documentation:

https://docs.aws.amazon.com/vpc/latest/userguide/

---

# 3.1.2 Naming Standard

Use consistent names.

Example:

```text
company-lab-vpc

company-lab-public-a
company-lab-public-b

company-lab-app-a
company-lab-app-b

company-lab-db-a
company-lab-db-b

company-lab-public-rt
company-lab-app-rt-a
company-lab-app-rt-b
company-lab-db-rt-a
company-lab-db-rt-b

company-lab-igw
company-lab-nat-a
company-lab-nat-b
```

Tags:

```text
Environment=lab
Project=aws-networking
ManagedBy=manual
Owner=learning
```

---

# 3.1.3 Step 1 — Create the VPC

Go to:

```text
AWS Console
  ->
VPC
  ->
Your VPCs
  ->
Create VPC
```

Select:

```text
Resources to create:
VPC only
```

Set:

```text
Name:
company-lab-vpc

IPv4 CIDR:
10.0.0.0/16
```

Enable the normal DNS options needed by your workloads.

Create the VPC.

Verify:

```text
VPC ID
CIDR = 10.0.0.0/16
State = Available
```

---

# 3.1.4 Step 2 — Choose Two Availability Zones

Use two AZs in the same AWS Region.

Example:

```text
Region
  |
  +-- AZ-A
  +-- AZ-B
```

Do not hard-code a specific AZ letter as universally better.

AWS Availability Zone names are account-specific mappings, so your `a`/`b` choice is about distributing resources across distinct AZs, not about a universal global meaning.

---

# 3.1.5 Step 3 — Create Six Subnets

## Public-A

```text
Name:
company-lab-public-a

CIDR:
10.0.1.0/24

AZ:
AZ-A
```

## Public-B

```text
Name:
company-lab-public-b

CIDR:
10.0.2.0/24

AZ:
AZ-B
```

## Private-App-A

```text
Name:
company-lab-app-a

CIDR:
10.0.11.0/24

AZ:
AZ-A
```

## Private-App-B

```text
Name:
company-lab-app-b

CIDR:
10.0.12.0/24

AZ:
AZ-B
```

## Private-DB-A

```text
Name:
company-lab-db-a

CIDR:
10.0.21.0/24

AZ:
AZ-A
```

## Private-DB-B

```text
Name:
company-lab-db-b

CIDR:
10.0.22.0/24

AZ:
AZ-B
```

Final:

```text
10.0.0.0/16
|
+-- 10.0.1.0/24    Public-A
+-- 10.0.2.0/24    Public-B
+-- 10.0.11.0/24   App-A
+-- 10.0.12.0/24   App-B
+-- 10.0.21.0/24   DB-A
+-- 10.0.22.0/24   DB-B
```

---

# 3.1.6 Step 4 — Create Internet Gateway

Go to:

```text
VPC
 ->
Internet Gateways
 ->
Create Internet Gateway
```

Name:

```text
company-lab-igw
```

Then:

```text
Internet Gateway
      |
      v
Attach to VPC
```

Select:

```text
company-lab-vpc
```

Verify the IGW is attached.

An Internet Gateway provides a target in route tables for internet-routable traffic. A public subnet requires a route to the IGW.

Official documentation:

https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html

---

# 3.1.7 Step 5 — Create Public Route Table

Go to:

```text
VPC
 ->
Route Tables
 ->
Create route table
```

Name:

```text
company-lab-public-rt
```

VPC:

```text
company-lab-vpc
```

Add route:

```text
Destination:
0.0.0.0/0

Target:
Internet Gateway
company-lab-igw
```

The route table will also contain a local route such as:

```text
10.0.0.0/16 -> local
```

Associate:

```text
company-lab-public-a
company-lab-public-b
```

with the public route table.

Now these are public subnets from a routing perspective.

---

# 3.1.8 Step 6 — Create NAT Gateways

Create one NAT Gateway in each public subnet for the high-availability learning design:

```text
NAT-A -> Public-A
NAT-B -> Public-B
```

Each NAT Gateway needs a public IPv4 address allocation as required by the current AWS console workflow.

Conceptually:

```text
Private-App-A
     |
     v
NAT-A
     |
     v
IGW
     |
     v
Internet
```

and:

```text
Private-App-B
     |
     v
NAT-B
     |
     v
IGW
     |
     v
Internet
```

Important:

NAT Gateway costs money. Delete the NAT Gateways after the lab if you do not need them.

Official documentation:

https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat.html

---

# 3.1.9 Step 7 — Create Private App Route Tables

Create:

```text
company-lab-app-rt-a
company-lab-app-rt-b
```

App-A route table:

```text
10.0.0.0/16 -> local
0.0.0.0/0   -> NAT-A
```

Associate:

```text
company-lab-app-a
```

App-B route table:

```text
10.0.0.0/16 -> local
0.0.0.0/0   -> NAT-B
```

Associate:

```text
company-lab-app-b
```

Now:

```text
Private-App-A
      |
      v
NAT-A
      |
      v
Internet Gateway
      |
      v
Internet
```

and similarly for App-B.

---

# 3.1.10 Step 8 — Create DB Route Tables

Create:

```text
company-lab-db-rt-a
company-lab-db-rt-b
```

Use only:

```text
10.0.0.0/16 -> local
```

Associate:

```text
DB-A -> company-lab-db-rt-a
DB-B -> company-lab-db-rt-b
```

This creates an isolated-style database network from the routing perspective.

The database subnets do not have a default internet route.

---

# 3.1.11 Step 9 — Verify Route Tables

Your final route design should look approximately like:

```text
PUBLIC RT
-------------------------
10.0.0.0/16 -> local
0.0.0.0/0   -> IGW

APP RT A
-------------------------
10.0.0.0/16 -> local
0.0.0.0/0   -> NAT-A

APP RT B
-------------------------
10.0.0.0/16 -> local
0.0.0.0/0   -> NAT-B

DB RT A
-------------------------
10.0.0.0/16 -> local

DB RT B
-------------------------
10.0.0.0/16 -> local
```

Official route-table documentation:

https://docs.aws.amazon.com/vpc/latest/userguide/RouteTables.html

---

# 3.1.12 Step 10 — Create Security Groups

Create an application security group:

```text
company-lab-app-sg
```

For the learning lab, you can allow application traffic from your load balancer security group later.

Create a database security group:

```text
company-lab-db-sg
```

Example database rule:

```text
TCP
5432
Source:
company-lab-app-sg
```

for PostgreSQL.

Or:

```text
TCP
3306
Source:
company-lab-app-sg
```

for MySQL.

Do not use:

```text
0.0.0.0/0
```

for database access in this architecture.

Security groups are stateful and operate at the network-interface/resource level.

Official documentation:

https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html

---

# 3.1.13 Step 11 — Optional EC2 Test

Launch a test EC2 instance in:

```text
company-lab-public-a
```

For the lab, assign a public IPv4 address if you need direct SSH access.

Security group:

```text
SSH
22
Source:
YOUR_PUBLIC_IP/32
```

Do not use:

```text
0.0.0.0/0
```

for SSH unless you have a specific temporary test reason and understand the exposure.

Then test:

```text
Your computer
    |
    v
Internet
    |
    v
IGW
    |
    v
Public subnet
    |
    v
EC2
```

---

# 3.1.14 Step 12 — Packet Flow Test

## Public EC2 -> Internet

Expected path:

```text
EC2
 |
 v
Public Subnet
 |
 v
Public Route Table
 |
 | 0.0.0.0/0
 v
Internet Gateway
 |
 v
Internet
```

Check:

- Public IPv4 exists.
- Route exists.
- Security group allows outbound.
- NACL allows traffic.

---

# 3.1.15 Step 13 — Private App -> Internet

If you later launch EC2 into Private-App-A:

```text
Private EC2
 |
 v
App Route Table
 |
 | 0.0.0.0/0
 v
NAT-A
 |
 v
IGW
 |
 v
Internet
```

This tests why private subnets can have outbound internet connectivity without being directly reachable from the internet through the NAT path.

---

# 3.1.16 Step 14 — DB Network

The DB subnets have:

```text
10.0.0.0/16 -> local
```

only.

Therefore:

```text
Internet
   X
   |
Private DB subnet
```

But resources inside the VPC can route to the DB subnet because of the local VPC route, subject to security controls.

---

# 3.1.17 Step 15 — Add RDS Later

When you reach the RDS phase, place the DB subnet group across:

```text
DB-A
DB-B
```

Then use:

```text
App Security Group
       |
       v
DB Security Group
       |
       v
RDS
```

This is the common pattern to practice:

```text
Internet
   |
   v
Load Balancer
   |
   v
App
   |
   v
Database
```

---

# 3.1.18 VPC Verification Checklist

Verify each item.

## VPC

```text
CIDR = 10.0.0.0/16
DNS support = enabled as required
DNS hostnames = enabled as required
```

## Subnets

```text
Public-A      10.0.1.0/24
Public-B      10.0.2.0/24
App-A         10.0.11.0/24
App-B         10.0.12.0/24
DB-A          10.0.21.0/24
DB-B          10.0.22.0/24
```

## Routing

```text
Public -> IGW
App-A  -> NAT-A
App-B  -> NAT-B
DB     -> local only
```

## Availability

```text
AZ-A
AZ-B
```

## Security

```text
SSH -> your IP only
DB -> application SG only
```

---

# 3.1.19 Troubleshooting Lab

Intentionally break one component at a time.

## Break #1

Remove:

```text
0.0.0.0/0 -> IGW
```

from the public route table.

Question:

> Can a public EC2 reach the internet?

Restore the route after observing the failure.

## Break #2

Remove:

```text
0.0.0.0/0 -> NAT-A
```

from App-A.

Question:

> Can a private App-A instance reach the internet?

## Break #3

Remove the NAT subnet's route to the IGW.

Question:

> What breaks?

## Break #4

Remove SSH from the EC2 security group.

Question:

> Does the route still exist?

This teaches the difference between:

```text
Routing failure
vs
Security failure
```

---

# 3.1.20 Cleanup

After the lab, remove resources you do not need.

Recommended cleanup order:

```text
Test EC2
   |
   v
NAT Gateways
   |
   v
Route-table associations/routes
   |
   v
Subnets
   |
   v
Internet Gateway
   |
   v
VPC
```

Also release public IPv4/EIP resources that are no longer needed according to the current AWS console behavior.

Always check Billing/Cost Explorer after a lab that includes paid networking resources.

---

# Official Documentation

- Create VPC:
  https://docs.aws.amazon.com/vpc/latest/userguide/create-vpc.html
- Route tables:
  https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html
- Internet Gateway:
  https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html
- NAT Gateway:
  https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat.html
- Security Groups:
  https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html

# Completion Checklist

You are finished when you can:

- Create a VPC from scratch.
- Choose a CIDR.
- Create non-overlapping subnets.
- Place subnets across two AZs.
- Attach an Internet Gateway.
- Build a public route table.
- Build private application route tables.
- Build isolated DB route tables.
- Deploy NAT Gateways.
- Explain every route.
- Trace public EC2 -> Internet.
- Trace private EC2 -> NAT -> Internet.
- Explain why DB subnets have no default internet route.
- Break a route and troubleshoot it.
- Clean up the lab.
