# [Project Name]

**Quick Links:**
- [Overview](#quick-navigation) - Start here to understand the project
- [Architecture](./architecture.md) - Technical architecture and design
- [Operations](./operations.md) - Deployment and production operations
- [Development](./development.md) - Developer guide and standards
- [Decisions](./decisions.md) - Architecture decisions and rationale

---

## Quick Navigation

**New to the project?** Read in this order:
1. **[overview.md](./overview.md)** - Understand what and why
2. **[development.md](./development.md)** - Get started locally
3. **[architecture.md](./architecture.md)** - Understand how it works
4. **[operations.md](./operations.md)** - Deploy and monitor

**By role:**
- **Product/Business:** [overview.md](./overview.md)
- **Engineers (new):** [overview.md](./overview.md) → [development.md](./development.md)
- **Engineers (existing):** [architecture.md](./architecture.md) → [decisions.md](./decisions.md)
- **DevOps/SRE:** [architecture.md](./architecture.md) → [operations.md](./operations.md)
- **Architects/Tech Leads:** All documents

---

## Project Status

| Aspect | Status | Details |
|--------|--------|---------|
| **Environment** | [Production/Staging/Development] | [URL or status] |
| **Version** | [v1.0.0] | [Release date] |
| **Team** | [Team name] | [Slack channel or contact] |
| **Documentation** | [Up to date/Needs review] | Last updated: [Date] |

---

## One-Minute Overview

**What:** [One sentence describing what this project does]

**Why:** [One sentence explaining the business value]

**Who:** [Primary users/stakeholders]

**Tech:** [Primary technology stack - e.g., Python/FastAPI, PostgreSQL, React]

For complete details, see [overview.md](./overview.md).

---

## Quick Start

```bash
# Clone the repository
git clone [repository-url]

# Install dependencies
[installation commands]

# Run locally
[run commands]
```

For detailed setup, see [development.md](./development.md).

---

## Documentation Structure

This repository follows a **single source of truth** documentation pattern:

```
/docs
├── README.md           # This file - navigation and quick reference
├── overview.md         # What, why, who (business context + tech stack)
├── architecture.md     # How it works (system design, APIs, data)
├── operations.md       # How to run it (deployment, monitoring, incidents)
├── development.md      # How to build it (setup, standards, testing)
├── decisions.md        # Why we built it this way (ADR index)
└── adr-YYYYMMDD-*.md   # Individual architecture decisions
```

Each document is self-contained and comprehensive - no need to jump between files.

---

## Contributing

1. Read [development.md](./development.md) for coding standards
2. Create a feature branch
3. Submit a PR following the guidelines in [development.md](./development.md)
4. Significant architectural changes require an ADR (see [decisions.md](./decisions.md))

---

## Support & Contact

- **Team:** [Team name]
- **Slack:** [Channel]
- **On-call:** [Rotation link or contact]
- **Repository:** [GitHub/GitLab URL]

---

## License

[License information]

---

**Last Updated:** [Date]  
**Maintained By:** [Team/Person]
