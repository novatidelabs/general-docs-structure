# Networking Architecture

**Status**: ✅ Implemented  
**Last Updated**: 2026-01-15

---

## Overview

Penelope uses a **Multi-AZ VPC** in us-east-1 with public and private subnets, designed for high availability and security isolation.

---

## VPC Configuration

### Main VPC

| Attribute | Value |
|-----------|-------|
| **VPC ID** | `vpc-XXXXX` (penelope-main-vpc) |
| **CIDR Block** | `10.0.0.0/16` |
| **Region** | us-east-1 |
| **Availability Zones** | us-east-1a, us-east-1b |
| **DNS Hostnames** | Enabled |
| **DNS Support** | Enabled |

**Terraform**: `environments/shared-services/vpc/main.tf`

---

## Subnet Design

### Public Subnets (ECS Tasks)

| Subnet | CIDR | AZ | Purpose |
|--------|------|----|---------| 
| **public-subnet-1** | `10.0.1.0/24` | us-east-1a | ECS Fargate tasks |
| **public-subnet-2** | `10.0.2.0/24` | us-east-1b | ECS Fargate tasks (HA) |

**Configuration**:
- ✅ Auto-assign public IP: Enabled
- ✅ Internet Gateway attached
- ✅ Route to 0.0.0.0/0 via IGW

**Use Cases**:
- ECS Fargate tasks (news-extractor, news-classifier, etc.)
- Tasks need internet access for:
  - Pulling Docker images from GHCR
  - API calls (OpenAI, external services)
  - S3 access (via public endpoint)

### Private Subnets (RDS)

| Subnet | CIDR | AZ | Purpose |
|--------|------|----|---------| 
| **private-subnet-1** | `10.0.10.0/24` | us-east-1a | RDS primary |
| **private-subnet-2** | `10.0.11.0/24` | us-east-1b | RDS standby (Multi-AZ) |

**Configuration**:
- ❌ No public IP assignment
- ❌ No direct internet access
- ✅ Isolated from public internet
- ✅ Accessible only from VPC

**Use Cases**:
- RDS PostgreSQL database
- Future: ElastiCache Redis

---

## Internet Connectivity

### Internet Gateway (IGW)

| Attribute | Value |
|-----------|-------|
| **IGW ID** | `igw-XXXXX` |
| **Attached to** | penelope-main-vpc |
| **Purpose** | Public subnet internet access |

**Route Table (Public)**:
```
Destination     Target
10.0.0.0/16     local
0.0.0.0/0       igw-XXXXX
```

### NAT Gateway

**Status**: ❌ Not Implemented

**Rationale**: 
- ECS tasks use public subnets (no need for NAT)
- RDS in private subnet doesn't need internet access
- Cost optimization (~$32/month per NAT Gateway)

**Future**: If private ECS tasks needed, deploy NAT Gateway in public subnet.

---

## Security Groups

### ECS Security Group

**Name**: `penelope-ecs-sg`  
**ID**: `sg-00cd762383ae2f0b6`

**Inbound Rules**:
```
Type        Protocol    Port Range    Source          Description
All Traffic    All         All        sg-00cd762383ae2f0b6   Self-referencing (tasks communicate)
```

**Outbound Rules**:
```
Type        Protocol    Port Range    Destination    Description
All Traffic    All         All        0.0.0.0/0      Internet access (APIs, S3, GHCR)
```

**Attached to**:
- All ECS Fargate tasks
- SearXNG service

---

### RDS Security Group

**Name**: `penelope-rds-sg`  
**ID**: `sg-0b7a36a67c8714406`

**Inbound Rules**:
```
Type        Protocol    Port    Source                  Description
PostgreSQL     TCP       5432    sg-00cd762383ae2f0b6    From ECS tasks only
PostgreSQL     TCP       5432    sg-0b7a36a67c8714406    Self (Multi-AZ replication)
```

**Outbound Rules**:
```
Type        Protocol    Port Range    Destination    Description
All Traffic    All         All        0.0.0.0/0      RDS managed service communication
```

**Attached to**:
- RDS PostgreSQL instance

**Security**: RDS only accessible from ECS tasks, not from internet.

---

## DNS Configuration

### Route 53

**Status**: ❌ Not Implemented

**Current**: Services use internal IPs or AWS service endpoints
- RDS: `db-penelope-dev.c6p8ywiq6v9s.us-east-1.rds.amazonaws.com`
- SearXNG: Private IP `10.0.X.X` (ECS service discovery)

**Future**: 
- 📋 Private hosted zone for internal services
- 📋 Public domain for API endpoints (if needed)

---

## Network Flow Diagrams

### Data Flow: ECS Task → RDS

```
┌─────────────────┐
│  ECS Task       │
│  (Public Subnet)│
│  10.0.1.x       │
└────────┬────────┘
         │
         │ PostgreSQL (5432)
         │ via sg-00cd762383ae2f0b6
         ↓
┌─────────────────┐
│  RDS PostgreSQL │
│ (Private Subnet)│
│  10.0.10.x      │
└─────────────────┘
```

### Data Flow: ECS Task → Internet

```
┌─────────────────┐
│  ECS Task       │
│  (Public Subnet)│
│  10.0.1.x       │
└────────┬────────┘
         │
         │ HTTPS (443)
         │ via IGW
         ↓
┌─────────────────┐
│  Internet       │
│  (OpenAI, GHCR) │
└─────────────────┘
```

### Data Flow: ECS Task → S3

```
┌─────────────────┐
│  ECS Task       │
│  (Public Subnet)│
│  10.0.1.x       │
└────────┬────────┘
         │
         │ HTTPS (443)
         │ via S3 public endpoint
         ↓
┌─────────────────┐
│  S3 Bucket      │
│  (AWS Service)  │
└─────────────────┘
```

**Note**: S3 access via public endpoint (no VPC endpoint yet).

---

## Network Performance

### Bandwidth
- **ECS Fargate**: Up to 10 Gbps (task-dependent)
- **RDS**: Up to 10 Gbps (instance-dependent)
- **S3**: Unlimited (AWS managed)

### Latency
- **ECS ↔ RDS**: <1ms (same VPC)
- **ECS ↔ S3**: ~5-10ms (us-east-1)
- **ECS ↔ Internet**: Variable (depends on destination)

---

## IP Address Management

### CIDR Allocation

| Block | Usage | Available IPs |
|-------|-------|---------------|
| `10.0.0.0/16` | VPC | 65,536 |
| `10.0.1.0/24` | Public Subnet 1 | 251 |
| `10.0.2.0/24` | Public Subnet 2 | 251 |
| `10.0.10.0/24` | Private Subnet 1 | 251 |
| `10.0.11.0/24` | Private Subnet 2 | 251 |
| `10.0.3.0/24` - `10.0.9.0/24` | **Reserved** | Future expansion |
| `10.0.12.0/24` - `10.0.255.0/24` | **Reserved** | Future expansion |

**Expansion Capacity**: 60,000+ IPs available for future subnets.

---

## VPC Endpoints

**Status**: ❌ Not Implemented

**Future Optimization**:
- 📋 **S3 Gateway Endpoint**: Free, reduce data transfer costs
- 📋 **ECR Interface Endpoint**: Faster image pulls, private access
- 📋 **Secrets Manager Endpoint**: Private secret access

**Cost-Benefit**: Not critical at current scale, evaluate when traffic increases.

---

## Network Security Best Practices

### Implemented ✅
- ✅ **Least Privilege**: Security groups allow only required traffic
- ✅ **Private Subnets**: RDS isolated from internet
- ✅ **Multi-AZ**: High availability for critical services
- ✅ **Encryption in Transit**: TLS for all external communication

### Planned 📋
- 📋 **VPC Flow Logs**: Network traffic logging
- 📋 **Network ACLs**: Additional subnet-level firewall
- 📋 **WAF**: Web Application Firewall (if public API added)
- 📋 **VPC Peering**: Connect to other VPCs (if multi-account)

---

## Troubleshooting

### Common Issues

**ECS Task Can't Reach Internet**:
- ✅ Check: Task in public subnet?
- ✅ Check: Public IP assigned?
- ✅ Check: Security group allows outbound traffic?
- ✅ Check: Route table has IGW route?

**ECS Task Can't Reach RDS**:
- ✅ Check: RDS security group allows traffic from ECS SG?
- ✅ Check: RDS in same VPC?
- ✅ Check: Connection string correct?

**High Data Transfer Costs**:
- 📋 Consider: VPC endpoints for S3/ECR
- 📋 Consider: Move tasks to private subnets with NAT

---

## Terraform Resources

### Key Files
- `environments/shared-services/vpc/main.tf` - VPC, subnets, IGW
- `environments/shared-services/vpc/outputs.tf` - VPC/subnet IDs
- `environments/shared-services/vpc/variables.tf` - CIDR blocks, AZs

### Outputs
```bash
cd environments/shared-services/vpc
terraform output vpc_id
terraform output public_subnet_ids
terraform output private_subnet_ids
terraform output ecs_security_group_id
terraform output rds_security_group_id
```

---

## Related Documentation

- [provider-strategy.md](./provider-strategy.md) - AWS region strategy
- [account-structure.md](./account-structure.md) - AWS account setup
- [../04-security-infrastructure/network-security.md](../04-security-infrastructure/network-security.md) - Security groups details

---

**Next Steps**:
1. 📋 Implement VPC Flow Logs for network monitoring
2. 📋 Evaluate S3 Gateway Endpoint for cost savings
3. 📋 Plan private subnets for future ECS tasks (with NAT)
