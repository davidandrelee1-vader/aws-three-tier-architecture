# AWS Three-Tier Web Architecture

## Project Overview

This project demonstrates how to build a production-style three-tier web architecture in AWS.

## Architecture

[(insert architecture diagram)](AWS Architecture diagram.png)

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

## Phase 5 - Configure Security Groups

Three security groups were created to implement layered network security across the three-tier architecture.

### Security Group Design

| Security Group | Purpose |
|---------------|---------|
| alb-sg | Allows inbound HTTP traffic from the internet |
| app-sg | Allows HTTP traffic only from the Application Load Balancer |
| db-sg | Allows MySQL connections only from the EC2 application servers |

### Screenshot 1 - Security Groups Created

![Security Groups](15-security-groups-created.png)

### Screenshot 2 - ALB Security Group

![ALB SG](16-alb-security-group.png)

### Screenshot 3 - Application Security Group

![Application SG](17-app-security-group.png)

### Screenshot 4 - Database Security Group

![Database SG](18-db-security-group.png)

### Why?

Using separate security groups for each tier enforces the principle of least privilege. The Application Load Balancer is the only component exposed to the internet, application servers accept traffic only from the load balancer, and the database accepts connections only from the application tier. This layered approach improves security and reduces the attack surface.

### Cloud Engineer Notes

- Created dedicated security groups for each architecture tier.
- Used security group references instead of CIDR blocks for internal communication.
- Restricted direct access to application servers and the database.
- Implemented layered network security following AWS best practices.

### Skills Demonstrated

- Amazon VPC Security Groups
- Least Privilege Access
- Layered Network Security
- Application Load Balancer Security
- EC2 Security
- Amazon RDS Security

## Phase 6 - Launch EC2 Web Server

A private Amazon EC2 instance was launched in the application subnet to host the web application. The instance was configured using EC2 User Data to automatically install the Apache web server, enable the service at boot, and deploy a custom HTML page displaying the instance ID and Availability Zone.

### Screenshot 1 - EC2 Launch Configuration

![EC2 Launch Configuration](19-ec2-launch-configuration.png)

### Screenshot 2 - EC2 Instance Running

![EC2 Instance Running](20-ec2-instance-running.png)

### Screenshot 3 - EC2 Networking

![EC2 Networking](21-ec2-networking.png)

### Why?

The EC2 instance was deployed in a private application subnet to prevent direct internet access while allowing traffic only from the Application Load Balancer. During launch, a User Data script automatically configured the web server, demonstrating infrastructure automation and reducing the need for manual server configuration.

### Cloud Engineer Notes

- Launched an Amazon Linux 2023 EC2 instance in a private application subnet.
- Associated the instance with the `app-sg` security group.
- Disabled public IP assignment to keep the instance private.
- Used EC2 User Data to automatically install and configure Apache.
- Prepared the instance to become the golden image for the Auto Scaling Group.

### Skills Demonstrated

- Amazon EC2
- Amazon Linux 2023
- EC2 User Data
- Infrastructure Automation
- Private Subnet Deployment
- Apache HTTP Server
- Security Groups

  ## Phase 7 - Configure Application Load Balancer

An internet-facing Application Load Balancer (ALB) and Target Group were deployed to distribute incoming HTTP requests to the private EC2 application server. Health checks verified that the application was successfully deployed and responding to client requests.

### Screenshot 1 - Target Group Created

![Target Group](22-target-group-created.png)

### Screenshot 2 - Registered Target (Healthy)

![Target Group Healthy](25-target-healthy.png)

### Screenshot 3 - Application Load Balancer Active

![Application Load Balancer](24-application-load-balancer-active.png)

### Screenshot 4 - Website Accessed Through ALB

![Website Through ALB](26-website-through-alb.png)

### Why?

The Application Load Balancer acts as the single entry point for web traffic, distributing requests across application servers while continuously monitoring their health. This improves scalability, availability, and fault tolerance by ensuring traffic is only sent to healthy instances.

### Cloud Engineer Notes

- Created an Application Load Balancer in two public subnets.
- Configured an HTTP listener on port 80.
- Created a Target Group using instance targets.
- Registered the EC2 application server with the Target Group.
- Verified successful health checks and application accessibility through the ALB DNS name.

### Skills Demonstrated

- Application Load Balancer (ALB)
- Target Groups
- Health Checks
- High Availability
- Layer 7 Load Balancing
- AWS Networking

## Phase 8 - Create the Golden AMI

A custom Amazon Machine Image (AMI) was created from the configured EC2 web server. This AMI serves as the golden image for future deployments, ensuring that every EC2 instance launched by the Auto Scaling Group has an identical operating system, web server configuration, and application content.

### Screenshot 1 - AMI Available

![AMI Available](27-ami-available.png)

### Screenshot 2 - EBS Snapshot Created

![EBS Snapshot](28-ebs-snapshot-created.png)

### Why?

Creating a golden AMI provides a standardized and reusable server image for future deployments. Instead of manually configuring each EC2 instance, the Auto Scaling Group can launch identical instances from the AMI, ensuring consistency, reducing deployment time, and minimizing configuration drift.

### Cloud Engineer Notes

- Created a custom AMI from the configured EC2 web server.
- Captured the Amazon Linux 2023 operating system, Apache web server, and deployed application.
- Generated an Amazon EBS snapshot to preserve the instance volume.
- Prepared the AMI for use in a Launch Template and Auto Scaling Group.

### Skills Demonstrated

- Amazon Machine Image (AMI)
- Amazon EBS Snapshots
- Infrastructure Standardization
- Immutable Infrastructure
- Auto Scaling Preparation

## Phase 9 - Create the Launch Template

A Launch Template was created using the custom Amazon Machine Image (AMI). This template defines the EC2 configuration that the Auto Scaling Group will use to automatically launch replacement and additional instances.

### Screenshot 1 - Launch Template Created

![Launch Template Created](29-launch-template-created.png)

### Screenshot 2 - Launch Template Details

![Launch Template Details](30-launch-template-details.png)

### Why?

A Launch Template standardizes EC2 deployments by defining the instance configuration in a reusable template. This allows the Auto Scaling Group to launch identical instances from the golden AMI, ensuring consistency, reducing manual configuration, and supporting automated scaling.

### Cloud Engineer Notes

- Created a Launch Template using the custom AMI.
- Configured the template to use the `app-sg` security group.
- Selected the `t3.micro` instance type.
- Left subnet selection to the Auto Scaling Group for multi-AZ deployments.
- Used the Launch Template as the foundation for automated instance provisioning.

### Skills Demonstrated

- EC2 Launch Templates
- Amazon Machine Images (AMI)
- Infrastructure Standardization
- Auto Scaling Preparation
- Immutable Infrastructure

## Phase 10 - Configure Auto Scaling Group

An Auto Scaling Group was created using the custom Launch Template to automatically deploy and maintain EC2 application servers across two Availability Zones. The Auto Scaling Group was integrated with the existing Application Load Balancer and Target Group to provide high availability, automated recovery, and scalable application capacity.

### Screenshot 1 - Auto Scaling Group Created

![Auto Scaling Group](31-autoscaling-group-created.png)

### Screenshot 2 - Auto Scaling Instances Running

![ASG Instances](32-asg-instances-running.png)

### Screenshot 3 - Auto Scaling Group Healthy Targets

![ASG Healthy Targets](33-asg-targets-healthy.png.png)

### Why?

The Auto Scaling Group improves application resilience by maintaining the required number of EC2 instances and automatically replacing failed instances. Distributing instances across two Availability Zones reduces the impact of a single Availability Zone failure, while the Application Load Balancer routes traffic only to healthy application servers.

### Cloud Engineer Notes

- Created an Auto Scaling Group using the custom Launch Template.
- Configured a desired capacity of two EC2 instances.
- Distributed application servers across two Availability Zones.
- Integrated the Auto Scaling Group with the existing Target Group.
- Enabled Elastic Load Balancer health checks.
- Configured dynamic scaling based on average CPU utilization.
- Verified that all Auto Scaling Group targets successfully passed health checks.

### Skills Demonstrated

- Amazon EC2 Auto Scaling
- EC2 Launch Templates
- Application Load Balancer (ALB)
- Target Groups
- Health Checks
- High Availability
- Fault Tolerance
- Dynamic Scaling

## Phase 11 - Deploy Amazon RDS

An Amazon RDS for MySQL database was deployed within the private database tier of the three-tier architecture. A dedicated DB Subnet Group was created using the two private database subnets, ensuring the database is isolated from direct internet access. Database connectivity is secured using a dedicated security group that only permits MySQL traffic from the application tier.

### Screenshot 1 - Database Subnet Group

![Database Subnet Group](34-db-subnet-group-created.png)

### Screenshot 2 - Amazon RDS Available

![Amazon RDS Available](35-rds-database-available.png)

### Screenshot 3 - Database Security Group

![Database Security Group](36-rds-security-group-rules.png)

### Why?

Amazon RDS provides a fully managed relational database service that automates database provisioning, backups, patching, and maintenance. Deploying the database within private subnets prevents direct internet access, while restricting inbound MySQL traffic to the application security group (`app-sg`) follows the principle of least privilege and strengthens the overall security of the three-tier architecture.

### Cloud Engineer Notes

- Created a dedicated DB Subnet Group spanning two Availability Zones.
- Deployed an Amazon RDS for MySQL database in private database subnets.
- Disabled public accessibility to prevent direct internet access.
- Configured the `db-sg` security group to allow MySQL (TCP 3306) traffic only from the `app-sg` security group.
- Isolated the database tier from the public internet while allowing secure communication from the application tier.
- Protected database credentials by excluding passwords and sensitive information from project documentation and screenshots.

### Skills Demonstrated

- Amazon RDS
- MySQL
- DB Subnet Groups
- Private Database Deployment
- VPC Networking
- Security Group Referencing
- Network Segmentation
- Managed Database Services
- AWS Security Best Practices
