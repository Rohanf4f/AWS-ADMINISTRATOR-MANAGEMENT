# PHASE 3.2 — SECURITY GROUPS VS NACL

## Goal

Understand AWS network security deeply enough to answer:

> Why is this packet allowed or denied?

You will learn:

```text
Security Group
vs
Network ACL
```

and then intentionally break network access and troubleshoot it.

---

# 3.2.1 Core Difference

## Security Group

```text
Resource / ENI level
Stateful
Allow rules
```

## Network ACL

```text
Subnet level
Stateless
Allow + Deny rules
```

AWS documents these differences directly. Security groups are stateful; NACLs are stateless and evaluate numbered rules in order.

Official documentation:

https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html

https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html

---

# 3.2.2 Mental Model

Think of the packet path like this:

```text
Internet
   |
   v
Internet Gateway
   |
   v
Route Table
   |
   v
NACL
   |
   v
Security Group
   |
   v
EC2 ENI
```

For traffic leaving a subnet:

```text
EC2 ENI
   |
   v
Security Group
   |
   v
NACL
   |
   v
Route
```

The exact network path can contain additional components, but this is a useful learning model.

---

# 3.2.3 Security Group

A security group is attached to resources through their network interfaces.

For EC2, think:

```text
EC2
 |
 +-- ENI
      |
      +-- Security Group
```

A security group can contain inbound and outbound rules.

Example:

```text
Inbound
------------------------------
SSH    TCP 22
Source: YOUR_IP/32

HTTP   TCP 80
Source: 0.0.0.0/0

HTTPS  TCP 443
Source: 0.0.0.0/0
```

Security groups allow traffic; they do not use explicit deny rules.

Security groups are stateful.

That means if an inbound request is allowed, response traffic is automatically allowed by the stateful behavior of the security group.

Official documentation:

https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html

---

# 3.2.4 Security Group Example

Create:

```text
Name:
company-lab-web-sg
```

Inbound:

```text
SSH
TCP
22
YOUR_PUBLIC_IP/32

HTTP
TCP
80
0.0.0.0/0

HTTPS
TCP
443
0.0.0.0/0
```

Outbound:

```text
Allow required outbound traffic
```

For a simple learning instance, the default outbound rule may be sufficient, but understand exactly what it allows.

---

# 3.2.5 Why /32?

If your public IP is:

```text
203.0.113.50
```

then:

```text
203.0.113.50/32
```

means one IPv4 address.

This is much narrower than:

```text
0.0.0.0/0
```

which represents all IPv4 addresses.

---

# 3.2.6 Do Not Open SSH Globally by Default

Avoid:

```text
TCP 22
0.0.0.0/0
```

for normal administration.

Prefer:

```text
TCP 22
YOUR_IP/32
```

For production environments, stronger administrative patterns may use Systems Manager Session Manager, bastion/controlled access patterns, VPN/private connectivity, or other approved access mechanisms rather than exposing SSH directly.

---

# 3.2.7 Security Group References

Instead of:

```text
Source:
10.0.0.0/16
```

you can often allow one security group to reach another.

Example:

```text
ALB-SG
   |
   v
APP-SG
```

APP-SG inbound:

```text
TCP 8080
Source:
ALB-SG
```

Then:

```text
APP-SG
   |
   v
DB-SG
```

DB-SG inbound:

```text
TCP 5432
Source:
APP-SG
```

This creates a logical trust relationship:

```text
Load Balancer
      |
      v
Application
      |
      v
Database
```

rather than allowing entire CIDR ranges unnecessarily.

---

# 3.2.8 Network ACL

A Network ACL is associated with a subnet.

Example:

```text
Subnet-A
   |
   v
NACL-A
```

A subnet can be associated with one NACL at a time, while a NACL can be associated with multiple subnets.

NACLs have:

```text
Inbound rules
Outbound rules
```

Rules can:

```text
ALLOW
DENY
```

They are evaluated in ascending rule-number order.

Example:

```text
Rule 100
ALLOW TCP 443

Rule 200
DENY ALL
```

Traffic matching rule 100 is allowed before the later deny is considered.

Use rule numbers with gaps, such as:

```text
100
200
300
```

so you can insert rules later.

---

# 3.2.9 NACL Is Stateless

This is one of the most important concepts.

Suppose:

```text
Client
  |
  | TCP request
  v
EC2
```

You allow inbound TCP 443.

The response goes back in the opposite direction.

With a stateless NACL, you must allow the return traffic appropriately in the outbound rules.

Security groups behave differently because they are stateful.

AWS documentation example:

https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-records-examples.html

---

# 3.2.10 Security Group vs NACL

| Characteristic | Security Group | NACL |
|---|---|---|
| Scope | Resource/ENI | Subnet |
| State | Stateful | Stateless |
| Rules | Allow | Allow + Deny |
| Rule order | No numbered first-match model | Lowest-number matching rule |
| Return traffic | Automatically handled by stateful behavior | Must be allowed |
| Typical use | Resource-level access | Subnet-level network filtering |

Official AWS comparison:

https://docs.aws.amazon.com/vpc/latest/userguide/nacl-examples.html

---

# 3.2.11 Exercise 1 — Create EC2 Security Group

Create:

```text
company-lab-ec2-sg
```

Inbound:

```text
SSH
TCP
22
Source:
YOUR_IP/32

HTTP
TCP
80
Source:
0.0.0.0/0

HTTPS
TCP
443
Source:
0.0.0.0/0
```

Outbound:

```text
Use the minimum outbound access needed for the lab.
```

Launch a test EC2 in the public subnet.

---

# 3.2.12 Exercise 2 — Verify SSH

From your machine:

```text
ssh user@PUBLIC_IP
```

If SSH works, record:

```text
Source IP
Destination IP
Destination port
Route table
Security group
NACL
```

You should be able to explain why the packet was allowed.

---

# 3.2.13 Exercise 3 — Break the Security Group

Remove:

```text
TCP 22
YOUR_IP/32
```

Try SSH again.

Expected result:

```text
Connection fails
```

Now ask:

```text
Did the route disappear?
```

No.

The network path may still exist, but the security group no longer permits the traffic.

Restore the rule.

---

# 3.2.14 Exercise 4 — Change the SSH Source

Change:

```text
YOUR_IP/32
```

to a different IP that is not your current public IP.

Test again.

This teaches that security group source CIDRs are actual source-network restrictions.

Restore your correct IP afterward.

---

# 3.2.15 Exercise 5 — NACL Deny

Create a custom NACL:

```text
company-lab-test-nacl
```

Associate it with the test subnet.

Add a numbered inbound deny rule for your SSH source.

For example, conceptually:

```text
Rule 100
DENY
TCP 22
YOUR_IP/32
```

Then make sure the subnet's NACL configuration is understood before testing.

Try SSH.

Expected:

```text
SSH fails
```

Restore the NACL association/rules after the exercise.

---

# 3.2.16 Exercise 6 — NACL Return Traffic

This is the deeper exercise.

Because NACLs are stateless, investigate both directions.

For a TCP connection:

```text
Client
  |
  | destination port 22
  v
EC2
  |
  | response traffic
  v
Client
```

Your NACL configuration must permit the relevant traffic in both directions.

Do not blindly add:

```text
ALLOW ALL
```

as your permanent solution.

Understand the source/destination ports and ephemeral return traffic for the protocol and operating system involved.

---

# 3.2.17 Exercise 7 — HTTP

Keep:

```text
TCP 80
0.0.0.0/0
```

in the security group.

Run a simple HTTP server on the EC2 instance.

Example:

```text
python3 -m http.server 80
```

Then access:

```text
http://PUBLIC_IP
```

If it fails, troubleshoot:

```text
Internet
  |
  v
IGW
  |
  v
Public Route Table
  |
  v
NACL
  |
  v
Security Group
  |
  v
EC2
  |
  v
Application listening on port 80?
```

---

# 3.2.18 Exercise 8 — Database Security

Create:

```text
app-sg
db-sg
```

DB rule:

```text
TCP 5432
Source:
app-sg
```

Do not use:

```text
TCP 5432
0.0.0.0/0
```

The desired logical model:

```text
Internet
   |
   v
Load Balancer
   |
   v
app-sg
   |
   v
EC2 / Application
   |
   v
db-sg
   |
   v
RDS
```

This creates a much smaller trust boundary than exposing the database to the internet.

---

# 3.2.19 Troubleshooting Decision Tree

When SSH fails:

```text
SSH fails
 |
 +-- Is EC2 running?
 |
 +-- Does EC2 have a public IP?
 |
 +-- Is subnet public?
 |      |
 |      +-- Route 0.0.0.0/0 -> IGW?
 |
 +-- NACL inbound allows SSH?
 |
 +-- NACL outbound allows return traffic?
 |
 +-- Security Group inbound allows TCP 22?
 |
 +-- Is source IP correct?
 |
 +-- Is sshd/application listening?
 |
 +-- Is OS firewall blocking it?
```

Do not assume:

```text
SG allows = connection must work
```

Networking has multiple layers.

---

# 3.2.20 VPC Flow Logs

VPC Flow Logs can help diagnose network connectivity and security-group/NACL problems.

Conceptually:

```text
Traffic
   |
   v
Network Interface
   |
   v
Flow Logs
   |
   v
Logs / analysis
```

Use them when you need evidence about accepted/rejected traffic patterns.

Official documentation:

https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html

---

# 3.2.21 Common Mistakes

## Mistake 1

Opening SSH:

```text
0.0.0.0/0
```

Avoid unless there is a controlled temporary reason.

## Mistake 2

Opening a database:

```text
3306 -> 0.0.0.0/0
```

Avoid for normal private DB architectures.

## Mistake 3

Forgetting NACL return traffic.

NACLs are stateless.

## Mistake 4

Thinking a route table is a firewall.

It is not.

## Mistake 5

Thinking a public subnet automatically makes every resource public.

Public routing, addressing, and security permissions are separate concerns.

## Mistake 6

Changing five network settings at once.

Change one thing, test, observe, restore, and document.

---

# 3.2.22 Final Lab

Build:

```text
Internet
   |
   v
Public Subnet
   |
   v
EC2
   |
   | SG: ec2-sg
   |
   v
Private App Subnet
   |
   | SG: app-sg
   |
   v
Private DB Subnet
   |
   | SG: db-sg
   |
   v
RDS
```

Then test:

### Test A

```text
Your IP -> EC2:22
```

Expected:

```text
Allowed
```

### Test B

```text
Random Internet IP -> EC2:22
```

Expected:

```text
Not allowed by SG
```

### Test C

```text
EC2/App -> DB:5432
```

Expected:

```text
Allowed if routing + SG + NACL + DB are configured correctly
```

### Test D

```text
Internet -> DB:5432
```

Expected:

```text
No direct public exposure in this architecture
```

### Test E

Break one control:

```text
SG
NACL
Route
```

Identify which layer caused the failure.

---

# 3.2.23 Completion Checklist

You should be able to explain:

- Security group.
- Network ACL.
- Resource/ENI-level filtering.
- Subnet-level filtering.
- Stateful.
- Stateless.
- Allow-only security groups.
- Allow + deny NACLs.
- NACL rule ordering.
- Return traffic.
- Security group references.
- `/32`.
- Why SSH should normally be restricted.
- Why databases should not be publicly exposed.
- How to troubleshoot a failed connection.
- How VPC Flow Logs can help.

---

# Final Mental Model

```text
                 PACKET
                    |
                    v
              ROUTING LAYER
                    |
                    v
              NETWORK ACL
              subnet level
                    |
                    v
            SECURITY GROUP
             resource level
                    |
                    v
                RESOURCE
```

Remember:

```text
Route table
= Where should the packet go?

Security Group
= Is this resource allowed to communicate?

NACL
= Is this subnet allowed to send/receive this traffic?

Application
= Is anything actually listening on the destination port?
```

The most important skill is not memorizing the definitions.

It is being able to troubleshoot:

```text
Connection failed
        |
        v
Trace the packet
        |
        v
Find the exact layer that blocked it
        |
        v
Fix only that layer
        |
        v
Test again
```

Official documentation:

https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html

https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html

https://docs.aws.amazon.com/vpc/latest/userguide/nacl-examples.html
