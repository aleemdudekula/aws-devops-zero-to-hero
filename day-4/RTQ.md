# AWS VPC Networking – Practical DevOps Guide

This document explains key **Amazon VPC networking components** and the **real problems they solve in production environments**.

---

# 1. VPC (Virtual Private Cloud)

## What it is

A **VPC** is your private network inside AWS.

You define:

* IP range (CIDR)
* Subnets
* Routing
* Security

### Example

```
VPC CIDR
10.0.0.0/16
```

---

## Real Production Problem

Without VPC:

* All EC2 instances would exist in one shared AWS network
* No isolation between companies
* Security risks across workloads

---

## Real Scenario

Company infrastructure inside a VPC:

```
VPC
10.0.0.0/16

Frontend servers
Backend servers
Databases
Load balancers
```

Everything runs inside **one isolated network**.

---

# 2. Subnets

## What it is

A **subnet** is a smaller network inside the VPC.

Each subnet belongs to **one Availability Zone**.

### Example

```
VPC
10.0.0.0/16

Public Subnet
10.0.1.0/24

Private Subnet
10.0.2.0/24
```

---

## Real Production Problem

You cannot expose everything to the internet.

Applications must separate layers.

| Layer    | Internet Access |
| -------- | --------------- |
| Frontend | Yes             |
| Backend  | No              |
| Database | Never           |

---

## Real Architecture

```
Public Subnet
- Load Balancer
- Bastion Host

Private Subnet
- Application servers

Private Subnet
- Database
```

This is a **standard 2-tier / 3-tier architecture in AWS**.

---

# 3. IP Addressing

## What it is

Every AWS resource requires an IP address.

| Type       | Use                    |
| ---------- | ---------------------- |
| Private IP | Internal communication |
| Public IP  | Internet access        |
| Elastic IP | Static public IP       |

---

## Real Production Problem

Example problem:

```
You deploy an EC2 server
Users cannot access website
```

Why?

Because **no public IP was assigned**.

---

### Solution

```
Elastic IP → attach to EC2
```

---

### Another Issue

Two networks using the same CIDR.

```
On-prem
10.0.0.0/16

AWS
10.0.0.0/16
```

Result:

```
VPN fails because of IP conflict
```

---

# 4. Network Access Control List (NACL)

## What it is

A **Network ACL** is a firewall at the **subnet level**.

Rules include:

* Allow / Deny
* IP
* Port
* Protocol

Important characteristic:

```
Stateless
```

Meaning:

Inbound and outbound rules must both exist.

---

## Real Production Problem

Example: blocking malicious IP ranges.

Attack source:

```
45.12.0.0/16
```

Solution:

```
NACL Rule
DENY 45.12.0.0/16
```

All servers in that subnet are protected.

---

# 5. Security Groups

## What it is

A **Security Group** is a firewall attached to instances.

Example:

```
EC2 instance security group
```

Important:

```
Stateful
```

If inbound traffic is allowed → outbound response is automatically allowed.

---

## Real Production Problem

Example:

```
Web server deployed
Website not opening
```

Problem:

```
Port 80 not allowed
```

Solution:

```
Security Group Rule

Inbound
HTTP 80
0.0.0.0/0
```

---

### Database Example

```
MySQL 3306
Allowed only from app servers
```

---

# 6. Route Tables

## What it is

Route tables decide **where network traffic goes**.

Example route:

```
Destination      Target
0.0.0.0/0        Internet Gateway
```

---

## Real Production Problem

Your EC2 instance has:

* Public IP
* Security group open

But website still not reachable.

Reason:

```
No route to Internet Gateway
```

---

### Fix

```
Route table
0.0.0.0/0 → IGW
```

---

# 7. Gateways & Endpoints

## Internet Gateway (IGW)

Allows internet access for public subnet resources.

Example:

```
Internet → IGW → EC2
```

---

## NAT Gateway

Private servers need internet for updates but must not be exposed.

Example tasks:

```
apt update
docker pull
```

Architecture:

```
Private EC2 → NAT Gateway → Internet
```

---

## VPC Endpoint

Access AWS services **without using the internet**.

Example:

```
Private EC2 → S3
```

Without endpoint:

```
Traffic goes through internet
```

With endpoint:

```
Private AWS network
```

Benefits:

* More secure
* Faster
* No internet dependency

---

# 8. VPC Peering

## What it is

Connect two VPC networks directly.

Example:

```
VPC-A → VPC-B
```

---

## Real Production Problem

Organizations often separate environments.

```
VPC-DEV
VPC-STAGING
VPC-PROD
```

Some services must communicate across VPCs.

Example:

```
Analytics VPC → Production Database
```

Solution:

```
VPC Peering
```

---

# 9. Traffic Mirroring

## What it is

Copies network packets for inspection.

Used by:

* IDS
* IPS
* Security monitoring systems

---

## Real Production Problem

Example scenario:

Security team suspects **data exfiltration**.

Traffic is mirrored from:

```
EC2 network interface
```

Sent to:

```
Security appliance
```

For packet inspection.

---

# 10. Transit Gateway

## What it is

A **central hub** connecting many VPCs.

Without it:

```
VPC1 ↔ VPC2
VPC2 ↔ VPC3
VPC3 ↔ VPC4
```

This becomes difficult to manage.

---

## Real Production Problem

Large companies may operate:

```
50+ VPCs
```

Without Transit Gateway:

```
Hundreds of peering connections
```

With Transit Gateway:

```
All VPCs connect to a central router
```

Much easier network management.

---

# 11. VPC Flow Logs

## What it is

Logs network traffic metadata.

Example log data:

```
Source IP
Destination IP
Port
Allow / Deny
```

---

## Real Production Problem

Example issue:

```
Application cannot connect to database
```

Flow logs reveal:

```
Traffic denied
Security group blocking
```

Also useful for detecting suspicious traffic.

---

# 12. VPN Connections

## What it is

Connect on-premise networks to AWS VPC.

Example:

```
Office network → AWS
```

---

## Real Production Problem

Companies migrating to cloud still depend on:

* On-prem databases
* Internal enterprise systems
* Corporate networks

They need secure connectivity.

Solution:

```
Site-to-Site VPN
```

Architecture:

```
Office Router
     │
Encrypted VPN Tunnel
     │
AWS VPC
```

---

# Real Production Architecture Example

Typical DevOps production setup:

```
VPC
│
├── Public Subnet
│     ├── Load Balancer
│     └── NAT Gateway
│
├── Private Subnet
│     └── Application Servers
│
├── Private Subnet
│     └── Database
│
├── Security Groups
├── NACL
├── Route Tables
└── Internet Gateway
```

---

# Hard Reality (Most Beginners Miss This)

Knowing definitions is useless.

Real DevOps skill is understanding:

* Why traffic fails
* Why instances cannot communicate
* How to debug networking issues

Most common AWS outages happen due to:

* Security group misconfiguration
* Route table mistakes
* NACL blocking traffic
* NAT gateway misconfiguration
