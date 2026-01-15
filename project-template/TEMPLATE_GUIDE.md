# Single-Project Documentation Template

**Version:** 1.0  
**Created:** 2026-01-15  
**Purpose:** Simplified, KISS-based documentation structure for single projects

---

## Overview

This template provides a **single source of truth** documentation structure for **monolithic applications and small-to-medium projects**. It consolidates all essential documentation into 5 core files plus individual ADRs.

### Design Principles

1. **Single Source of Truth:** Each topic has exactly one authoritative location
2. **KISS (Keep It Simple, Stupid):** No unnecessary complexity or structure
3. **Self-Contained Documents:** Each file is comprehensive and standalone
4. **Scalable:** Easy to upgrade to full microservices structure if needed
5. **Developer-Friendly:** Quick to navigate, easy to maintain

---

## Documentation Structure

```
/docs
├── README.md                    # Master index and quick navigation
├── overview.md                  # What, why, who (executive summary + tech stack)
├── architecture.md              # How it works (system design + APIs + data)
├── operations.md                # How to run it (deployment + config + monitoring)
├── development.md               # How to build it (setup + standards + testing)
├── decisions.md                 # Why we built it this way (ADR index)
└── adr-YYYYMMDD-title.md        # Individual architecture decisions
```

---

## File Responsibilities

### README.md - Navigation Hub
**Purpose:** Entry point and navigation guide  
**Audience:** Everyone  
**Contents:**
- Quick links to all documents
- Reading order by role
- One-minute overview
- Quick start commands
- Project status dashboard
- Support contacts

**When to update:** Whenever document structure changes

---

### overview.md - Project Context
**Purpose:** Business and technical context  
**Audience:** Everyone, especially new team members and stakeholders  
**Contents:**
1. **Executive Summary**
   - What the project does
   - Why it exists
   - Who it's for
   - Success metrics

2. **Project Context**
   - Scope (in/out)
   - Constraints
   - Dependencies

3. **Architecture Overview**
   - High-level diagram
   - Architecture style and rationale
   - Key components
   - Data flow

4. **Technology Stack**
   - Core technologies with rationale
   - Key libraries
   - Development tools
   - Cloud infrastructure
   - Third-party integrations

**When to update:** 
- Major feature changes
- Technology changes
- Business requirement changes

---

### architecture.md - Technical Design
**Purpose:** Detailed technical architecture  
**Audience:** Engineers, architects, tech leads  
**Contents:**
1. **System Architecture**
   - System context diagram
   - Container architecture (C4 Level 2)
   - Component details
   - Component interactions
   - Architecture patterns
   - Cross-cutting concerns (auth, logging, error handling, config)

2. **API Contracts**
   - API overview and versioning
   - Authentication
   - Core endpoints (full specs)
   - Event contracts (if applicable)
   - Rate limiting
   - OpenAPI specification

3. **Database Schema**
   - Schema overview
   - Entity relationship diagram
   - Table definitions
   - Data access patterns
   - Migrations strategy
   - Data retention

4. **Security Architecture**
   - Authentication flow
   - Data protection
   - Input validation
   - Compliance

**When to update:**
- New features that change architecture
- API changes
- Database schema changes
- Security changes

---

### operations.md - Running the System
**Purpose:** How to deploy, configure, and monitor  
**Audience:** DevOps, SRE, on-call engineers  
**Contents:**
1. **Deployment**
   - Environments
   - Deployment strategy
   - Deployment process
   - Rollback strategy
   - Release process
   - Infrastructure as Code

2. **Configuration**
   - Configuration management
   - Environment variables
   - Secrets management
   - Feature flags
   - Configuration validation

3. **Monitoring**
   - Monitoring stack
   - Key metrics and alerts
   - Logging
   - Distributed tracing
   - Dashboards
   - Health checks
   - Incident response
   - Disaster recovery

**When to update:**
- Infrastructure changes
- Deployment process changes
- New monitoring/alerting rules
- Incident process updates

---

### development.md - Developer Guide
**Purpose:** How to set up, develop, and test  
**Audience:** Engineers  
**Contents:**
1. **Setup Guide**
   - Prerequisites
   - Local setup (Docker + native)
   - Project structure
   - Configuration
   - Development tools
   - Common tasks

2. **Coding Standards**
   - Code style
   - Naming conventions
   - Type hints
   - Documentation
   - Error handling
   - Dependency injection
   - Layered architecture
   - Code review checklist

3. **Testing Guide**
   - Testing philosophy
   - Test structure
   - Unit tests (with examples)
   - Integration tests (with examples)
   - API tests (with examples)
   - Test fixtures
   - Running tests
   - Coverage requirements

4. **Git Workflow**
   - Branching strategy
   - Commit messages
   - Pull request process

**When to update:**
- New development practices
- Tool changes
- Testing strategy changes
- Workflow updates

---

### decisions.md - Architecture Decisions
**Purpose:** Track and explain architectural decisions  
**Audience:** Architects, tech leads, engineers  
**Contents:**
1. **Overview**
   - ADR purpose
   - What requires an ADR
   - ADR lifecycle

2. **ADR Index**
   - Table of all ADRs with status

3. **ADR Template**
   - Standard template for new ADRs

4. **Decision Tracking**
   - Current active decisions
   - Decisions under review
   - Recent changes
   - Upcoming reviews

5. **Decision-Making Process**
   - How to propose
   - How to review
   - How to communicate

**When to update:**
- New architectural decisions
- ADR status changes
- Regular review cycles

---

### adr-YYYYMMDD-title.md - Individual ADRs
**Purpose:** Document specific architectural decisions  
**Audience:** Technical team  
**Contents:**
- Context
- Decision
- Rationale
- Consequences
- Implementation
- Validation
- References

**When to create:**
- Technology choices
- Architecture patterns
- Major refactoring
- Integration decisions
- Breaking changes

---

## Document Maintenance

### Ownership
- **Tech Lead:** Ensures all documents are current
- **Engineers:** Update documents when making changes
- **DevOps:** Maintains operations documentation
- **Product:** Reviews overview for accuracy

### Review Schedule
- **Monthly:** Quick review of all documents
- **Quarterly:** Comprehensive review and updates
- **On-demand:** When major changes occur

### Quality Standards
- No orphaned documents
- No contradictory information
- Examples are current and working
- Links are not broken
- Diagrams are up-to-date

---

## Migration Guide

### From No Documentation
1. Start with README.md and overview.md
2. Add architecture.md as you build
3. Document deployment in operations.md
4. Create development.md for team onboarding
5. Add decisions.md and first ADR when making first major decision

### From Heavy Documentation
1. Consolidate scattered docs into 5 core files
2. Archive old documentation
3. Update links and references
4. Train team on new structure

### To Microservices Structure
When project grows beyond single service:
1. Keep this structure at repository level
2. Add service-specific documentation per service
3. Upgrade to full system-of-systems structure
4. Migrate content systematically

---

## Usage Guidelines

### For New Projects
1. Copy entire template folder
2. Fill in README.md with project basics
3. Complete overview.md before coding
4. Build architecture.md as you build
5. Set up operations.md before first deployment
6. Create development.md for team onboarding

### For Existing Projects
1. Start with README.md (quick wins)
2. Create overview.md from existing docs
3. Extract architecture from code/docs
4. Document current operations
5. Formalize development practices
6. Create first ADR for next major decision

### For Documentation Review
1. Check README navigation links
2. Verify overview reflects current state
3. Validate architecture diagrams
4. Test deployment procedures
5. Try development setup from scratch
6. Review recent ADRs

---

## Benefits

### Single Source of Truth
- No duplicate information
- Clear ownership of each topic
- Easy to keep current

### Developer Velocity
- Fast onboarding
- Quick reference
- Self-service documentation

### Quality
- Consistent structure
- Comprehensive coverage
- Easy to review

### Flexibility
- Works for any project size
- Scales up or down
- Easy to customize

---

## Anti-Patterns to Avoid

### ❌ Don't
- Duplicate information across files
- Create additional "quick reference" docs
- Split single topics across multiple files
- Leave documents unmaintained
- Create documents "just in case"
- Over-engineer the structure

### ✅ Do
- Keep each file self-contained
- Update docs with code changes
- Link between documents when needed
- Archive old documentation
- Remove outdated content
- Follow KISS principle

---

## Checklist for Complete Documentation

**Initial Setup:**
- [ ] README.md with navigation
- [ ] overview.md with business context
- [ ] architecture.md with system design
- [ ] operations.md with deployment guide
- [ ] development.md with setup guide
- [ ] decisions.md with ADR index

**Ongoing Maintenance:**
- [ ] All documents reviewed in last 3 months
- [ ] No contradictory information
- [ ] All examples tested and working
- [ ] All links functional
- [ ] All diagrams current
- [ ] Team knows documentation exists

**Quality Gates:**
- [ ] New engineer can set up locally using development.md
- [ ] DevOps can deploy using operations.md
- [ ] Stakeholders understand value from overview.md
- [ ] Architects understand design from architecture.md
- [ ] Decisions are documented in ADRs

---

## Support

For questions or improvements to this template:
- Create an issue in the template repository
- Propose changes via pull request
- Share feedback with documentation team

---

**Template Version:** 1.0  
**Last Updated:** 2026-01-15  
**Maintained By:** Engineering Documentation Team
