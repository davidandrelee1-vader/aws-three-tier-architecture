# AWS Three-Tier Web Architecture

## Project Overview

This project demonstrates how to build a production-style three-tier web architecture in AWS.

## Architecture

(insert architecture diagram)

## Technologies

- Amazon VPC
- EC2
- Auto Scaling
- Application Load Balancer
- Amazon RDS
- Security Groups
- IAM

## Step 1 - Create the VPC

Created a custom VPC named three-tier-vpc using the CIDR block 10.0.0.0/16.

### Screenshot 1 - VPC Created

![VPC Created](vpc-created.png)

### Screenshot 2 - DNS Settings

![DNS Settings](02-vpc-dns-settings.png)

### Screenshot 3 - Internet Gateway Attached

![Internet Gateway](03-internet-gateway-attached.png)

### Why?

A custom VPC provides complete control over networking, routing, and security.

---

## Phase 2 - Create Public and Private Subnets

Six subnets were created across two Availability Zones to support high availability and isolate the presentation, application, and database tiers.

### Subnet Design

| Subnet | CIDR Block | Purpose |
|---|---|---|
| public-subnet-az1 | 10.0.1.0/24 | Public-facing infrastructure in AZ1 |
| public-subnet-az2 | 10.0.2.0/24 | Public-facing infrastructure in AZ2 |
| app-subnet-az1 | 10.0.11.0/24 | Application servers in AZ1 |
| app-subnet-az2 | 10.0.12.0/24 | Application servers in AZ2 |
| db-subnet-az1 | 10.0.21.0/24 | Database resources in AZ1 |
| db-subnet-az2 | 10.0.22.0/24 | Database resources in AZ2 |

### Screenshot 1 - Six Subnets Created

![Six Subnets Created](04-six-subnets-created.png)

### Screenshot 2 - Subnets Across Two Availability Zones

![Subnets Across Two Availability Zones](05-subnets-across-two-azs.png)

### Screenshot 3 - Private Subnet Details

![Private Subnet Details](06-private-subnet-details.png)

### Why?

Separating the architecture into public, application, and database subnets limits direct exposure to the internet. Distributing the subnets across two Availability Zones also improves resilience and prepares the environment for a highly available load balancer, Auto Scaling group, and RDS deployment.

## Step 3...

...
