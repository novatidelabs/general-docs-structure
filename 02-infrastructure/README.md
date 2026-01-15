# 02 — Infrastructure Documentation

**Audience**: DevOps, SRE, Platform Engineers, Cloud Architects  
**Purpose**: Complete technical reference for Penelope's physical infrastructure - cloud resources, networking, compute, data layers, security, and operational tooling.

**Last Updated**: 2026-01-15  
**Status**: ✅ Active Production Infrastructure

---

## 📋 Table of Contents

1. [Cloud Platform](#01-cloud-platform) - AWS setup, accounts, networking, VPC
2. [Compute](#02-compute) - ECS, Fargate, container orchestration
3. [Data Infrastructure](#03-data-infrastructure) - RDS, S3, Redis, caching
4. [Security Infrastructure](#04-security-infrastructure) - IAM, secrets, network security
5. [Observability](#05-observability-infrastructure) - Logging, metrics, alerting
6. [CI/CD Infrastructure](#06-ci-cd-infrastructure) - GitHub Actions, GHCR, pipelines
7. [Infrastructure as Code](#07-iac) - Terraform structure, modules, state management
8. [Disaster Recovery](#08-disaster-recovery) - Backups, failover, business continuity

---

## 🎯 Reading Order

### For New Team Members
1. Start with [01-cloud-platform/provider-strategy.md](./01-cloud-platform/provider-strategy.md) - understand our cloud approach
2. Read [01-cloud-platform/networking.md](./01-cloud-platform/networking.md) - learn the network topology
3. Review [02-compute/orchestration.md](./02-compute/orchestration.md) - understand how services run
4. Check [07-iac/terraform-structure.md](./07-iac/terraform-structure.md) - learn how to make changes

### For Operations/SRE
1. [05-observability-infrastructure/](./05-observability-infrastructure/) - monitoring and alerting
2. [08-disaster-recovery/](./08-disaster-recovery/) - backup and recovery procedures
3. [04-security-infrastructure/](./04-security-infrastructure/) - security controls

### For Developers
1. [02-compute/orchestration.md](./02-compute/orchestration.md) - how your code runs
2. [03-data-infrastructure/](./03-data-infrastructure/) - databases and storage
3. [06-ci-cd-infrastructure/](./06-ci-cd-infrastructure/) - deployment pipelines

---

## 🏗️ Infrastructure Overview

### Current Stack
- **Cloud Provider**: AWS (us-east-1)
- **Compute**: ECS Fargate (serverless containers)
- **Database**: RDS PostgreSQL 17
- **Storage**: S3 (raw data, processed data)
- **Orchestration**: EventBridge (event-driven workflows)
- **Networking**: VPC with public/private subnets, Multi-AZ
- **IaC**: Terraform (modular structure)
- **CI/CD**: GitHub Actions + GHCR

### Environments
- **dev**: Development environment (us-east-1)
- **shared-services**: VPC, networking (us-east-1)

---

## 📂 Documentation Structure

```
02-infrastructure/
├── README.md (this file)
├── PENDING.md (features not yet implemented)
│
├── 01-cloud-platform/
│   ├── provider-strategy.md       # AWS strategy, region selection
│   ├── account-structure.md       # AWS account hierarchy
│   ├── networking.md              # VPC, subnets, CIDR, security groups
│   └── cost-management.md         # Cost allocation, tagging
│
├── 02-compute/
│   ├── orchestration.md           # ECS cluster, Fargate configuration
│   ├── container-registry.md      # GHCR, image management
│   ├── compute-resources.md       # Task sizing, CPU/memory allocation
│   └── serverless.md              # Lambda functions (Slack notifications)
│
├── 03-data-infrastructure/
│   ├── databases.md               # RDS PostgreSQL configuration
│   ├── caching.md                 # Redis (in-container, ElastiCache future)
│   ├── message-queues.md          # EventBridge event bus
│   └── object-storage.md          # S3 buckets, lifecycle policies
│
├── 04-security-infrastructure/
│   ├── iam-roles.md               # ECS execution/task roles, policies
│   ├── secrets-management.md      # Secrets Manager, SSM Parameter Store
│   ├── network-security.md        # Security groups, NACLs, ingress/egress
│   └── encryption.md              # Encryption at rest/transit, KMS
│
├── 05-observability-infrastructure/
│   ├── logging-stack.md           # CloudWatch Logs, log groups
│   ├── metrics-stack.md           # CloudWatch metrics (future)
│   ├── alerting-stack.md          # Lambda → Slack notifications
│   └── dashboards.md              # CloudWatch dashboards (future)
│
├── 06-ci-cd-infrastructure/
│   ├── pipeline-tooling.md        # GitHub Actions workflows
│   ├── artifact-storage.md        # GHCR (GitHub Container Registry)
│   └── deployment-automation.md    # Manual terraform apply (ArgoCD future)
│
├── 07-iac/
│   ├── terraform-structure.md     # Workspace layout, module organization
│   ├── modules.md                 # Reusable modules (future)
│   ├── provisioning-guide.md      # How to provision new environments
│   └── drift-detection.md         # Drift detection strategy (future)
│
└── 08-disaster-recovery/
    ├── backup-strategy.md         # RDS snapshots, S3 versioning
    ├── failover-procedures.md     # Multi-region failover (future)
    └── business-continuity.md     # DR plans, RTO/RPO targets
```

---

## 🔄 Change Management

### Making Infrastructure Changes

1. **Review Documentation**: Read relevant sections before making changes
2. **Update Terraform**: Make changes in `environments/dev/services/` or `environments/shared-services/`
3. **Plan First**: Always run `terraform plan` before `apply`
4. **Apply Changes**: Use `AWS_PROFILE=penelope terraform apply`
5. **Update Docs**: Update this documentation to reflect changes
6. **Commit**: Commit both code and documentation changes together

### Documentation Updates

- **When to Update**: Any infrastructure change must update relevant docs
- **Review Process**: DevOps team reviews documentation PRs
- **Versioning**: Documentation is versioned with infrastructure code
- **Accuracy**: Documentation must match actual deployed infrastructure

---

## 🚀 Quick Links

### Common Operations
- [Provision New Environment](./07-iac/provisioning-guide.md)
- [Deploy New Microservice](./02-compute/orchestration.md#adding-new-tasks)
- [Update Secrets](./04-security-infrastructure/secrets-management.md#updating-secrets)
- [View Logs](./05-observability-infrastructure/logging-stack.md#accessing-logs)
- [Disaster Recovery](./08-disaster-recovery/backup-strategy.md)

### Terraform Commands
```bash
# Navigate to service directory
cd environments/dev/services/ecs

# Set AWS profile
export AWS_PROFILE=penelope

# Plan changes
terraform plan

# Apply changes
terraform apply

# View outputs
terraform output
```

---

## 📞 Support & Contacts

- **DevOps Team**: Primary infrastructure maintainers
- **On-Call**: See PagerDuty rotation (future)
- **Slack**: `#penelope-infrastructure` (if exists)
- **Documentation Issues**: Open GitHub issue with label `documentation`

---

## 📝 Document Conventions

- ✅ **Implemented**: Feature is live in production
- 🚧 **In Progress**: Feature is being implemented
- 📋 **Planned**: Feature is planned but not started
- ❌ **Not Applicable**: Feature not needed for our use case

---

## 🔗 Related Documentation

- [01-system-architecture/](../01-system-architecture/) - Logical architecture patterns
- [03-runbooks/](../03-runbooks/) - Operational procedures
- [Terraform Code](../../environments/) - Actual infrastructure code

---

**Note**: This documentation reflects the current state of Penelope infrastructure as of 2026-01-15. For planned features not yet implemented, see [PENDING.md](./PENDING.md).
