# Terraform Structure

**Status**: ✅ Implemented  
**Last Updated**: 2026-01-15

---

## Overview

Penelope uses **Terraform** for Infrastructure as Code, organized in a modular structure by environment and service.

---

## Directory Structure

```
terraform/penelope/
├── environments/
│   ├── shared-services/
│   │   └── vpc/                    # VPC, subnets, security groups
│   │       ├── main.tf
│   │       ├── variables.tf
│   │       ├── outputs.tf
│   │       └── terraform.tfvars
│   │
│   └── dev/
│       └── services/
│           ├── ecs/                # ECS cluster, tasks, EventBridge
│           │   ├── main.tf
│           │   ├── task_definitions.tf
│           │   ├── eventbridge_rules.tf
│           │   ├── schedulers.tf
│           │   ├── lambda_notifications.tf
│           │   ├── ssm_parameters.tf
│           │   ├── variables.tf
│           │   ├── outputs.tf
│           │   └── terraform.tfvars
│           │
│           ├── rds/                # RDS PostgreSQL
│           │   ├── main.tf
│           │   ├── variables.tf
│           │   ├── outputs.tf
│           │   └── terraform.tfvars
│           │
│           └── s3/                 # S3 buckets
│               ├── main.tf
│               ├── variables.tf
│               ├── outputs.tf
│               └── terraform.tfvars
│
├── docs/                           # Documentation
└── README.md
```

---

## State Management

**Backend**: Local (file-based)  
**State Files**: `terraform.tfstate` in each module directory

**Future**: 📋 Migrate to S3 backend with DynamoDB locking

---

## Terraform Workflow

```bash
# Navigate to service
cd environments/dev/services/ecs

# Set AWS profile
export AWS_PROFILE=penelope

# Initialize (first time only)
terraform init

# Plan changes
terraform plan

# Apply changes
terraform apply

# View outputs
terraform output
```

---

## Module Organization

### shared-services/vpc
- VPC, subnets, IGW
- Security groups (ECS, RDS)
- Outputs: VPC ID, subnet IDs, SG IDs

### dev/services/ecs
- ECS cluster
- Task definitions (all microservices)
- EventBridge rules & schedulers
- Lambda notification function
- SSM parameters (secrets)
- CloudWatch log groups

### dev/services/rds
- RDS PostgreSQL instance
- DB subnet group
- Outputs: Endpoint, port, secret ARN

### dev/services/s3
- S3 buckets for data storage
- Bucket policies
- Lifecycle rules (future)

---

## Variables Pattern

**variables.tf**: Define all variables
**terraform.tfvars**: Set actual values (not committed if sensitive)

```hcl
# variables.tf
variable "aws_region" {
  description = "AWS region"
  type        = string
}

# terraform.tfvars
aws_region = "us-east-1"
```

---

## Tagging Strategy

All resources tagged via `default_tags`:
```hcl
provider "aws" {
  default_tags {
    tags = {
      Project     = "penelope"
      Environment = "dev"
      ManagedBy   = "terraform"
      Owner       = "devops"
    }
  }
}
```

---

## Best Practices

✅ **Implemented**:
- Separate environments (shared-services, dev)
- Variables for all configurable values
- Outputs for cross-module references
- Consistent naming conventions
- Default tags on all resources

📋 **Planned**:
- Remote state (S3 + DynamoDB)
- Terraform modules for reusable patterns
- Terraform workspaces (dev/prod)
- State locking
- Drift detection

---

## Related Documentation

- [provisioning-guide.md](./provisioning-guide.md) - How to provision new environments
- [modules.md](./modules.md) - Reusable modules (future)
- [drift-detection.md](./drift-detection.md) - Drift detection strategy (future)

---

**Next Steps**:
1. 📋 Migrate to S3 backend with state locking
2. 📋 Create reusable Terraform modules
3. 📋 Implement automated drift detection
