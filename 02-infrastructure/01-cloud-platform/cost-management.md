# Cost Management

**Status**: ✅ Implemented (Basic Tagging)  
**Last Updated**: 2026-01-15

---

## Overview

Penelope uses resource tagging and serverless architecture to optimize costs. Current monthly spend: **~$70-100** (dev environment).

---

## Cost Breakdown (Monthly)

| Service | Cost | Usage |
|---------|------|-------|
| **RDS PostgreSQL** | ~$50 | db.t4g.small, 20GB, single-AZ |
| **ECS Fargate** | ~$10-20 | Batch tasks (hours/day) |
| **S3** | ~$5 | 100GB storage + requests |
| **CloudWatch Logs** | ~$5 | 30-day retention, 5GB/month |
| **EventBridge** | ~$1 | 1M events/month |
| **Lambda** | <$1 | Low invocation volume |
| **Secrets Manager** | ~$1 | 2 secrets |
| **Data Transfer** | ~$2-5 | Outbound to internet |
| **Total** | **~$74-87** | Dev environment |

---

## Tagging Strategy

### Required Tags

All resources must have:
```hcl
tags = {
  Project     = "penelope"
  Environment = "dev" | "prod" | "shared-services"
  ManagedBy   = "terraform"
  Owner       = "devops"
}
```

### Optional Tags
- `Service`: `ecs`, `rds`, `s3`, `lambda`
- `CostCenter`: (future)
- `Application`: (future)

---

## Cost Optimization

### Implemented ✅

1. **Serverless First**:
   - ECS Fargate: Pay only when tasks run
   - Lambda: Pay per invocation
   - EventBridge: Pay per event

2. **Right-Sizing**:
   - RDS: db.t4g.small (sufficient for dev)
   - ECS Tasks: 512 CPU, 1024 MB (minimal for workload)

3. **Storage Optimization**:
   - S3: Standard class (no Glacier yet)
   - CloudWatch: 30-day retention (not indefinite)

### Planned 📋

1. **S3 Lifecycle Policies**:
   - Transition to Glacier after 90 days
   - Delete after 1 year

2. **RDS Optimization**:
   - Stop RDS when not in use (dev)
   - Use Aurora Serverless for prod (future)

3. **VPC Endpoints**:
   - S3 Gateway Endpoint (free, reduce data transfer)
   - ECR Interface Endpoint (reduce NAT costs if private subnets)

4. **Reserved Capacity**:
   - Not applicable (batch workloads, not always-on)

---

## Budget Alerts

**Status**: 📋 Not Implemented

**Planned**:
```
AWS Budgets:
- Dev: $150/month threshold
- Prod: $500/month threshold (future)
- Alerts: 80%, 100%, 120% of budget
```

---

## Cost Monitoring

### AWS Cost Explorer

**Filters**:
```
Tag: Project = penelope
Tag: Environment = dev
Group by: Service
```

### Monthly Review Process

1. Export cost report from Cost Explorer
2. Review top 5 services by cost
3. Identify anomalies or spikes
4. Optimize if needed

---

## Cost Projections

### Production Environment (Future)

| Service | Dev Cost | Prod Cost (Est.) |
|---------|----------|------------------|
| RDS | $50 | $200 (Multi-AZ, larger instance) |
| ECS Fargate | $15 | $50 (more frequent runs) |
| S3 | $5 | $20 (more data) |
| CloudWatch | $5 | $20 (more logs) |
| EventBridge | $1 | $5 |
| Lambda | $1 | $5 |
| **Total** | **$77** | **$300** |

---

## Related Documentation

- [provider-strategy.md](./provider-strategy.md) - Service selection rationale
- [account-structure.md](./account-structure.md) - Account setup

---

**Next Steps**:
1. 📋 Implement AWS Budgets with alerts
2. 📋 S3 lifecycle policies for old data
3. 📋 Evaluate Aurora Serverless for prod
