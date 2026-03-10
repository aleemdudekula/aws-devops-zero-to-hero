# VPC

Imagine you want to set up a private, secure, and isolated area in the cloud where you can run your applications and store your data. This is where a VPC comes into play.

A VPC is a virtual network that you create in the cloud. It allows you to have your own private section of the internet, just like having your own network within a larger network. Within this VPC, you can create and manage various resources, such as servers, databases, and storage.

Think of it as having your own little "internet" within the bigger internet. This virtual network is completely isolated from other users' networks, so your data and applications are secure and protected.

Just like a physical network, a VPC has its own set of rules and configurations. You can define the IP address range for your VPC and create smaller subnetworks within it called subnets. These subnets help you organize your resources and control how they communicate with each other.

To connect your VPC to the internet or other networks, you can set up gateways or routers. These act as entry and exit points for traffic going in and out of your VPC. You can control the flow of traffic and set up security measures to protect your resources from unauthorized access.

With a VPC, you have control over your network environment. You can define access rules, set up firewalls, and configure security groups to regulate who can access your resources and how they can communicate.

![image](https://github.com/iam-veeramalla/aws-devops-zero-to-hero/assets/43399466/12cc10b6-724c-42c9-b07b-d8a7ce124e24)

By default, when you create an AWS account, AWS will create a default VPC for you but this default VPC is just to get started with AWS. You should create VPCs for applications or projects. 

## VPC components 

The following features help you configure a VPC to provide the connectivity that your applications need:

# Virtual private clouds (VPC)

    A VPC is a virtual network that closely resembles a traditional network that you'd operate in your own data center. 
    After you create a VPC, you can add subnets. 

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


Subnets

    A subnet is a range of IP addresses in your VPC. A subnet must reside in a single Availability Zone. 
    After you add subnets, you can deploy AWS resources in your VPC.

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

IP addressing

    You can assign IP addresses, both IPv4 and IPv6, to your VPCs and subnets. 
    You can also bring your public IPv4 and IPv6 GUA addresses to AWS and allocate them to resources in your VPC, 
    such as EC2 instances, NAT gateways, and Network Load Balancers.
    
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

Network Access Control List (NACL)

    A Network Access Control List is a stateless firewall that controls inbound and outbound traffic at the subnet level. 
    It operates at the IP address level and can allow or deny traffic based on rules that you define. 
    NACLs provide an additional layer of network security for your VPC.

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

   
Security Group

    A security group acts as a virtual firewall for instances (EC2 instances or other resources) within a VPC. 
    It controls inbound and outbound traffic at the instance level. 
    Security groups allow you to define rules that permit or restrict traffic based on protocols, ports, and IP addresses.  
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

Routing

    Use route tables to determine where network traffic from your subnet or gateway is directed.
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

Gateways and endpoints

    A gateway connects your VPC to another network. For example, use an internet gateway to connect your VPC to the internet. 
    Use a VPC endpoint to connect to AWS services privately, without the use of an internet gateway or NAT device.
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

Peering connections

    Use a VPC peering connection to route traffic between the resources in two VPCs.
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
Traffic Mirroring

    Copy network traffic from network interfaces and send it to security and monitoring appliances for deep packet inspection.
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

Transit gateways

    Use a transit gateway, which acts as a central hub, to route traffic between your VPCs, VPN connections, 
    and AWS Direct Connect connections.
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

VPC Flow Logs

    A flow log captures information about the IP traffic going to and from network interfaces in your VPC.
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
VPN connections

    Connect your VPCs to your on-premises networks using AWS Virtual Private Network (AWS VPN).
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


## Resources 

VPC with servers in private subnets and NAT

https://docs.aws.amazon.com/vpc/latest/userguide/vpc-example-private-subnets-nat.html

![image](https://github.com/iam-veeramalla/aws-devops-zero-to-hero/assets/43399466/89d8316e-7b70-4821-a6bf-67d1dcc4d2fb)



