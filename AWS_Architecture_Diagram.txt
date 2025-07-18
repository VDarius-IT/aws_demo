+-------------------------------------------------------------------------------------------------------------+
|                                                                                                             |
|  AWS Region: eu-central-1 (Frankfurt)                                                                       |
|                                                                                                             |
|  +-------------------------------------------------------------------------------------------------------+  |
|  |                                                                                                       |  |
|  |  VPC (/18 CIDR)                                                                                       |  |
|  |                                                                                                       |  |
|  |  +-----------------------------------+                           +-----------------------------------+  |
|  |  |                                   |                           |                                   |  |
|  |  |  Public Subnet (/22)              |                           |  Route Table                      |  |
|  |  |                                   |                           |                                   |  |
|  |  |  +-----------------------------+  |                           |  - Route to Internet Gateway      |  |
|  |  |  |                             |  |                           |  - Route to VPC local             |  |
|  |  |  |  EC2 Instance               |  |                           |                                   |  |
|  |  |  |                             |  |                           +-----------------------------------+  |
|  |  |  |  - Nginx                    |  |                                                                |
|  |  |  |  - Node.js Application      |  |                                                                |
|  |  |  |  - UFW Firewall             |  |                           +-----------------------------------+  |
|  |  |  |                             |  |                           |                                   |  |
|  |  |  +-----------------------------+  |                           |  Security Groups                  |  |
|  |  |                                   |                           |                                   |  |
|  |  +-----------------------------------+                           |  EC2 Security Group:              |  |
|  |               |                                                  |  - Allow HTTP/HTTPS from anywhere |  |
|  |  Security Group: EC2 → RDS                                       |  - Allow SSH from admin IP only   |  |
|  |  (MySQL 3306 only)                                               |  - Allow outbound to internet     |  |
|  |               ↓                                                  |  - Allow MySQL to RDS SG          |  |
|  |  +-----------------------------------+                           |                                   |  |
|  |  |                                   |                           |  RDS Security Group:              |  |
|  |  |  RDS Subnet Group                 |                           |  - Allow MySQL from EC2 SG only   |  |
|  |  |                                   |                           |                                   |  |
|  |  |  +-----------------------------+  |                           +-----------------------------------+  |
|  |  |  |                             |  |                                                                |
|  |  |  |  RDS MySQL Instance         |  |                                                                |
|  |  |  |                             |  |                                                                |
|  |  |  |  - Daily Backups            |  |                                                                |
|  |  |  |  - Multi-AZ Subnets         |  |                                                                |
|  |  |  |    (/25 each in a,b,c)      |  |                                                                |
|  |  |  |                             |  |                                                                |
|  |  |  +-----------------------------+  |                                                                |
|  |  |                                   |                                                                |
|  |  +-----------------------------------+                                                                |
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
            direction TB
            
            subgraph PUB["Public Subnet (/22)"]
                EC2["EC2 Instance<br/>• Nginx<br/>• Node.js Application<br/>• UFW Firewall"]
            end

            subgraph RT["Route Table"]
                RTDesc["• Route to Internet Gateway<br/>• Route to VPC local"]
            end
            
            subgraph RDSGROUP["RDS Subnet Group (Private)"]
                RDS["RDS MySQL Instance<br/>• Daily Backups<br/>• Multi-AZ"]
            end

            subgraph SGROUPS["Security Groups"]
                SG_EC2["<b>EC2 SG</b><br/>• Allow HTTP/HTTPS from anywhere<br/>• Allow SSH from admin IP<br/>• Allow outbound to internet"]
                SG_RDS["<b>RDS SG</b><br/>• Allow MySQL 3306 from EC2 SG"]
            end

            %% Define relationships
            EC2 -- "uses subnet with" --> RT
            EC2 -- "connects via port 3306" --> RDS
            EC2 -- "is protected by" --> SG_EC2
            RDS -- "is protected by" --> SG_RDS
            SG_EC2 -- "allows traffic to" --> SG_RDS
        end

        subgraph IAM["IAM Configuration"]
          IAMDesc["• Admin User Group<br/>• MFA-Protected Console Access<br/>• Email Alerts for Suspicious Activity"]
        end

        subgraph MON["Monitoring & Alerting"]
          MONDesc["• Billing Alerts (Budget set)<br/>• CloudTrail Login Monitoring"]
        end
    end
