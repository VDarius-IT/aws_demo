+-------------------------------------------------------------------------------------------------------------+
|                                                                                                             |
|  AWS Region: eu-central-1 (Frankfurt)                                                                       |
|                                                                                                             |
|  +-------------------------------------------------------------------------------------------------------+  |
|  |                                                                                                       |  |
|  |  VPC (/18 CIDR)                                                                                       |  |
|  |                                                                                                       |  |
|  |  +-----------------------------------+     +-----------------------------------+                      |  |
|  |  |                                   |     |                                   |                      |  |
|  |  |  Public Subnet (/22)              |     |  Route Table                      |                      |  |
|  |  |                                   |     |                                   |                      |  |
|  |  |  +-----------------------------+  |     |  - Route to Internet Gateway      |                      |  |
|  |  |  |                             |  |     |  - Route to VPC local             |                      |  |
|  |  |  |  EC2 Instance               |  |     |                                   |                      |  |
|  |  |  |                             |  |     +-----------------------------------+                      |  |
|  |  |  |  - Nginx                    |  |                                                                |  |
|  |  |  |  - Node.js Application      |  |                                                                |  |
|  |  |  |  - UFW Firewall             |  |                                                                |  |
|  |  |  |                             |  |                                                                |  |
|  |  |  +-----------------------------+  |                                                                |  |
|  |  |           |                       |                                                                |  |
|  |  +-----------------------------------+                                                                |  |
|  |              |                                                                                        |  |
|  |              | Security Group: EC2 → RDS                                                              |  |
|  |              | (MySQL 3306 only)                                                                      |  |
|  |              ↓                                                                                        |  |
|  |  +-----------------------------------+     +-----------------------------------+                      |  |
|  |  |                                   |     |                                   |                      |  |
|  |  |  RDS Subnet Group                 |     |  Security Groups                  |                      |  |
|  |  |                                   |     |                                   |                      |  |
|  |  |  +-----------------------------+  |     |  EC2 Security Group:              |                      |  |
|  |  |  |                             |  |     |  - Allow HTTP/HTTPS from anywhere |                      |  |
|  |  |  |  RDS MySQL Instance         |  |     |  - Allow SSH from admin IP only   |                      |  |
|  |  |  |                             |  |     |  - Allow outbound to internet     |                      |  |
|  |  |  |  - Daily Backups            |  |     |  - Allow MySQL to RDS SG          |                      |  |
|  |  |  |  - Multi-AZ Subnets         |  |     |                                   |                      |  |
|  |  |  |    (/25 each in a,b,c)      |  |     |  RDS Security Group:              |                      |  |
|  |  |  |                             |  |     |  - Allow MySQL from EC2 SG only   |                      |  |
|  |  |  +-----------------------------+  |     |                                   |                      |  |
|  |  |                                   |     +-----------------------------------+                      |  |
|  |  +-----------------------------------+                                                                |  |
|  |                                                                                                       |  |
|  +-------------------------------------------------------------------------------------------------------+  |
|                                                                                                             |
|  +-------------------------------------------------------------------------------------------------------+  |
|  |                                                                                                       |  |
|  |  IAM Configuration                                                                                    |  |
|  |  - Admin User Group                                                                                   |  |
|  |  - MFA-Protected Console Access                                                                       |  |
|  |  - Email Alerts for Suspicious Activity                                                               |  |
|  |                                                                                                       |  |
|  +-------------------------------------------------------------------------------------------------------+  |
|                                                                                                             |
|  +-------------------------------------------------------------------------------------------------------+  |
|  |                                                                                                       |  |
|  |  Monitoring & Alerting                                                                                |  |
|  |  - Billing Alerts (Budget set to zero)                                                                |  |
|  |  - Login Activity Monitoring                                                                          |  |
|  |                                                                                                       |  |
|  +-------------------------------------------------------------------------------------------------------+  |
|                                                                                                             |
+-------------------------------------------------------------------------------------------------------------+

flowchart TB
  subgraph AWS["AWS Region: eu-central-1 (Frankfurt)"]
    direction TB
    subgraph VPC["VPC (/18 CIDR)"]
      direction LR
     %% Public Subnet and Route Table side by side
      subgraph PUB["Public Subnet (/22)"]
        EC2["EC2 Instance
        • Nginx
        • Node.js Application
        • UFW Firewall"]
      end
      subgraph RT["Route Table"]
        RTDesc["• Route to Internet Gateway
        • Route to VPC local"]
      end
      %% RDS components below
      subgraph RDSGROUP["RDS Subnet Group"]
        RDS["RDS MySQL Instance
        • Daily Backups
        • Multi-AZ Subnets (/25 each in a, b, c)"]
      end
      subgraph SGROUPS["Security Groups"]
        SG_EC2["EC2 SG:
        • Allow HTTP/HTTPS from anywhere
        • Allow SSH from admin IP only
        • Allow outbound to internet
        • Allow MySQL to RDS SG"]
        SG_RDS["RDS SG:
        • Allow MySQL from EC2 SG only"]
      end
      %% Layout arrows
      EC2 -->|“uses”| RTDesc
      EC2 -->|“connects on 3306”| RDS
      EC2 ---> SG_EC2
      SG_EC2 -->|“MySQL”| SG_RDS
    end

    %% Bottom row: IAM and Monitoring
    subgraph IAM["IAM Configuration"]
      IAMDesc["• Admin User Group
      • MFA-Protected Console Access
      • Email Alerts for Suspicious Activity"]
    end

    subgraph MON["Monitoring & Alerting"]
      MONDesc["• Billing Alerts (Budget set to zero)
      • Login Activity Monitoring"]
    end
  end
