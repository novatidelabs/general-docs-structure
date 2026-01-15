# AWS Account Structure

**Status**: ✅ Implemented (Single Account)  
**Last Updated**: 2026-01-15

---

## Overview

Penelope currently operates in a **single AWS account** with environment separation via naming conventions and tags.

---

## Current Structure

### Single Account Model

**Account ID**: `492094933547`  
**Account Name**: `penelope` (AWS CLI profile)  
**Region**: us-east-1

**Environments**:
- `dev`: Development environment
- `shared-services`: Networking, VPC

---

## Environment Separation

### Strategy: Tags + Naming Conventions

All resources tagged with:
```hcl
tags = {
  Project     = "penelope"
  Environment = "dev" | "shared-services"
  ManagedBy   = "terraform"
  Owner       = "devops"
}
```

**Resource Naming**:
- VPC: `penelope-main-vpc`
- ECS Cluster: `cluster-penelope-dev`
- RDS: `db-penelope-dev`
- S3 Buckets: `novatide-penelope-raw-news-v2`

---

## IAM Structure

### Users
- **DevOps Team**: Admin access via AWS CLI profile `penelope`

### Service Roles
- **ECS Execution Role**: `penelope-ecs-execution-dev`
- **ECS Task Role**: `penelope-ecs-task-dev`
- **EventBridge Scheduler Role**: `Amazon_EventBridge_Scheduler_ECS_*`
- **Lambda Execution Role**: `penelope-lambda-notify-dev`

---

## Future: Multi-Account Strategy

### Planned Structure (📋 Future)

```
AWS Organization
├── Management Account (root)
├── Production Account
│   ├── VPC (prod)
│   ├── ECS Cluster (prod)
│   └── RDS (prod, Multi-AZ)
├── Development Account
│   ├── VPC (dev)
│   ├── ECS Cluster (dev)
│   └── RDS (dev, single-AZ)
└── Shared Services Account
    ├── CI/CD (GitHub Actions runners)
    ├── Logging (centralized CloudWatch)
    └── Security (GuardDuty, Config)
```

**Benefits**:
- ✅ Blast radius isolation
- ✅ Separate billing
- ✅ Granular IAM policies
- ✅ Compliance requirements

---

## Access Management

### AWS CLI Profile

```bash
# ~/.aws/config
[profile penelope]
region = us-east-1
output = json

# ~/.aws/credentials
[penelope]
aws_access_key_id = AKIA...
aws_secret_access_key = ...
```

### Terraform Authentication

```bash
export AWS_PROFILE=penelope
terraform plan
```

---

## Cost Allocation

### Tagging Strategy

All resources tagged for cost tracking:
- **Project**: `penelope`
- **Environment**: `dev` | `prod` | `shared-services`
- **Service**: `ecs` | `rds` | `s3` | `lambda`
- **Owner**: `devops`

### Cost Explorer Filters

```
Tag: Project = penelope
Tag: Environment = dev
```

---

## Related Documentation

- [provider-strategy.md](./provider-strategy.md) - Cloud provider selection
- [cost-management.md](./cost-management.md) - Cost allocation details
- [../04-security-infrastructure/iam-roles.md](../04-security-infrastructure/iam-roles.md) - IAM roles details

---

**Next Steps**:
1. 📋 Implement AWS Organizations for multi-account
2. 📋 Separate prod/dev into different accounts
3. 📋 Implement AWS SSO for team access
