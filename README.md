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

![Six Subnets Created](04-subnets-created.png)

### Screenshot 2 - Subnets Across Two Availability Zones

![Subnets Across Two Availability Zones](05-subnets-across-two-azs.png)

### Screenshot 3 - Private Subnet Details

![Private Subnet Details](06-private-subnet-details.png)

### Why?

Separating the architecture into public, application, and database subnets limits direct exposure to the internet. Distributing the subnets across two Availability Zones also improves resilience and prepares the environment for a highly available load balancer, Auto Scaling group, and RDS deployment.

### Cloud Engineer Notes

- Designed a three-tier network architecture using dedicated public, application, and database subnets.
- Distributed resources across two Availability Zones to improve availability and resilience.
- Reserved private subnets for application and database resources to minimize internet exposure.
- Prepared the network for an Application Load Balancer, Auto Scaling Group, NAT Gateway, and Amazon RDS deployment.
  
## Phase 3 - Configure Route Tables

Three custom route tables were created to control traffic for the public, application, and database tiers.

### Route Table Design

| Route Table | Subnets | Internet Route |
|---|---|---|
| public-rt | public-subnet-az1 and public-subnet-az2 | 0.0.0.0/0 → Internet Gateway |
| app-private-rt | app-subnet-az1 and app-subnet-az2 | Local route only |
| db-private-rt | db-subnet-az1 and db-subnet-az2 | Local route only |

### Screenshot 1 - Route Tables Created

![Route Tables](07-route-tables-created.png)

### Screenshot 2 - Public Internet Route

![Public Route](08-public-route-to-internet-gateway.png)

### Screenshot 3 - Public Subnet Associations

![Public Associations](09-public-subnet-associations.png)

### Screenshot 4 - Private Route Table Associations

![Private Associations](10-private-route-table-associations.png)

### Why?

The public route table sends internet-bound traffic to the Internet Gateway, making its associated subnets public. The application and database route tables currently contain only the local VPC route, preventing direct internet access and maintaining separation between the architecture tiers.

## Phase 4 - Configure NAT Gateway

A public NAT Gateway was deployed to provide secure outbound internet access for application servers hosted in private subnets. The NAT Gateway allows EC2 instances to download updates and access external services without exposing them directly to inbound internet traffic.

### Screenshot 1 - Elastic IP Allocated

![Elastic IP](11-nat-elastic-ip.png)

### Screenshot 2 - NAT Gateway Available

![NAT Gateway](12-nat-gateway-created.png)

### Screenshot 3 - Application Route Table

![Application Route](13-app-route-table-nat.png)

### Screenshot 4 - Database Route Table

![Database Route](14-db-route-table-isolated.png)

### Why?

A NAT Gateway provides outbound internet connectivity for private resources while blocking unsolicited inbound connections. This design follows AWS security best practices by keeping application servers private while still allowing them to access software repositories and AWS services. The database subnets remain isolated to reduce the attack surface.

### Cloud Engineer Notes

- Allocated an Elastic IP for the NAT Gateway.
- Deployed a public NAT Gateway.
- Updated the application route table to route outbound traffic through the NAT Gateway.
- Left the database route table isolated with only the local VPC route.

### Skills Demonstrated

- Amazon NAT Gateway
- Elastic IP
- Route Tables
- Private Networking
- Secure Internet Access
- AWS Networking Best Practices
