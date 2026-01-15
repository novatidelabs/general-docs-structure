# Cloud Provider Strategy

**Status**: ✅ Implemented  
**Last Updated**: 2026-01-15

---

## Overview

Penelope infrastructure runs entirely on **Amazon Web Services (AWS)**, leveraging managed services to minimize operational overhead and maximize reliability.

---

## Cloud Provider Selection

### Primary Provider: AWS

**Rationale**:
- ✅ **Mature ecosystem**: Comprehensive managed services (ECS, RDS, S3, EventBridge)
- ✅ **Serverless-first**: Fargate, Lambda, EventBridge eliminate server management
- ✅ **Cost-effective**: Pay-per-use model for batch workloads
- ✅ **Regional presence**: us-east-1 provides low latency for target users
- ✅ **Team expertise**: DevOps team has AWS experience

**Alternatives Considered**:
- **GCP**: Excellent for ML workloads, but AWS ECS Fargate better fits our event-driven architecture
- **Azure**: Strong enterprise integration, but less mature serverless container orchestration
- **Multi-cloud**: Rejected due to complexity overhead for current scale

---

## Multi-Cloud Strategy

**Current Approach**: **Single Cloud (AWS)**

**Rationale**:
- Current scale doesn't justify multi-cloud complexity
- Event-driven architecture is cloud-agnostic at application level
- Can migrate to multi-cloud if needed (containers + IaC make it feasible)

**Future Considerations**:
- 📋 **Planned**: Evaluate GCP for ML workloads if AI/ML requirements grow
- 📋 **Planned**: Consider multi-cloud for disaster recovery (see [08-disaster-recovery/failover-procedures.md](../08-disaster-recovery/failover-procedures.md))

---

## Region Strategy

### Primary Region: **us-east-1** (N. Virginia)

**Selection Criteria**:
1. **Latency**: Lowest latency to primary user base (North America/Europe)
2. **Service Availability**: All AWS services available (ECS Fargate, EventBridge, RDS)
3. **Cost**: Generally lower costs than newer regions
4. **Maturity**: Most stable AWS region with longest track record

### Region Configuration

| Region | Purpose | Status |
|--------|---------|--------|
| **us-east-1** | Primary production | ✅ Active |
| us-west-2 | DR failover | 📋 Planned |
| eu-west-1 | EU expansion | 📋 Future |

---

## Availability Zones

**Strategy**: **Multi-AZ for critical services**

### Current Setup
- **VPC**: Spans 2 AZs (us-east-1a, us-east-1b)
- **Public Subnets**: 2 AZs for ECS tasks
- **Private Subnets**: 2 AZs for RDS (high availability)
- **RDS**: Single-AZ (dev), Multi-AZ planned for production

**Benefits**:
- ✅ High availability for database layer
- ✅ ECS tasks can run in either AZ
- ✅ Automatic failover for RDS (when Multi-AZ enabled)

---

## Service Selection Philosophy

### Preference Order
1. **Managed Services First**: RDS over self-hosted PostgreSQL
2. **Serverless When Possible**: Fargate over EC2, Lambda over containers
3. **Event-Driven**: EventBridge over polling/cron
4. **Pay-per-use**: Avoid always-on infrastructure

### AWS Services Used

| Service | Purpose | Rationale |
|---------|---------|-----------|
| **ECS Fargate** | Container orchestration | Serverless, no server management |
| **RDS PostgreSQL** | Relational database | Managed backups, patching, HA |
| **S3** | Object storage | Durable, scalable, cost-effective |
| **EventBridge** | Event bus | Native event-driven orchestration |
| **Lambda** | Slack notifications | Serverless, pay-per-invocation |
| **Secrets Manager** | RDS credentials | Automatic rotation, secure |
| **SSM Parameter Store** | App secrets | Free tier, simple key-value |
| **CloudWatch** | Logging, monitoring | Native AWS integration |
| **VPC** | Networking | Secure, isolated network |

---

## Vendor Lock-in Mitigation

### Strategy
While we use AWS-specific services, we mitigate lock-in through:

1. **Containerization**: All workloads run in Docker containers (portable)
2. **Infrastructure as Code**: Terraform (cloud-agnostic, can target other providers)
3. **Standard Protocols**: PostgreSQL (portable), S3-compatible APIs
4. **Event-Driven Architecture**: Application logic is cloud-agnostic

### Portability Assessment

| Component | Portability | Migration Effort |
|-----------|-------------|------------------|
| **Containers** | ✅ High | Low (Docker standard) |
| **PostgreSQL** | ✅ High | Low (standard SQL) |
| **S3 Storage** | 🟡 Medium | Medium (S3 API widely supported) |
| **EventBridge** | ❌ Low | High (AWS-specific, needs redesign) |
| **Lambda** | 🟡 Medium | Medium (can use Cloud Functions/Azure Functions) |
| **ECS Fargate** | 🟡 Medium | Medium (migrate to GKE/AKS) |

---

## Cost Optimization Strategy

### Principles
1. **Right-sizing**: Match resources to actual workload needs
2. **Serverless**: Pay only when tasks run (Fargate, Lambda)
3. **Spot Instances**: Not used (Fargate Spot future consideration)
4. **Reserved Capacity**: Not used (workloads are batch/event-driven)
5. **S3 Lifecycle**: Transition old data to cheaper storage classes

### Current Cost Profile
- **ECS Fargate**: $0.04/hour per task (only when running)
- **RDS**: ~$50/month (db.t4g.small, single-AZ)
- **S3**: ~$5/month (100GB storage)
- **EventBridge**: ~$1/month (1M events)
- **Lambda**: <$1/month (low invocation volume)
- **CloudWatch Logs**: ~$5/month (30-day retention)

**Total Estimated**: ~$70-100/month (dev environment)

---

## Compliance & Governance

### Current State
- ✅ **Tagging Strategy**: All resources tagged with Project, Environment, ManagedBy, Owner
- ✅ **IAM Least Privilege**: ECS tasks have minimal required permissions
- ✅ **Encryption**: RDS encrypted at rest, S3 server-side encryption
- ✅ **Network Isolation**: Private subnets for RDS, public for ECS tasks

### Future Requirements
- 📋 **AWS Organizations**: Multi-account strategy for prod/dev separation
- 📋 **AWS Config**: Track configuration changes
- 📋 **CloudTrail**: Audit API calls
- 📋 **GuardDuty**: Threat detection

---

## Decision Log

| Date | Decision | Rationale |
|------|----------|-----------|
| 2025-11 | Choose AWS over GCP | Team expertise, ECS Fargate maturity |
| 2025-11 | us-east-1 as primary region | Lowest latency, service availability |
| 2025-12 | Fargate over EC2 | Eliminate server management |
| 2025-12 | EventBridge over Airflow | Simpler, serverless, event-driven |
| 2026-01 | Single-AZ RDS (dev) | Cost optimization for non-prod |

---

## Related Documentation

- [account-structure.md](./account-structure.md) - AWS account hierarchy
- [networking.md](./networking.md) - VPC and network design
- [cost-management.md](./cost-management.md) - Cost allocation and budgets
- [../02-compute/orchestration.md](../02-compute/orchestration.md) - ECS Fargate details

---

**Next Steps**:
1. 📋 Evaluate Multi-AZ RDS for production
2. 📋 Implement cost alerts (AWS Budgets)
3. 📋 Plan DR region (us-west-2)
