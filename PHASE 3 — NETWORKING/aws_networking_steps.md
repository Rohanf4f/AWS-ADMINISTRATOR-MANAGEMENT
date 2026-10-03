# PHASE 3 — AWS NETWORKING

## Goal

AWS networking is not something to memorize as a list of services.

You should be able to look at a packet and answer:

> Where does this packet start, what IP does it use, which route table handles it, which gateway/target receives it, and which security controls can allow or block it?

The core mental model is:

```text
Client / Internet
       |
       v
Internet Gateway
       |
       v
Public Subnet
       |
       v
Load Balancer
       |
       v
Private App Subnet
       |
       v
EC2 / ECS
       |
       v
Private DB Subnet
       |
       v
RDS
```

A VPC is your logical network boundary. Subnets divide the VPC IP space. Route tables decide where packets go. Gateways/endpoints provide connectivity to other networks or AWS services. Security groups and network ACLs control traffic.

---

# 3.1 Networking Mental Model

Before creating anything, understand these layers:

```text
VPC
 |
 +-- CIDR
 |
 +-- Subnets
 |    |
 |    +-- Public
 |    +-- Private
 |    +-- Isolated
 |
 +-- Route Tables
 |
 +-- Gateways / Endpoints
 |    |
 |    +-- Internet Gateway
 |    +-- NAT Gateway
 |    +-- VPC Endpoint
 |    +-- Transit Gateway
 |    +-- VPC Peering
 |
 +-- Network Security
      |
      +-- Security Groups
      +-- Network ACLs
```

## Packet-flow checklist

For any connection, ask:

1. What is the source IP?
2. What is the destination IP?
3. Which subnet contains the source?
4. Which route table is associated with that subnet?
5. What is the most specific matching route?
6. What target does that route point to?
7. Is an Internet Gateway, NAT Gateway, peering connection, Transit Gateway, VPN, or endpoint involved?
8. Does the Network ACL allow the traffic?
9. Does the Security Group allow the traffic?
10. Does the destination resource allow the connection?

AWS route tables use destination/target pairs, and routing uses the most specific matching route. Every route table also has a local route for communication inside the VPC. [AWS Route Tables](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)

---

# 3.2 VPC

## What is a VPC?

Amazon VPC gives you a logically isolated network in AWS.

Example:

```text
VPC
10.0.0.0/16
```

This gives the VPC an IPv4 address range from which subnet ranges can be allocated.

Example:

```text
10.0.0.0/16
    |
    +-- 10.0.1.0/24
    +-- 10.0.2.0/24
    +-- 10.0.11.0/24
    +-- 10.0.12.0/24
```

Official documentation:

https://docs.aws.amazon.com/vpc/latest/userguide/

---

# 3.3 CIDR

CIDR describes an IP address range.

Examples:

```text
10.0.0.0/16
10.0.1.0/24
10.0.11.0/24
```

The `/16` or `/24` is the prefix length.

For learning, remember:

```text
/16 → 16 bits network + 16 bits host
/24 → 24 bits network + 8 bits host
/32 → 32 bits network + 0 bits host
```

A subnet must be contained within the VPC CIDR and should not overlap another subnet in the same VPC.

Example:

```text
VPC
10.0.0.0/16

Public-A
10.0.1.0/24

Public-B
10.0.2.0/24

Private-App-A
10.0.11.0/24
```

---

# 3.4 Subnets

A subnet is an IP range inside a VPC.

Common architecture:

```text
VPC 10.0.0.0/16

AZ-A
 |
 +-- Public-A
 +-- Private-App-A
 +-- Private-DB-A

AZ-B
 |
 +-- Public-B
 +-- Private-App-B
 +-- Private-DB-B
```

Use multiple Availability Zones for higher availability.

A subnet is not automatically "public" just because its name says Public.

A subnet is considered public when its associated route table has a route to an Internet Gateway.

Official AWS documentation:

https://docs.aws.amazon.com/vpc/latest/userguide/subnet-route-tables.html

---

# 3.5 Public vs Private Subnet

## Public subnet

Typical route:

```text
0.0.0.0/0 -> Internet Gateway
```

A resource that needs direct IPv4 internet connectivity also needs an appropriate public IPv4 address/EIP and security rules.

## Private subnet

Typical route:

```text
0.0.0.0/0 -> NAT Gateway
```

The private resource can initiate outbound IPv4 connections through the NAT Gateway, but unsolicited inbound connections from the internet are not allowed through that NAT path.

## Isolated subnet

Example:

```text
No default route to Internet Gateway
No default route to NAT Gateway
```

This is useful for resources that should not have general internet access.

---

# 3.6 Route Tables

A route table is the traffic controller for a subnet.

Example public route table:

```text
Destination       Target

10.0.0.0/16       local
0.0.0.0/0         igw-xxxx
```

Example private application route table:

```text
Destination       Target

10.0.0.0/16       local
0.0.0.0/0         nat-xxxx
```

Example database route table:

```text
Destination       Target

10.0.0.0/16       local
```

Important:

```text
Route table != firewall
```

A route says:

> If the destination matches this network, send the packet toward this target.

Security groups and NACLs decide whether traffic is allowed.

AWS uses the most specific matching route.

Official documentation:

https://docs.aws.amazon.com/vpc/latest/userguide/RouteTables.html

---

# 3.7 Internet Gateway

An Internet Gateway (IGW) connects a VPC to the internet.

Typical public subnet:

```text
Internet
   |
   v
Internet Gateway
   |
   v
Public Subnet
```

The IGW must be attached to the VPC.

The public subnet's route table needs:

```text
0.0.0.0/0 -> Internet Gateway
```

For IPv6, an internet route normally uses:

```text
::/0 -> Internet Gateway
```

An IGW does not make every resource public automatically.

The resource also needs appropriate addressing and security rules.

Official documentation:

https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html

---

# 3.8 NAT Gateway

NAT Gateway is commonly used when resources in private subnets need outbound IPv4 internet access.

Architecture:

```text
Private EC2
    |
    v
Private Route Table
    |
    v
NAT Gateway
    |
    v
Internet Gateway
    |
    v
Internet
```

Important:

```text
Private subnet
     |
     v
NAT Gateway
     |
     v
Internet
```

The internet cannot normally start a new connection to the private EC2 through the NAT Gateway.

A NAT Gateway is normally placed in a public subnet.

Its public-side connectivity requires the public subnet to have a route to the Internet Gateway.

AWS recommends deploying a NAT Gateway in each active AZ in production designs where AZ-independent outbound connectivity is required.

Official documentation:

https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat.html

---

# 3.9 Elastic IP

An Elastic IP is a static public IPv4 address allocated to your AWS account.

Common use:

```text
NAT Gateway
```

Historically, Elastic IPs were also commonly associated with EC2 instances.

Do not assume:

```text
Private IP = Internet reachable
```

A private IP is internal to the VPC.

---

# 3.10 VPC DNS

AWS VPC provides DNS capabilities through the Route 53 Resolver.

Understand:

```text
EC2
 |
 | DNS query
 v
VPC DNS resolver
 |
 v
DNS resolution
```

When creating a VPC, DNS support and DNS hostnames are important settings for normal AWS workloads.

Official documentation:

https://docs.aws.amazon.com/vpc/latest/userguide/AmazonDNS-concepts.html

---

# 3.11 DHCP Options

DHCP options control network configuration information supplied to instances in a VPC.

Typical concepts include:

```text
Domain name
Domain-name servers
NTP servers
NetBIOS settings
```

You do not normally need to customize DHCP options for a basic AWS application.

Learn them because enterprise networks may use custom DNS or domain settings.

Official documentation:

https://docs.aws.amazon.com/vpc/latest/userguide/VPC_DHCP_Options.html

---

# 3.12 VPC Endpoints

VPC endpoints allow private connectivity from a VPC to supported AWS services.

This can avoid sending traffic through the public internet.

Conceptually:

```text
Private EC2
    |
    v
VPC Endpoint
    |
    v
AWS Service
```

Important endpoint types to understand:

```text
Gateway Endpoint
Interface Endpoint
```

A common example is private access from workloads to Amazon S3 using an S3 gateway endpoint.

Official documentation:

https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html

---

# 3.13 VPC Peering

VPC Peering connects two VPCs privately.

Example:

```text
VPC-A
10.0.0.0/16
    |
    | VPC Peering
    |
VPC-B
10.1.0.0/16
```

Both sides need appropriate route-table entries.

The CIDR ranges must not overlap for normal VPC peering.

Do not think:

```text
VPC Peering = automatic communication
```

You still need routing and security controls.

---

# 3.14 Transit Gateway

Transit Gateway acts as a central network hub.

Example:

```text
             Transit Gateway
              /     |      \
             /      |       \
            v       v        v
          VPC-A   VPC-B    VPC-C
```

It can also connect VPCs with VPN and Direct Connect connectivity.

This is useful when a company has many VPCs and wants a centralized connectivity model rather than managing many individual peering relationships.

Official documentation:

https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html

---

# 3.15 Load Balancer Placement

A common production architecture is:

```text
Internet
   |
   v
Internet Gateway
   |
   v
Public Subnets
   |
   v
Application Load Balancer
   |
   v
Private App Subnets
   |
   v
EC2 / ECS
   |
   v
Private DB Subnets
   |
   v
RDS
```

The exact design depends on the workload, but the key principle is:

> Keep resources that do not need direct internet reachability away from direct internet exposure.

---

# 3.16 Packet Flow Exercises

For every scenario, draw the path.

## Scenario A

Public EC2 -> Internet

```text
EC2
 |
 v
Public Route Table
 |
 v
Internet Gateway
 |
 v
Internet
```

Ask:

- Does the subnet have an IGW route?
- Does the EC2 have public IPv4/EIP?
- Does the security group allow outbound traffic?
- Does the NACL allow the traffic?

## Scenario B

Private EC2 -> Internet

```text
EC2
 |
 v
Private Route Table
 |
 v
NAT Gateway
 |
 v
Internet Gateway
 |
 v
Internet
```

## Scenario C

EC2 -> RDS

```text
EC2
 |
 v
EC2 Security Group
 |
 v
RDS Security Group
 |
 v
RDS
```

The RDS security group should normally allow the database port from the application security group, rather than from the whole internet.

---

# 3.17 Phase 3 Hands-on Lab

Create a VPC:

```text
VPC
10.0.0.0/16
```

Create:

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

Then build:

```text
                    Internet
                       |
                       v
                Internet Gateway
                       |
              +--------+--------+
              |                 |
          Public-A          Public-B
              |                 |
           ALB / NAT         ALB / NAT
              |                 |
              +--------+--------+
                       |
               Private App
               /         \
        App-A             App-B
           \               /
            \             /
             Private DB
             /        \
          DB-A       DB-B
```

You do not need to create every application component immediately.

First create the network and verify:

1. VPC exists.
2. All six subnets exist.
3. Subnets are in the intended AZs.
4. Public route table exists.
5. Private-app route tables exist.
6. DB route tables exist.
7. Internet Gateway is attached.
8. NAT Gateway is reachable from private route tables.
9. Routes are correct.
10. Security groups are correct.

---

# 3.18 Troubleshooting Method

When a connection fails, never randomly change settings.

Use this order:

```text
1. Source IP
2. Destination IP
3. Source subnet
4. Destination subnet
5. Route table
6. Route target
7. Internet Gateway / NAT / Endpoint / Peering / TGW
8. NACL
9. Security Group
10. Application/service itself
```

Example:

```text
EC2 cannot reach internet
```

Check:

```text
Is EC2 in public subnet?
        |
        +-- Does route table have 0.0.0.0/0 -> IGW?
        |
        +-- Does EC2 have public IPv4?
        |
        +-- Does SG allow outbound?
        |
        +-- Does NACL allow outbound and return traffic?
```

For private EC2:

```text
Private route table
        |
        +-- 0.0.0.0/0 -> NAT Gateway?
        |
        +-- NAT Gateway in public subnet?
        |
        +-- NAT subnet route -> IGW?
        |
        +-- NAT Gateway healthy?
```

---

# 3.19 Official Documentation

- VPC: https://docs.aws.amazon.com/vpc/latest/userguide/
- Create a VPC: https://docs.aws.amazon.com/vpc/latest/userguide/create-vpc.html
- Route tables: https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html
- Subnet routing: https://docs.aws.amazon.com/vpc/latest/userguide/subnet-route-tables.html
- Internet Gateway: https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html
- NAT: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat.html
- Security Groups: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html
- Network ACLs: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html
- VPC Endpoints: https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html
- DNS: https://docs.aws.amazon.com/vpc/latest/userguide/AmazonDNS-concepts.html
- DHCP: https://docs.aws.amazon.com/vpc/latest/userguide/VPC_DHCP_Options.html
- VPC Peering: https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html
- Transit Gateway: https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html

---

# 3.20 Completion Checklist

You should be able to explain without notes:

- What a VPC is.
- What CIDR means.
- How subnets divide a VPC.
- Public vs private vs isolated subnet.
- What makes a subnet public.
- How route tables work.
- What a local route is.
- What an Internet Gateway does.
- What a NAT Gateway does.
- Why private instances use NAT.
- What an Elastic IP is.
- How VPC DNS works.
- What DHCP options are.
- What a VPC endpoint does.
- VPC Peering vs Transit Gateway.
- How a packet moves from the internet to an application.
- How a private EC2 reaches the internet.
- How an EC2 connects to RDS.

Do not move to Phase 3.1 until you can draw packet flow manually.

---

# Final Mental Model

```text
VPC
 |
 +-- CIDR
 |
 +-- Subnets
 |     |
 |     +-- Public
 |     +-- Private
 |     +-- Isolated
 |
 +-- Route Tables
 |
 +-- Internet Gateway
 |
 +-- NAT Gateway
 |
 +-- VPC Endpoints
 |
 +-- VPC Peering
 |
 +-- Transit Gateway
 |
 +-- DNS / DHCP
 |
 +-- Security Groups
 |
 +-- Network ACLs
```

The key question is always:

> Where is the packet going, what route does it take, and what security control can stop it?
