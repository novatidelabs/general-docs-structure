# Operations

**Version:** 1.0  
**Last Updated:** [Date]  
**Status:** [Draft | Review | Approved]

---

## Table of Contents
1. [Deployment](#1-deployment)
2. [Configuration](#2-configuration)
3. [Monitoring](#3-monitoring)

---

## 1. Deployment

### 1.1 Environments

| Environment | Purpose | URL | Deploy Trigger | Data |
|-------------|---------|-----|----------------|------|
| **Development** | Local development | `http://localhost:8000` | Manual | Synthetic/test data |
| **Staging** | Pre-production testing | `https://staging.example.com` | Merge to `develop` | Anonymized production data |
| **Production** | Live system | `https://api.example.com` | Merge to `main` + approval | Real production data |

### 1.2 Deployment Strategy

**Strategy:** [Blue-Green | Canary | Rolling | Feature Flags]

**Rationale:** [Why this strategy was chosen]

**Deployment Flow:**

```
Developer         GitHub           CI/CD             Staging          Production
    │                │                │                 │                 │
    ├──Push Code────▶│                │                 │                 │
    │                ├──Trigger CI────▶│                 │                 │
    │                │                ├──Run Tests──────┤                 │
    │                │                ├──Build Image────┤                 │
    │                │                ├──Deploy─────────▶│                 │
    │                │                ├──Run E2E Tests──┤                 │
    │                │                │                 │                 │
    │                │                ├──Wait Approval──┤                 │
    │◀──Approve PR───┤                │                 │                 │
    │                │                ├──Deploy─────────┴────────────────▶│
    │                │                ├──Health Check───────────────────▶│
    │                │                ├──Monitor────────────────────────▶│
```

### 1.3 Deployment Process

#### Pre-Deployment Checklist
- [ ] All tests passing (unit, integration, E2E)
- [ ] Code reviewed and approved
- [ ] Database migrations tested
- [ ] Configuration validated
- [ ] Rollback plan ready
- [ ] Stakeholders notified (for production)

#### Deployment Steps

**Staging Deployment:**
```bash
# Triggered automatically on merge to develop branch
# 1. CI runs tests
# 2. Build Docker image
# 3. Push to container registry
# 4. Deploy to staging environment
# 5. Run smoke tests
```

**Production Deployment:**
```bash
# 1. Merge to main branch (requires approval)
# 2. Tag release: git tag -a v1.0.0 -m "Release v1.0.0"
# 3. Push tag: git push origin v1.0.0
# 4. CI builds production image
# 5. Manual approval gate
# 6. Deploy to production
# 7. Health checks
# 8. Monitor for 30 minutes
```

#### Manual Deployment (Emergency)
```bash
# SSH to production server (avoid if possible)
ssh deploy@production-server

# Pull latest code
cd /app
git pull origin main

# Install dependencies
pip install -r requirements.txt

# Run migrations
alembic upgrade head

# Restart service
sudo systemctl restart app

# Verify health
curl http://localhost:8000/health
```

### 1.4 Rollback Strategy

**Rollback Triggers:**
- Error rate > 5% for more than 5 minutes
- Critical functionality broken
- Database corruption
- Security incident

**Rollback Process:**

**Container-based (Kubernetes/ECS):**
```bash
# Rollback to previous version
kubectl rollout undo deployment/app-deployment

# Or rollback to specific revision
kubectl rollout undo deployment/app-deployment --to-revision=2
```

**Traditional Deployment:**
```bash
# Revert to previous git tag
git checkout v1.0.0
./deploy.sh production
```

**Database Rollback:**
```bash
# Rollback migrations (if safe)
alembic downgrade -1

# Or restore from backup (if needed)
./scripts/restore_database.sh backup-20260115-1030
```

**Rollback Validation:**
- [ ] Service health checks passing
- [ ] Error rates back to normal
- [ ] Critical user flows working
- [ ] Database integrity verified
- [ ] Incident documented

### 1.5 Release Process

**Release Cadence:** [Weekly | Bi-weekly | As needed]

**Release Checklist:**
1. **Code Freeze:** [2 days before release]
2. **Staging Deployment:** [1 day before release]
3. **QA Sign-off:** [Required before production]
4. **Release Notes:** [Prepared and reviewed]
5. **Production Deployment:** [Scheduled maintenance window]
6. **Post-Deployment Monitoring:** [4 hours minimum]
7. **Retrospective:** [Within 1 week]

**Release Notes Template:**
```markdown
## Release v1.0.0 - 2026-01-15

### New Features
- [Feature 1]: Description
- [Feature 2]: Description

### Bug Fixes
- [Fix 1]: Description
- [Fix 2]: Description

### Improvements
- [Improvement 1]: Description

### Breaking Changes
- [Breaking change]: Description and migration guide

### Database Migrations
- Migration 001: Description
- Migration 002: Description

### Deployment Notes
- [Special deployment considerations]

### Rollback Plan
- [Specific rollback instructions if needed]
```

### 1.6 Infrastructure as Code

**Tool:** [Terraform | CloudFormation | Pulumi]

**Repository:** [Link to IaC repository]

**Structure:**
```
infrastructure/
├── environments/
│   ├── dev/
│   ├── staging/
│   └── production/
├── modules/
│   ├── database/
│   ├── compute/
│   └── networking/
└── README.md
```

**Provisioning Process:**
```bash
# Initialize Terraform
cd infrastructure/environments/production
terraform init

# Plan changes
terraform plan -out=plan.tfplan

# Review plan carefully
cat plan.tfplan

# Apply changes (with approval)
terraform apply plan.tfplan

# Verify infrastructure
./scripts/verify_infrastructure.sh
```

---

## 2. Configuration

### 2.1 Configuration Management

**Method:** Environment variables + configuration files

**Configuration Hierarchy:**
1. Default values (in code)
2. Configuration files (config/default.yaml)
3. Environment-specific files (config/production.yaml)
4. Environment variables (highest priority)

### 2.2 Environment Variables

**Required Environment Variables:**

| Variable | Description | Example | Required |
|----------|-------------|---------|----------|
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://user:pass@host:5432/db` | Yes |
| `REDIS_URL` | Redis connection string | `redis://host:6379/0` | Yes |
| `SECRET_KEY` | JWT signing key | `[random-256-bit-key]` | Yes |
| `LOG_LEVEL` | Logging level | `INFO` | No (default: INFO) |
| `ENVIRONMENT` | Environment name | `production` | Yes |

**Development `.env` file:**
```bash
# Database
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/myapp_dev

# Redis
REDIS_URL=redis://localhost:6379/0

# Security
SECRET_KEY=dev-secret-key-change-in-production
JWT_EXPIRATION_MINUTES=60

# External APIs
EXTERNAL_API_KEY=dev-api-key
EXTERNAL_API_URL=https://api.example.com

# Logging
LOG_LEVEL=DEBUG

# Environment
ENVIRONMENT=development
```

**Production Configuration:**
- Environment variables set via [AWS Systems Manager Parameter Store | Kubernetes Secrets | etc]
- Secrets managed via [AWS Secrets Manager | Vault | etc]
- Never commit secrets to version control

### 2.3 Secrets Management

**Tool:** [AWS Secrets Manager | HashiCorp Vault | GCP Secret Manager]

**Secret Rotation:**
- Database passwords: Rotated every 90 days
- API keys: Rotated every 180 days
- JWT signing keys: Rotated every 365 days

**Accessing Secrets:**
```python
# Example: AWS Secrets Manager
import boto3
import json

def get_secret(secret_name):
    client = boto3.client('secretsmanager', region_name='us-east-1')
    response = client.get_secret_value(SecretId=secret_name)
    return json.loads(response['SecretString'])

# Usage
db_credentials = get_secret('myapp/database/production')
```

### 2.4 Feature Flags

**Tool:** [LaunchDarkly | Split | Custom]

**Usage:**
```python
# Example: Feature flag check
from feature_flags import FeatureFlags

flags = FeatureFlags()

if flags.is_enabled('new_ui_feature', user_id=user.id):
    # Show new UI
    return render_new_ui()
else:
    # Show old UI
    return render_old_ui()
```

**Feature Flag Strategy:**
- Use for gradual rollouts
- Use for A/B testing
- Use for kill switches
- Clean up flags after full rollout

### 2.5 Configuration Validation

**Startup Validation:**
```python
# Example: Configuration validation on startup
from pydantic import BaseSettings, validator

class Settings(BaseSettings):
    database_url: str
    redis_url: str
    secret_key: str
    environment: str
    
    @validator('environment')
    def validate_environment(cls, v):
        valid = ['development', 'staging', 'production']
        if v not in valid:
            raise ValueError(f'Invalid environment: {v}')
        return v
    
    @validator('secret_key')
    def validate_secret_key(cls, v):
        if len(v) < 32:
            raise ValueError('Secret key must be at least 32 characters')
        return v

# Application fails to start if configuration is invalid
settings = Settings()
```

---

## 3. Monitoring

### 3.1 Monitoring Stack

**Infrastructure:**
- **Metrics:** [Prometheus | Datadog | CloudWatch]
- **Logging:** [ELK Stack | CloudWatch Logs | Datadog Logs]
- **Tracing:** [Jaeger | X-Ray | Datadog APM]
- **Dashboards:** [Grafana | Datadog | CloudWatch Dashboards]
- **Alerting:** [PagerDuty | Opsgenie | Slack]

### 3.2 Key Metrics

**Application Metrics:**

| Metric | Target | Alert Threshold | Description |
|--------|--------|-----------------|-------------|
| **Request Rate** | - | - | Requests per second |
| **Error Rate** | < 0.1% | > 1% for 5 min | Percentage of failed requests |
| **Response Time (p95)** | < 200ms | > 500ms for 5 min | 95th percentile response time |
| **Response Time (p99)** | < 500ms | > 1000ms for 5 min | 99th percentile response time |
| **Database Connection Pool** | < 80% | > 90% for 5 min | Active connections / pool size |
| **Cache Hit Rate** | > 80% | < 60% for 10 min | Cache hits / total requests |

**Infrastructure Metrics:**

| Metric | Target | Alert Threshold | Description |
|--------|--------|-----------------|-------------|
| **CPU Usage** | < 70% | > 85% for 10 min | CPU utilization percentage |
| **Memory Usage** | < 80% | > 90% for 5 min | Memory utilization percentage |
| **Disk Usage** | < 70% | > 85% | Disk space utilization |
| **Network I/O** | - | Anomaly detection | Network traffic patterns |

**Business Metrics:**

| Metric | Description | Dashboard |
|--------|-------------|-----------|
| **Active Users** | Current active user count | Real-time |
| **Signups** | New user registrations per hour | Daily |
| **[Business Event]** | [Description] | [Frequency] |

### 3.3 Logging

**Log Format:**
```json
{
  "timestamp": "2026-01-15T10:30:00.000Z",
  "level": "INFO",
  "service": "backend",
  "environment": "production",
  "request_id": "uuid-1234",
  "user_id": "uuid-5678",
  "message": "User logged in successfully",
  "metadata": {
    "ip_address": "192.168.1.1",
    "user_agent": "Mozilla/5.0..."
  }
}
```

**Log Levels:**
- `DEBUG`: Detailed debugging (development only)
- `INFO`: General operational messages
- `WARNING`: Potential issues that don't prevent operation
- `ERROR`: Errors that need attention
- `CRITICAL`: Critical failures requiring immediate action

**Log Retention:**
- Development: 7 days
- Staging: 30 days
- Production: 90 days

**Searchable Fields:**
- `request_id`: Trace requests across services
- `user_id`: Track user-specific issues
- `error_code`: Filter by error types
- `service`: Filter by component

### 3.4 Distributed Tracing

**Trace Context Propagation:**
```python
# Example: Trace context in HTTP headers
headers = {
    'X-Request-ID': request_id,
    'X-Trace-ID': trace_id,
    'X-Span-ID': span_id
}

# All downstream requests include these headers
response = requests.get(url, headers=headers)
```

**Trace Sampling:**
- All errors: 100% sampling
- Slow requests (>1s): 100% sampling
- Normal requests: 10% sampling

### 3.5 Alerting

**Alert Severity Levels:**

| Severity | Response Time | Notification | Example |
|----------|--------------|--------------|---------|
| **Critical** | Immediate | PagerDuty (on-call) | Service down, database unavailable |
| **High** | 15 minutes | PagerDuty + Slack | Error rate > 5%, p95 > 1s |
| **Medium** | 1 hour | Slack | Error rate > 1%, cache hit rate low |
| **Low** | Next business day | Email | Disk usage > 70% |

**Alert Definitions:**

**Critical Alerts:**
```yaml
# Service Health Check Failing
alert: ServiceDown
expr: up{job="backend"} == 0
for: 1m
severity: critical
message: "Backend service is down"
runbook: "https://docs.example.com/runbooks/service-down"

# Database Connection Failed
alert: DatabaseUnavailable
expr: database_up == 0
for: 1m
severity: critical
message: "Database is unavailable"
runbook: "https://docs.example.com/runbooks/database-down"
```

**High Priority Alerts:**
```yaml
# High Error Rate
alert: HighErrorRate
expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
for: 5m
severity: high
message: "Error rate is above 5%"

# Slow Response Time
alert: SlowResponseTime
expr: histogram_quantile(0.95, http_request_duration_seconds) > 0.5
for: 5m
severity: high
message: "p95 response time is above 500ms"
```

**Alert Runbooks:**
Each alert must have a runbook that includes:
- Description of the alert
- Potential causes
- Investigation steps
- Resolution procedures
- Escalation path if unresolved

### 3.6 Dashboards

**Production Dashboard:**
- Request rate (last 1h, 24h, 7d)
- Error rate (last 1h, 24h)
- Response time (p50, p95, p99)
- Active users
- Database connections
- Cache hit rate
- Recent deployments
- Active alerts

**Infrastructure Dashboard:**
- CPU usage per instance
- Memory usage per instance
- Disk I/O
- Network I/O
- Container health
- Database metrics

**Business Metrics Dashboard:**
- User signups (daily, weekly, monthly)
- Active users (hourly, daily)
- [Business-specific metrics]

### 3.7 Health Checks

**Health Check Endpoint:**
```http
GET /health

Response: 200 OK
{
  "status": "healthy",
  "version": "1.0.0",
  "timestamp": "2026-01-15T10:30:00Z",
  "checks": {
    "database": "healthy",
    "redis": "healthy",
    "external_api": "healthy"
  }
}

Response: 503 Service Unavailable (if unhealthy)
{
  "status": "unhealthy",
  "version": "1.0.0",
  "timestamp": "2026-01-15T10:30:00Z",
  "checks": {
    "database": "unhealthy",
    "redis": "healthy",
    "external_api": "timeout"
  }
}
```

**Health Check Types:**
1. **Liveness:** Is the service running?
2. **Readiness:** Is the service ready to accept traffic?
3. **Dependency:** Are dependencies healthy?

### 3.8 Incident Response

**Incident Severity:**

| Severity | Definition | Response Time | Example |
|----------|-----------|---------------|---------|
| **SEV-1** | Critical outage | Immediate | Complete service outage |
| **SEV-2** | Major degradation | 15 minutes | Core functionality broken |
| **SEV-3** | Partial degradation | 1 hour | Non-critical feature broken |
| **SEV-4** | Minor issue | Next business day | UI bug, typo |

**Incident Response Process:**

1. **Detection:**
   - Alert fires → PagerDuty notifies on-call
   - User report → Support escalates to engineering

2. **Triage (5 minutes):**
   - Assess severity
   - Create incident channel (#incident-YYYY-MM-DD)
   - Assign incident commander
   - Notify stakeholders if SEV-1 or SEV-2

3. **Investigation (varies by severity):**
   - Check recent deployments
   - Review logs and metrics
   - Check dependencies
   - Document findings in incident channel

4. **Mitigation (varies by severity):**
   - Rollback deployment (if deployment-related)
   - Scale up resources (if capacity-related)
   - Disable feature flag (if feature-related)
   - Apply hotfix (if code-related)

5. **Resolution:**
   - Verify metrics back to normal
   - Monitor for 30+ minutes
   - Communicate resolution to stakeholders
   - Close incident

6. **Post-Mortem (within 48 hours for SEV-1/SEV-2):**
   - Timeline of events
   - Root cause analysis
   - Action items to prevent recurrence
   - Share learnings with team

**On-Call Rotation:**
- **Schedule:** [Weekly rotation | PagerDuty schedule link]
- **Responsibilities:**
  - Respond to alerts within 5 minutes
  - Triage and resolve incidents
  - Escalate if needed
  - Document incidents
- **Handoff:** [Monday 9am | includes runbook review]

### 3.9 Disaster Recovery

**Backup Strategy:**
- **Database:** Automated daily backups, 30-day retention
- **Files:** S3 versioning enabled, lifecycle policy
- **Configuration:** Version controlled in Git

**Recovery Time Objective (RTO):** [1 hour]  
**Recovery Point Objective (RPO):** [15 minutes]

**Disaster Recovery Procedures:**

1. **Database Failure:**
   ```bash
   # Restore from latest backup
   ./scripts/restore_database.sh latest
   
   # Or restore from specific backup
   ./scripts/restore_database.sh backup-20260115-1030
   
   # Verify data integrity
   ./scripts/verify_database.sh
   ```

2. **Complete Service Failure:**
   ```bash
   # Provision new infrastructure (if infrastructure destroyed)
   cd infrastructure/environments/production
   terraform apply
   
   # Deploy application
   ./deploy.sh production
   
   # Restore database
   ./scripts/restore_database.sh latest
   
   # Verify service health
   curl https://api.example.com/health
   ```

3. **Regional Outage (if multi-region):**
   - Failover to secondary region
   - Update DNS to point to secondary region
   - Monitor secondary region closely

**DR Testing:**
- Quarterly DR drills
- Document drill results
- Update procedures based on learnings

---

## 4. Operational Procedures

### 4.1 Database Operations

**Database Migrations:**
```bash
# Create migration
alembic revision -m "add_new_column"

# Review migration
cat alembic/versions/YYYYMMDD_add_new_column.py

# Apply migration in staging
alembic upgrade head

# Verify migration
psql -h staging-db -U user -d database -c "\d table_name"

# Apply migration in production (during maintenance window)
alembic upgrade head
```

**Database Backup Verification:**
```bash
# Weekly backup verification
./scripts/verify_backups.sh

# Checklist:
# - Backup file exists
# - Backup file is not corrupt
# - Backup can be restored to test database
# - Data integrity checks pass
```

### 4.2 Certificate Management

**Certificate Renewal:**
- Certificates renewed automatically via [Let's Encrypt / AWS ACM]
- Expiration monitoring: Alert 30 days before expiration
- Manual renewal process (if needed):
  ```bash
  # Renew certificate
  certbot renew
  
  # Restart services
  sudo systemctl reload nginx
  ```

### 4.3 Security Patching

**Patching Schedule:**
- **Critical Security Patches:** Within 24 hours
- **High Priority Patches:** Within 1 week
- **Regular Updates:** Monthly maintenance window

**Patching Process:**
1. Review security advisories
2. Test patches in staging
3. Schedule maintenance window
4. Apply patches to production
5. Verify service health
6. Document patches applied

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | [Date] | [Name] | Initial version |

**Next Review:** [Date]
