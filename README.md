# AWS Infrastructure Documentation: Demo Website (VDarius.IT.com)

## Overview

This repository documents the AWS cloud infrastructure designed and implemented to host a demo website, VDarius.IT.com. This project serves as a practical learning exercise in deploying web applications on AWS, focusing on utilizing the AWS Free Tier while incorporating fundamental security, networking, and operational best practices.

The infrastructure supports a Node.js based web application requiring a relational database backend. Key considerations during the design and implementation included cost efficiency, foundational security measures, and laying the groundwork for future enhancements like high availability and more robust monitoring.

## Architecture

The infrastructure is deployed within the AWS **EU Frankfurt (eu-central-1)** region, leveraging multiple Availability Zones for certain components to enhance resilience.

A high-level view of the architecture includes:

-   A **Virtual Private Cloud (VPC)** configured with specific subnets and routing.
-   An **EC2 Instance** hosting the web server (Nginx) and the Node.js application.
-   An **RDS MySQL Instance** providing the database service.
-   Layered **Security Controls** including Security Groups and host-based firewalls.
-   **IAM** for secure access management.
-   (Partially implemented) **Cognito** for user authentication flows.

+-----------------------------------------------------------------+
|                   AWS Region: EU Frankfurt (eu-central-1)       |
|                                                                 |
|   +-----------------------------------------------------------+ |
|   |                    VPC (CIDR: /18)                        | |
|   |                                                           | |
|   | +-------------------------+     +-------------------------+ | 
|   | |    Public Subnet        |     |   Database Subnets      | | 
|   | |      (/22)              |     |   (/25 x 3 AZs)         | | 
|   | |                         |     |                         | | 
|   | |  [SG EC2] <---+-------  |     |  [SG RDS] <---+-------- | | 
|   | |      |        | HTTP/S  |     |      |        | MySQL   | | 
|   | |      v        | (80,443)|     |      v        | (3306)  | | 
|   | | [EC2 Instance]|         |     | [RDS MySQL]   |         | | 
|   | |(Nginx,Node.js)|         |     | (Private)     |         | | 
|   | | (Public IP)   |         |     |               |         | | 
|   | +-------------------------+     +-------------------------+ | 
|   |         ^                                                 | |
|   |         | SSH (22)                                        | |
|   +---------|-------------------------------------------------+ |
|             |                                                   |
|   Internet --                                                   |
|   Admin   ----                                                  |
|                                                                 |
|  [IAM] ----------------> Manages AWS Resources                  |
|  [Cognito] -------------> User Authentication (Partial)         |
+-----------------------------------------------------------------+


## Network Configuration

The network infrastructure is built around a single VPC, segmented into multiple subnets across different Availability Zones to provide logical isolation and prepare for higher availability configurations.

### Virtual Private Cloud (VPC)

-   **Region**: eu-central-1 (Frankfurt)
-   **VPC CIDR Block**: `/18` - Configured to encompass all subnets used within the VPC, facilitating internal communication.
-   **Availability Zones Used**: `eu-central-1a`, `eu-central-1b`, `eu-central-1c`

### Subnet Configuration

Subnets are strategically placed within the VPC and across AZs:

| Subnet Type           | CIDR Block | Availability Zones Used | Purpose                                                          |
| :-------------------  | :--------- | :---------------------- | :--------------------------------------------------------------- |
| Public/Application    | `/22`      | Multiple AZs            | Hosts the EC2 instance and other public-facing resources.        |
|                       |            |                         | Used for resources requiring direct internet access.             |
| Database (RDS) - AZ A | `/25`      | eu-central-1a           | Dedicated subnet for RDS instance placement (part of a DB Subnet |
|                       |            |                         | Group).                                                          |
| Database (RDS) - AZ B | `/25`      | eu-central-1b           | Dedicated subnet for RDS instance placement (part of a DB Subnet |
|                       |            |                         | Group).                                                          |
| Database (RDS) - AZ C | `/25`      | eu-central-1c           | Dedicated subnet for RDS instance placement (part of a DB Subnet |
|                       |            |                         | Group).                                                          |

*Note: The database subnets are smaller `/25` blocks carved out of the larger `/18` VPC CIDR.*

### Routing

-   **Internet Gateway (IGW)**: Attached to the VPC to allow communication between resources in the VPC and the public internet.
-   **Route Tables**:
    -   **Public Route Table**: Associated with the Public/Application subnet. Contains a default route (`0.0.0.0/0`) pointing to the Internet Gateway, enabling resources in this subnet (like the EC2 instance) to access and be accessed from the internet (subject to Security Group/NACL rules).
    -   **Private Route Table**: Associated with the Database subnets. Contains routes only for local VPC communication, preventing direct internet access to the database instances.
-   **NAT Gateway**: Was tested during initial setup but subsequently removed as it was not required for the current architecture (no instances in private subnets needed outbound internet access initiated by them).

## Security Configuration

Security is implemented in layers, from AWS console access down to the instance level.

### AWS Account and Console Access

-   **MFA Protection**: Multi-Factor Authentication is enforced for all AWS console logins.
-   **Password Policy**: Strong password requirements are enforced.
-   **Suspicious Activity Monitoring**: Email notifications are configured to alert on detected suspicious login activities.

### Network Access Control

-   **Security Groups (Stateful Firewall)**:
    -   **EC2 Security Group**:
        -   Inbound:
            -   HTTP (Port 80): Allowed from `0.0.0.0/0` (anywhere) - for public web access.
            -   HTTPS (Port 443): Allowed from `0.0.0.0/0` (anywhere) - for public web access.
            -   SSH (Port 22): Restricted to a specific administrator IP address only - securing management access.
        -   Outbound:
            -   All Traffic: Allowed to `0.0.0.0/0` (anywhere) - for general internet access (e.g., updates).
            -   MySQL/Aurora (Port 3306): Allowed *only* to the **RDS Security Group** - ensuring the application can connect to the database, but not other external DBs.
    -   **RDS Security Group**:
        -   Inbound:
            -   MySQL/Aurora (Port 3306): Allowed *only* from the **EC2 Security Group** - restricting database access solely to the application server.
        -   Outbound: Restricted (typically allows outbound to required AWS services or is limited based on needs).

-   **Network Access Control Lists (NACLs) (Stateless Firewall)**:
    -   Configured at the subnet level.
    -   Rules include:
        -   Rule 100 (Inbound/Outbound): Allows specific traffic related to HTTP/HTTPS and ephemeral ports for web traffic return.
        -   Default Rules (`*`): Explicitly denies all other traffic (this is the default behavior for NACLs).
      *(Note: NACLs operate stateless, meaning both inbound and outbound rules must explicitly permit traffic in both directions, including ephemeral ports for return traffic.)*

### Instance-Level Security

-   **SSH Key Authentication**: Password authentication for SSH is disabled; access requires a specific public/private key pair.
-   **UFW Firewall**: Uncomplicated Firewall is enabled on the EC2 instance, providing an additional layer of host-based firewall protection.
-   **Nginx Configuration**: The Nginx web server is configured with security in mind, serving as a reverse proxy for the Node.js application.

## Database Configuration

The application utilizes an Amazon RDS for MySQL instance.

-   **Database Engine**: MySQL (Free Tier eligible)
-   **Deployment**: Configured within a **DB Subnet Group** spanning three Availability Zones (`eu-central-1a`, `eu-central-1b`, `eu-central-1c`). This setup is a prerequisite for Multi-AZ deployments, allowing for potential future HA upgrades.
-   **Accessibility**: The instance is configured as "private" (not publicly accessible), residing within the dedicated database subnets and only reachable from the EC2 instance via the configured Security Group rules.
-   **Automated Backups**: Daily automated backups are enabled, providing point-in-time recovery capabilities.

## Identity and Access Management (IAM)

IAM is used to manage access to AWS resources.

-   **Admin User Group**: A dedicated group for administrators (the primary user) with policies granting necessary permissions for managing the infrastructure.
-   **Principle of Least Privilege**: Efforts are made to assign only the minimum necessary permissions via custom or managed policies.
-   **Cognito Integration**: An attempt was made to integrate AWS Cognito User Pools and Identity Pools for managing user registration and login for the website. This implementation is currently paused due to technical challenges encountered during token handling and different API, but represents a planned feature for secure user authentication.

## Monitoring and Alerting

Basic monitoring and alerting are in place, with plans for expansion.

-   **Billing Alerts**: A budget alarm is set to $0, triggering notifications to prevent unexpected costs, ensuring adherence to Free Tier limits.
-   **Login Monitoring**: Email alerts are configured for suspicious login attempts on the AWS account.
-   **Basic CloudWatch Metrics**: AWS provides default CloudWatch metrics for EC2 and RDS instances (e.g., CPU utilization, network traffic, database connections).
-   **CloudWatch Alarms**: Setting thresholds on key metrics (e.g., high CPU, low disk space, high network traffic, high database connections) to trigger notifications.
-   **CloudWatch Logs**: Collecting system logs (e.g., `syslog`), application logs (Node.js output), and web server logs (Nginx access/error logs) for analysis and troubleshooting.
-   **CloudWatch Dashboards**: Creating visual summaries of key metrics and alarms for quick operational oversight.
-   **Application-Level Monitoring**: Implementing logging and metrics within the Node.js application itself to track performance and errors.

## High Availability and Disaster Recovery

This initial infrastructure design prioritizes cost-effectiveness and learning within the AWS Free Tier constraints. While this current configuration provides a functional environment for a demo website, it is built with a understanding of industry-standard High Availability (HA) and Disaster Recovery (DR) principles. Considerations for enhancing resilience in the future are documented, acknowledging trade-offs made in the current low-cost implementation.

-   **Current Design & Resilience Characteristics**:
    -   The current application tier, utilizing a **single EC2 instance**, represents a single point of failure. Should this instance become unavailable, the website would experience downtime.
    -   The EC2 instance is configured with a **dynamic public IP address**. Changes to this IP (e.g., upon instance stop/start or replacement) require manual updates to external DNS records, introducing a potential period of unavailability during the update process.
    -   The RDS database is placed within a **Multi-AZ Subnet Group**, which is a prerequisite for a Multi-AZ deployment. However, under Free Tier constraints, it is currently configured as a Single-AZ instance. This means automatic failover to a standby replica in another Availability Zone is not enabled, and recovery from a database instance failure would require restoration from backups.
    -   Recovery procedures in the current setup primarily rely on restoring from existing backups (AWS automated for RDS, custom script for application files) and manually reconfiguring/relaunching components, which involves manual intervention and potential downtime.

-   **Future Enhancements for High Availability & Disaster Recovery**:
    Demonstrating an understanding of robust cloud architecture and resilience strategies, the roadmap for enhancing this infrastructure includes:

    *   **Application Tier High Availability (In-Region)**: Implementing an **Auto Scaling Group (ASG)** spread across multiple Availability Zones (`eu-central-1a`, `eu-central-1b`, `eu-central-1c`). This would automatically maintain the desired number of instances and replace unhealthy ones. A **Load Balancer (ALB)** would be implemented in front of the ASG to distribute incoming traffic and provide a stable, highly available endpoint DNS name, eliminating manual DNS updates upon instance changes.
    *   **Database Tier High Availability (In-Region)**: Upgrading the RDS instance to a **Multi-AZ Deployment**. This automatically provisions and maintains a synchronous standby replica in a different Availability Zone. In the event of a primary instance failure or AZ outage, AWS automatically handles failover to the standby with minimal recovery time. (Note: This enhancement typically moves the RDS instance beyond Free Tier eligibility).
    *   **Data Resilience**: Reviewing and enhancing data persistence strategies, ensuring not only database backups (already in place) but also considering shared file systems like Amazon EFS if the application requires shared state across multiple potential instances in an ASG configuration.
    *   **Disaster Recovery (Cross-Region)**: For a more comprehensive recovery strategy against regional outages, implementing a service like **AWS Elastic Disaster Recovery (DRS)** would be considered. DRS offers a cost-effective solution for disaster recovery by enabling continuous block-level replication of instances to a low-cost staging area in a different AWS Region. This allows for rapid and reliable recovery with minimal data loss (low RPO) and quick restoration of services (low RTO) by automating the process of launching instances in the recovery region using the replicated data. While not necessary for the current demo project and introducing costs beyond the Free Tier (for replication and staging), understanding and planning for a robust DR solution like DRS is crucial for production-grade systems, providing a significant step up from reliance solely on backups.

Implementing these documented HA/DR strategies represents a progression towards a more resilient and production-ready architecture, demonstrating the ability to design systems that can withstand various failure scenarios.

## Deployment and Operations

Procedures are in place for deploying updates and managing data.

-   **Application Deployment**: Updates to the Node.js application and Nginx configuration are primarily handled via SSH access to the EC2 instance, using tools like Git to pull the latest code.
-   **Database Management**: Database schema changes and data manipulation are performed manually via standard MySQL clients or potentially via `.sh` scripts as needed.
-   **Backup & Restore**:
    -   RDS: Automated daily backups handled by AWS. Restoration can be initiated from the RDS console.
    -   Application: Custom `.sh` scripts are available on the EC2 instance for backing up website files and the application code, along with a corresponding script for restoring from these backups both localy and my private github repository.
-   **Maintenance**: Operating system patching and software updates on the EC2 instance are managed manually. Database engine patching is typically managed by AWS for RDS, but maintenance windows should be monitored.

## Cost Management

Operating within the AWS Free Tier limits is a primary objective of this project.

-   **Free Tier Utilization**: All deployed services (VPC, EC2 t2.micro, RDS db.t2.micro, Storage, Data Transfer within limits) are selected to fall within the AWS Free Tier eligibility.
-   **Budget Alarm**: A zero-dollar budget is set up with notifications to ensure immediate alerts if costs exceed the Free Tier thresholds.
-   **Regular Monitoring**: Periodic review of the AWS Cost Explorer dashboard helps confirm that usage remains within limits.

## Future Enhancements

Based on the current implementation and learning goals, potential future enhancements include:

-   Completing the AWS Cognito integration for robust and scalable user authentication.
-   Exploring Infrastructure as Code (IaC) using tools like AWS CloudFormation or Terraform for automated, repeatable infrastructure deployment and management.
-   Implementing the High Availability and Disaster Recovery strategies discussed, including Auto Scaling Groups, Load Balancers, and potentially RDS Multi-AZ (understanding this moves beyond Free Tier).
-   Setting up a continuous Integration/Continuous Deployment (CI/CD) pipeline for automated code deployments.
-   Further refining security configurations.

## Conclusion

This project demonstrates the ability to design, deploy, and manage a basic web application infrastructure on AWS, leveraging Free Tier resources while incorporating fundamental cloud concepts such as VPC networking, security groups, managed databases, and access control. It serves as a foundation for exploring more advanced AWS services and architectural patterns, particularly in the areas of automation, monitoring, and high availability.

---
