# Architecture

**Version:** 1.0  
**Last Updated:** [Date]  
**Status:** [Draft | Review | Approved]

---

## Table of Contents
1. [System Architecture](#1-system-architecture)
2. [API Contracts](#2-api-contracts)
3. [Database Schema](#3-database-schema)

---

## 1. System Architecture

### 1.1 System Context

**External Actors & Systems:**

```
┌──────────────────────────────────────────────────────────────┐
│                     System Context                            │
│                                                               │
│  ┌─────────┐                                                 │
│  │  Users  │────┐                                            │
│  └─────────┘    │                                            │
│                 │         ┌─────────────────┐                │
│  ┌─────────┐    ├────────▶│                 │                │
│  │ Admins  │────┘         │  [Project Name] │                │
│  └─────────┘              │                 │                │
│                           └────────┬────────┘                │
│  ┌──────────┐                      │                         │
│  │External  │◀─────────────────────┤                         │
│  │System A  │                      │                         │
│  └──────────┘                      │                         │
│                           ┌────────▼────────┐                │
│  ┌──────────┐             │   External      │                │
│  │External  │◀────────────│   System B      │                │
│  │System C  │             └─────────────────┘                │
│  └──────────┘                                                │
└──────────────────────────────────────────────────────────────┘
```

| Actor/System | Relationship | Protocol | Purpose |
|--------------|-------------|----------|---------|
| [External System] | [Consumes/Provides] | [REST/gRPC/Event] | [Purpose] |

### 1.2 Container Architecture (C4 Level 2)

**System Components:**

```
┌────────────────────────────────────────────────────────────────┐
│                      [Project Name]                            │
│                                                                │
│  ┌─────────────┐         ┌─────────────┐                      │
│  │   Client    │         │   Admin     │                      │
│  │ Application │         │   Portal    │                      │
│  │  (React)    │         │  (React)    │                      │
│  └──────┬──────┘         └──────┬──────┘                      │
│         │                       │                             │
│         │ HTTPS/REST            │ HTTPS/REST                  │
│         │                       │                             │
│  ┌──────▼───────────────────────▼──────┐                      │
│  │         API Gateway / LB            │                      │
│  └──────┬───────────────────────┬──────┘                      │
│         │                       │                             │
│  ┌──────▼──────┐         ┌──────▼──────┐                      │
│  │   Backend   │         │   Worker    │                      │
│  │  Service    │◀────────│   Service   │                      │
│  │  (FastAPI)  │  Events │  (Celery)   │                      │
│  └──────┬──────┘         └──────┬──────┘                      │
│         │                       │                             │
│         ├───────────┬───────────┤                             │
│         │           │           │                             │
│  ┌──────▼──┐  ┌────▼────┐ ┌────▼─────┐                       │
│  │Database │  │  Cache  │ │  Queue   │                       │
│  │(Postgres)  │ (Redis) │ │ (RabbitMQ)│                       │
│  └─────────┘  └─────────┘ └──────────┘                       │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### 1.3 Component Details

#### Component: Backend Service
- **Responsibility:** Primary API, business logic, data validation
- **Technology:** Python 3.11+, FastAPI
- **Port:** 8000
- **Dependencies:**
  - PostgreSQL (read/write)
  - Redis (caching, session storage)
  - RabbitMQ (async task publishing)
- **Exposed Interfaces:**
  - REST API (OpenAPI 3.0)
  - Health check endpoint
- **Location:** `/src/backend/`

#### Component: Worker Service
- **Responsibility:** Async background jobs, scheduled tasks
- **Technology:** Python 3.11+, Celery
- **Dependencies:**
  - PostgreSQL (read/write)
  - Redis (result backend)
  - RabbitMQ (task queue)
- **Exposed Interfaces:** None (internal only)
- **Location:** `/src/workers/`

#### Component: Database
- **Responsibility:** Persistent data storage
- **Technology:** PostgreSQL 15+
- **Configuration:**
  - Primary: read/write
  - [Optional] Replica: read-only
- **Backup:** Daily snapshots, 30-day retention

#### Component: Cache
- **Responsibility:** Session storage, query caching, rate limiting
- **Technology:** Redis 7+
- **TTL Strategy:** [Default 1 hour, configurable per key]

#### Component: Message Queue
- **Responsibility:** Async task distribution
- **Technology:** RabbitMQ / AWS SQS
- **Configuration:** [Queue names, DLQ strategy]

### 1.4 Component Interactions

**Critical Flow: [Flow Name - e.g., User Registration]**

```
Client          API Gateway       Backend         Database        Queue           Worker
  │                 │               │                │              │               │
  ├────Request─────▶│               │                │              │               │
  │                 ├───Forward────▶│                │              │               │
  │                 │               ├──Validate─────▶│              │               │
  │                 │               │◀──Response─────┤              │               │
  │                 │               ├──Publish Task────────────────▶│               │
  │                 │               │                │              │               │
  │                 │◀──Response────┤                │              │               │
  │◀────Response────┤               │                │              │               │
  │                 │               │                │              ├──Consume─────▶│
  │                 │               │                │              │               │
  │                 │               │                │◀─────Process Task────────────┤
```

1. Client sends request to API Gateway
2. API Gateway forwards to Backend Service
3. Backend validates request and stores data in Database
4. Backend publishes async task to Queue
5. Backend returns response to client
6. Worker consumes task from Queue
7. Worker processes task and updates Database

### 1.5 Architecture Patterns

**Applied Patterns:**

1. **Layered Architecture**
   - Presentation Layer (API)
   - Business Logic Layer (Services)
   - Data Access Layer (Repositories)
   - Infrastructure Layer (DB, Cache, Queue)

2. **Repository Pattern**
   - Abstracts data access
   - Enables testing without database
   - Location: `/src/repositories/`

3. **Dependency Injection**
   - FastAPI dependency injection system
   - Configuration via environment variables
   - Secrets managed via [tool]

4. **Circuit Breaker** (if applicable)
   - Protects against cascading failures
   - Implementation: [Library/pattern used]

### 1.6 Cross-Cutting Concerns

#### Authentication & Authorization
- **Strategy:** JWT-based authentication
- **Flow:**
  1. User authenticates (username/password or OAuth)
  2. System issues JWT (access token + refresh token)
  3. Access token expires in [duration]
  4. Refresh token expires in [duration]
- **Authorization:** Role-based access control (RBAC)
- **Roles:**
  - `admin`: Full access
  - `user`: Standard access
  - `readonly`: Read-only access

#### Error Handling
- **Standard Error Format:**
```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message",
    "details": {},
    "timestamp": "2026-01-15T10:30:00Z",
    "request_id": "uuid"
  }
}
```

- **Error Codes:** See API documentation
- **Error Tracking:** [Sentry/Rollbar/etc]

#### Logging
- **Format:** Structured JSON logging
- **Log Levels:**
  - `ERROR`: Errors requiring immediate attention
  - `WARNING`: Potential issues
  - `INFO`: General operational messages
  - `DEBUG`: Detailed debugging (dev only)
- **Correlation:** Each request has unique `request_id`
- **Storage:** [CloudWatch/ELK/etc]
- **Retention:** [30 days]

#### Monitoring
- **Metrics:**
  - Request rate (req/s)
  - Error rate (%)
  - Response time (p50, p95, p99)
  - Database connection pool usage
  - Cache hit/miss ratio
- **Alerting:** See [operations.md](./operations.md)

#### Configuration Management
- **Method:** Environment variables
- **Secrets:** [AWS Secrets Manager / Vault / etc]
- **Feature Flags:** [LaunchDarkly / etc] (if used)

---

## 2. API Contracts

### 2.1 API Overview

**Base URL:**
- **Development:** `http://localhost:8000`
- **Staging:** `https://staging-api.example.com`
- **Production:** `https://api.example.com`

**API Version:** v1  
**Documentation:** `/docs` (Swagger UI), `/redoc` (ReDoc)

### 2.2 Authentication

**All authenticated endpoints require:**
```http
Authorization: Bearer <access_token>
```

**Token Endpoint:**
```http
POST /api/v1/auth/token
Content-Type: application/json

{
  "username": "user@example.com",
  "password": "securepassword"
}

Response:
{
  "access_token": "eyJhbGc...",
  "refresh_token": "eyJhbGc...",
  "token_type": "bearer",
  "expires_in": 3600
}
```

### 2.3 Core Endpoints

#### [Resource Name - e.g., Users]

**List Users**
```http
GET /api/v1/users
Query Parameters:
  - page: int (default: 1)
  - page_size: int (default: 20, max: 100)
  - sort_by: string (default: "created_at")
  - order: string (default: "desc", options: "asc", "desc")

Response: 200 OK
{
  "data": [
    {
      "id": "uuid",
      "email": "user@example.com",
      "name": "John Doe",
      "role": "user",
      "created_at": "2026-01-15T10:30:00Z",
      "updated_at": "2026-01-15T10:30:00Z"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 20,
    "total_count": 150,
    "total_pages": 8
  }
}
```

**Get User by ID**
```http
GET /api/v1/users/{user_id}

Response: 200 OK
{
  "id": "uuid",
  "email": "user@example.com",
  "name": "John Doe",
  "role": "user",
  "created_at": "2026-01-15T10:30:00Z",
  "updated_at": "2026-01-15T10:30:00Z"
}

Error: 404 Not Found
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User with ID 'uuid' not found"
  }
}
```

**Create User**
```http
POST /api/v1/users
Content-Type: application/json

{
  "email": "newuser@example.com",
  "name": "Jane Doe",
  "password": "securepassword",
  "role": "user"
}

Response: 201 Created
{
  "id": "uuid",
  "email": "newuser@example.com",
  "name": "Jane Doe",
  "role": "user",
  "created_at": "2026-01-15T10:30:00Z"
}

Error: 400 Bad Request
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input",
    "details": {
      "email": ["Email already exists"]
    }
  }
}
```

**Update User**
```http
PUT /api/v1/users/{user_id}
Content-Type: application/json

{
  "name": "Jane Smith",
  "role": "admin"
}

Response: 200 OK
{
  "id": "uuid",
  "email": "newuser@example.com",
  "name": "Jane Smith",
  "role": "admin",
  "updated_at": "2026-01-15T11:00:00Z"
}
```

**Delete User**
```http
DELETE /api/v1/users/{user_id}

Response: 204 No Content
```

### 2.4 Event Contracts (if applicable)

**Event: user.created**
```json
{
  "event_type": "user.created",
  "event_id": "uuid",
  "timestamp": "2026-01-15T10:30:00Z",
  "data": {
    "user_id": "uuid",
    "email": "user@example.com",
    "name": "John Doe"
  }
}
```

**Event: user.updated**
```json
{
  "event_type": "user.updated",
  "event_id": "uuid",
  "timestamp": "2026-01-15T10:30:00Z",
  "data": {
    "user_id": "uuid",
    "changes": {
      "name": "New Name",
      "role": "admin"
    }
  }
}
```

### 2.5 Versioning Strategy

- **Current Version:** v1
- **Versioning Method:** URL path (`/api/v1/`, `/api/v2/`)
- **Deprecation Policy:**
  - 6 months notice before deprecation
  - Minimum 3 months support for old version after new version launch
  - Deprecation warnings in response headers

### 2.6 Rate Limiting

- **Anonymous:** 100 requests/hour
- **Authenticated:** 1000 requests/hour
- **Response Headers:**
  ```
  X-RateLimit-Limit: 1000
  X-RateLimit-Remaining: 999
  X-RateLimit-Reset: 1642248000
  ```

### 2.7 OpenAPI Specification

**Location:** `/openapi.json` or `/api/v1/openapi.json`

Full API specification available at `/docs` (Swagger UI).

---

## 3. Database Schema

### 3.1 Schema Overview

**Database:** PostgreSQL 15+  
**Schema Name:** `public` (or custom schema)

**Key Tables:**
- `users` - User accounts and authentication
- `[table_2]` - [Purpose]
- `[table_3]` - [Purpose]

### 3.2 Entity Relationship Diagram

```
┌─────────────────┐
│     users       │
├─────────────────┤
│ id (PK)         │
│ email           │
│ password_hash   │
│ name            │
│ role            │
│ created_at      │
│ updated_at      │
└────────┬────────┘
         │
         │ 1:N
         │
┌────────▼────────┐
│   [related]     │
├─────────────────┤
│ id (PK)         │
│ user_id (FK)    │
│ ...             │
└─────────────────┘
```

### 3.3 Table Definitions

#### Table: users

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL DEFAULT 'user',
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
CREATE INDEX idx_users_created_at ON users(created_at);
```

**Constraints:**
- `email` must be unique and valid email format
- `role` must be one of: `admin`, `user`, `readonly`
- Soft deletes via `deleted_at` timestamp

#### Table: [table_name]

```sql
CREATE TABLE [table_name] (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    [field_name] VARCHAR(255) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_[table]_user_id ON [table_name](user_id);
```

### 3.4 Data Access Patterns

**Common Queries:**

1. **Get active users:**
```sql
SELECT * FROM users 
WHERE deleted_at IS NULL 
ORDER BY created_at DESC 
LIMIT 20 OFFSET 0;
```

2. **Get user with related data:**
```sql
SELECT u.*, COUNT(r.*) as related_count
FROM users u
LEFT JOIN [related] r ON r.user_id = u.id
WHERE u.id = ?
GROUP BY u.id;
```

**Performance Considerations:**
- Users table partitioned by `created_at` (if > 10M rows)
- Indexes on frequently queried columns
- Connection pooling (min: 5, max: 20)

### 3.5 Migrations

**Migration Tool:** Alembic (Python) / Flyway (Java)

**Location:** `/migrations/` or `/alembic/versions/`

**Migration Naming:**
```
{timestamp}_{description}.py
Example: 20260115_add_users_table.py
```

**Migration Process:**
1. Generate migration: `alembic revision -m "description"`
2. Review generated SQL
3. Test migration in dev environment
4. Apply to staging: `alembic upgrade head`
5. Verify data integrity
6. Apply to production

**Rollback Strategy:**
- All migrations must have `downgrade()` function
- Test rollback in dev before production deployment
- Database backup before major migrations

### 3.6 Data Seeding

**Seed Data Location:** `/seeds/` or `/fixtures/`

**Development Seeds:**
- Sample users
- Test data for all major entities
- Relationships between entities

**Command:**
```bash
python scripts/seed_database.py --env development
```

### 3.7 Data Retention & Archival

**Retention Policies:**
| Table | Retention Period | Archival Strategy | Deletion Policy |
|-------|------------------|-------------------|-----------------|
| users | Indefinite (soft delete) | N/A | Manual purge after 2 years of inactivity |
| [table] | [period] | [strategy] | [policy] |

**Soft Delete Pattern:**
- Set `deleted_at` timestamp instead of hard delete
- Exclude `deleted_at IS NOT NULL` in queries
- Scheduled cleanup job purges after retention period

---

## 4. Security Architecture

### 4.1 Authentication Flow

```
Client                    Backend                   Database
  │                         │                          │
  ├──POST /auth/token──────▶│                          │
  │  (username, password)   │                          │
  │                         ├──Verify Credentials─────▶│
  │                         │◀──User Record────────────┤
  │                         │                          │
  │                         ├──Generate JWT────────────┤
  │◀──Return JWT────────────┤                          │
  │                         │                          │
  ├──GET /api/resource─────▶│                          │
  │  (Authorization: Bearer)│                          │
  │                         ├──Verify JWT──────────────┤
  │                         ├──Check Permissions───────┤
  │◀──Return Resource───────┤                          │
```

### 4.2 Data Protection

**Encryption at Rest:**
- Database: [AES-256 encryption, managed by RDS]
- File storage: [S3 server-side encryption]
- Backups: [Encrypted snapshots]

**Encryption in Transit:**
- TLS 1.3 for all HTTPS connections
- Certificate management: [Let's Encrypt / AWS ACM]
- Minimum cipher suite: [specification]

**Sensitive Data Handling:**
- Passwords: bcrypt hashing (cost factor: 12)
- API keys: Stored in secrets manager
- PII: [Strategy for handling PII]

### 4.3 Input Validation

**Validation Strategy:**
- Pydantic models for request validation
- SQL injection prevention via parameterized queries
- XSS prevention via output encoding
- CSRF protection via tokens (if applicable)

### 4.4 Compliance

**Standards:**
- [GDPR compliance: data portability, right to deletion]
- [SOC 2 Type II: access controls, audit logging]

**Audit Logging:**
- All authentication attempts
- All administrative actions
- All data access/modifications
- Retention: [1 year minimum]

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | [Date] | [Name] | Initial version |

**Next Review:** [Date]
