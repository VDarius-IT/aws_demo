# AWS Security Group Configuration

## EC2 Security Group

### Inbound Rules

| Type | Protocol | Port Range | Source | Description |
|------|----------|------------|--------|-------------|
| HTTP | TCP | 80 | 0.0.0.0/0 | Allow HTTP from anywhere |
| HTTPS | TCP | 443 | 0.0.0.0/0 | Allow HTTPS from anywhere |
| SSH | TCP | 22 | Admin IP/32 | Allow SSH from administrator IP only |

### Outbound Rules

| Type | Protocol | Port Range | Destination | Description |
|------|----------|------------|-------------|-------------|
| All traffic | All | All | 0.0.0.0/0 | Allow all outbound internet traffic |
| MySQL | TCP | 3306 | RDS Security Group | Allow MySQL connections to RDS |

## RDS Security Group

### Inbound Rules

| Type | Protocol | Port Range | Source | Description |
|------|----------|------------|--------|-------------|
| MySQL | TCP | 3306 | EC2 Security Group | Allow MySQL from EC2 instance only |

### Outbound Rules

| Type | Protocol | Port Range | Destination | Description |
|------|----------|------------|-------------|-------------|
| All traffic | All | All | 0.0.0.0/0 | Allow all outbound traffic |

## Network ACLs

### Inbound Rules

| Rule # | Type | Protocol | Port Range | Source | Allow/Deny |
|--------|------|----------|------------|--------|------------|
| 100 | All traffic | All | All | 0.0.0.0/0 | ALLOW |
| * | All traffic | All | All | 0.0.0.0/0 | DENY |

### Outbound Rules

| Rule # | Type | Protocol | Port Range | Destination | Allow/Deny |
|--------|------|----------|------------|-------------|------------|
| 100 | All traffic | All | All | 0.0.0.0/0 | ALLOW |
| * | All traffic | All | All | 0.0.0.0/0 | DENY |
