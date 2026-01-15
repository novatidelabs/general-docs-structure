# Container Orchestration - ECS Fargate

**Status**: ✅ Implemented  
**Last Updated**: 2026-01-15

---

## Overview

Penelope uses **AWS ECS Fargate** for serverless container orchestration. All microservices run as Fargate tasks, eliminating server management.

---

## ECS Cluster

**Name**: `cluster-penelope-dev`  
**Type**: Fargate  
**Region**: us-east-1  
**Capacity Providers**: FARGATE, FARGATE_SPOT (future)

**Terraform**: `environments/dev/services/ecs/main.tf`

---

## Task Definitions

### Active Tasks

| Task Family | CPU | Memory | Type | Trigger |
|-------------|-----|--------|------|---------|
| **news-extractor** | 512 | 1024 | Batch | EventBridge Scheduler (disabled) |
| **news-classifier** | 512 | 1024 | Batch | S3 Event (disabled) |
| **top-stories** | 512 | 1024 | Batch | EventBridge (disabled) |
| **image-generator** | 512 | 1024 | Batch | EventBridge (disabled) |
| **monday-portfolio** | 512 | 1024 | Batch | EventBridge Scheduler (Mon 10AM London) |
| **mcp-service** | 1024 | 2048 | Batch | Manual |
| **searxng** | 1024 | 2048 | Service | Always running |

**Note**: Most tasks currently disabled for cost optimization.

---

## Task Configuration

### news-extractor

**Purpose**: Extract raw news from sources, upload to S3

**Container**:
- Image: `ghcr.io/novatidelabs/penelope_data_extractor_to_s3:latest`
- Port: None (batch job)
- Credentials: GHCR secret from Secrets Manager

**Environment Variables**:
```
AWS_S3_REGION_NAME=us-east-1
AWS_S3_BUCKET_NAME=novatide-penelope-raw-news-v2
AWS_S3_FOLDER_NAME=raw-news-extracted
LOG_LEVEL=INFO
```

**Secrets** (SSM Parameter Store):
- None (uses IAM role for S3 access)

**Schedule**: ❌ Disabled (was: daily 10AM London)

**Network**:
- Subnets: Public (2 AZs)
- Security Group: `sg-00cd762383ae2f0b6`
- Public IP: Yes

---

### news-classifier

**Purpose**: Classify news articles using OpenAI

**Container**:
- Image: `ghcr.io/novatidelabs/penelope-webpages-news-classifier:latest`

**Environment Variables**:
```
AWS_S3_REGION_NAME=us-east-1
AWS_S3_BUCKET_NAME=novatide-penelope-raw-news-v2
AWS_S3_FOLDER_NAME=raw-news-extracted
LOG_LEVEL=INFO
```

**Secrets** (SSM):
- `OPENAI_API_KEY`
- `LANGSMITH_API_KEY`
- `LANGCHAIN_TRACING_V2`
- `LANGCHAIN_PROJECT`

**Trigger**: ❌ Disabled (was: S3 event on new file)

---

### monday-portfolio

**Purpose**: Update Monday.com portfolio with market data

**Container**:
- Image: `ghcr.io/novatidelabs/penelope-monday-portfolio:latest`

**Environment Variables**:
```
AWS_S3_REGION_NAME=us-east-1
AWS_S3_BUCKET_NAME=novatide-penelope-portfolio
AWS_S3_FOLDER_NAME=portfolio-data
LOG_LEVEL=INFO
```

**Secrets** (SSM):
- `SLACK_CHANNEL` (C0A0ZFT6C68)
- `SLACK_BOT_TOKEN`
- `COINGECKO_API_KEY`
- `MONDAY_API_KEY_NOVATIDE`
- `REDIS_HOST`, `REDIS_PORT`, `REDIS_DB`, `REDIS_PASSWORD`

**Schedule**: ✅ Active - Every Monday 10:00 AM (Europe/London)

---

### mcp-service

**Purpose**: MCP (Model Context Protocol) service for AI agents

**Container**:
- Image: `ghcr.io/novatidelabs/penelope-mcp-service:latest`
- CPU: 1024, Memory: 2048

**Secrets** (SSM):
- `OPENAI_API_KEY`
- `GITHUB_PAT`
- `COINGECKO_PRO_API_KEY`
- `SMITHERY_API_KEY`
- `SEARCH_ENGINE_URL` (SearXNG)
- `SLACK_BOT_TOKEN`
- `SLACK_DEFAULT_CHANNEL_ID`
- `MONDAY_API_KEY`
- `FRED_API_KEY`

**Trigger**: Manual execution

**Note**: Failed due to OpenAI quota exceeded.

---

### searxng (Service)

**Purpose**: Self-hosted search engine for mcp-service

**Type**: **ECS Service** (always running, not batch)

**Containers**: 2 (multi-container task)
1. **Redis** (sidecar):
   - Image: `redis:7-alpine`
   - Port: 6379
   - Memory: 512 MB

2. **SearXNG**:
   - Image: `searxng/searxng:latest`
   - Port: 8080
   - Memory: 1536 MB

**Service Configuration**:
- Desired Count: 1
- Launch Type: Fargate
- Network Mode: awsvpc
- Service Discovery: Private IP (10.0.X.X)

**Environment Variables** (SearXNG):
- `SEARXNG_BASE_URL`
- `SEARXNG_SECRET_KEY`

**Health Check**: Container-level (future: ALB health check)

**Access**: Private IP within VPC (mcp-service connects via `http://10.0.X.X:8080`)

---

## IAM Roles

### ECS Execution Role

**Name**: `penelope-ecs-execution-dev`  
**Purpose**: Pull images, write logs, read secrets

**Permissions**:
- `ecr:GetAuthorizationToken`
- `ecr:BatchCheckLayerAvailability`
- `ecr:GetDownloadUrlForLayer`
- `ecr:BatchGetImage`
- `logs:CreateLogStream`
- `logs:PutLogEvents`
- `secretsmanager:GetSecretValue` (GHCR credentials)
- `ssm:GetParameters` (all SSM secrets)

### ECS Task Role

**Name**: `penelope-ecs-task-dev`  
**Purpose**: Application permissions (S3, etc.)

**Permissions**:
- `s3:PutObject`, `s3:GetObject`, `s3:ListBucket` (S3 buckets)
- `s3:PutObjectAcl`

---

## Networking

**VPC**: `penelope-main-vpc`  
**Subnets**: Public (us-east-1a, us-east-1b)  
**Security Group**: `sg-00cd762383ae2f0b6`  
**Public IP**: Yes (required for internet access)

**Connectivity**:
- ✅ Internet (APIs, GHCR)
- ✅ S3 (via public endpoint)
- ✅ RDS (via private IP)
- ✅ SearXNG (via private IP)

---

## Logging

**CloudWatch Log Groups**:
- `/ecs/news-extractor-dev`
- `/ecs/news-classifier-dev`
- `/ecs/monday-portfolio-dev`
- `/ecs/mcp-service`
- `/ecs/searxng`
- `/ecs/searxng-redis`
- `/ecs/top-stories-dev`
- `/ecs/image-generator-dev`

**Retention**: 30 days  
**Log Driver**: awslogs

**Access Logs**:
```bash
aws logs tail /ecs/monday-portfolio-dev --follow --profile penelope
```

---

## Orchestration: EventBridge

### Workflow (Currently Disabled)

```
EventBridge Scheduler (cron)
    ↓
news-extractor (ECS Task)
    ↓ (uploads to S3)
S3 Event → EventBridge Rule
    ↓
news-classifier (ECS Task)
    ↓ (success)
EventBridge Rule
    ↓
top-stories (ECS Task)
    ↓ (success)
EventBridge Rule
    ↓
image-generator (ECS Task)
```

**Current State**: All triggers disabled for cost optimization.

### Active Workflows

**monday-portfolio**:
```
EventBridge Scheduler (Mon 10AM London)
    ↓
monday-portfolio (ECS Task)
    ↓ (success/failure)
Lambda → Slack notification
```

---

## Adding New Tasks

### Step 1: Create Task Definition

```hcl
# environments/dev/services/ecs/task_definitions.tf

resource "aws_ecs_task_definition" "new_task" {
  family                   = "new-task"
  cpu                      = "512"
  memory                   = "1024"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  
  execution_role_arn = aws_iam_role.execution_role.arn
  task_role_arn      = aws_iam_role.task_role.arn
  
  container_definitions = jsonencode([{
    name  = "new-task"
    image = "ghcr.io/org/repo:latest"
    environment = [
      { name = "ENV_VAR", value = "value" }
    ]
    secrets = [
      { name = "SECRET", valueFrom = aws_ssm_parameter.secret.arn }
    ]
    logConfiguration = {
      logDriver = "awslogs"
      options = {
        "awslogs-region"        = var.aws_region
        "awslogs-group"         = aws_cloudwatch_log_group.new_task.name
        "awslogs-stream-prefix" = "ecs"
      }
    }
  }])
}
```

### Step 2: Create CloudWatch Log Group

```hcl
resource "aws_cloudwatch_log_group" "new_task" {
  name              = "/ecs/new-task-dev"
  retention_in_days = 30
}
```

### Step 3: Add Secrets (if needed)

```hcl
resource "aws_ssm_parameter" "new_task_secret" {
  name  = "/dev/new-task/SECRET"
  type  = "SecureString"
  value = var.new_task_secret
}
```

### Step 4: Create EventBridge Trigger (optional)

```hcl
resource "aws_cloudwatch_event_rule" "new_task_schedule" {
  name                = "run-new-task-daily"
  schedule_expression = "cron(0 10 * * ? *)"
}

resource "aws_cloudwatch_event_target" "new_task" {
  rule      = aws_cloudwatch_event_rule.new_task_schedule.name
  target_id = "run-new-task"
  arn       = aws_ecs_cluster.cluster.arn
  role_arn  = aws_iam_role.events_invoke_ecs_role.arn
  
  ecs_target {
    task_definition_arn = aws_ecs_task_definition.new_task.arn
    task_count          = 1
    launch_type         = "FARGATE"
    network_configuration {
      subnets          = var.subnet_ids
      security_groups  = var.security_group_ids
      assign_public_ip = true
    }
  }
}
```

### Step 5: Apply

```bash
cd environments/dev/services/ecs
export AWS_PROFILE=penelope
terraform plan
terraform apply
```

---

## Troubleshooting

### Task Fails to Start

**Check**:
1. Image exists in GHCR? (`docker pull <image>`)
2. GHCR credentials valid? (check Secrets Manager)
3. Task role has permissions? (check IAM policies)
4. Secrets exist in SSM? (`aws ssm get-parameter --name /dev/...`)

### Task Runs but Fails

**Check Logs**:
```bash
aws logs tail /ecs/<task-name>-dev --follow --profile penelope
```

### Task Can't Reach Internet

**Check**:
- Task in public subnet?
- Public IP assigned?
- Security group allows outbound?

### Task Can't Reach RDS

**Check**:
- RDS security group allows traffic from ECS SG?
- Connection string correct?
- Database exists?

---

## Performance Tuning

### CPU/Memory Sizing

| Workload Type | CPU | Memory | Rationale |
|---------------|-----|--------|-----------|
| **Light** (API calls) | 256 | 512 | Minimal overhead |
| **Medium** (data processing) | 512 | 1024 | Current default |
| **Heavy** (ML inference) | 1024-2048 | 2048-4096 | CPU-intensive |

### Concurrency

**Current**: 1 task per execution  
**Future**: Multiple tasks in parallel (fan-out pattern)

---

## Related Documentation

- [compute-resources.md](./compute-resources.md) - Task sizing details
- [container-registry.md](./container-registry.md) - GHCR configuration
- [serverless.md](./serverless.md) - Lambda functions
- [../05-observability-infrastructure/logging-stack.md](../05-observability-infrastructure/logging-stack.md) - CloudWatch Logs

---

**Next Steps**:
1. 📋 Re-enable workflows when ready
2. 📋 Implement ALB for searxng (health checks, domain)
3. 📋 Add Fargate Spot for cost optimization
4. 📋 Implement auto-scaling for services
