---
title: "Database Integration"
description: "MariaDB configuration, schema, and operations for Firefly III."
tags: [database, mariadb, mysql, firefly-iii]
search: { boost: 3, exclude: false }
icon: material/database
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Document database integration and operations.

**Contents**
- [Database Requirements](#database-requirements)
- [Container Interface](#container-interface)
- [Schema Management](#schema-management)
- [Sources](#sources)

### Database Requirements

Firefly III requires a relational database for persistent storage:

- **Supported**: MySQL 8.0+, MariaDB 10.3+, PostgreSQL 12+
- **Development Only**: SQLite (not recommended for production)
- **Character Set**: utf8mb4 with utf8mb4_unicode_ci collation
- **Storage Engine**: InnoDB (for transactions and foreign keys)
- **Timezone**: UTC recommended for consistency

### Container Interface

| Aspect | Value / Path | Notes |
|-------|---------------|-------|
| Image | webgrip/firefly-iii-application-mariadb:latest | Org-owned image based on official MariaDB |
| Ports | 3306:3306 | Standard MySQL/MariaDB port |
| Healthcheck | `mariadb --execute="SELECT 1"` | Validates database connectivity |
| Volumes | `/var/lib/mysql` | Persistent data storage |
| Character Set | utf8mb4 | Required for emoji and international text |
| Collation | utf8mb4_unicode_ci | Case-insensitive Unicode collation |
| Storage Engine | InnoDB | Supports transactions and foreign keys |

### Schema Management

**Migration Process:**
- Firefly III uses Laravel migrations for schema versioning
- Migrations run automatically on container startup
- Database schema is maintained by upstream Firefly III releases

**Key Tables:**
- `accounts`: Financial accounts (asset, expense, revenue, liability)
- `transactions`: Individual money movements  
- `transaction_journals`: Containers for balanced transaction sets
- `budgets`: Budget definitions and limits
- `bills`: Recurring expense tracking
- `rules`: Automation rules and triggers
- `categories`: Transaction categorization
- `users`: User accounts and authentication

**Backup Considerations:**
- Regular dumps recommended (daily for production)
- Include both schema and data in backups
- Test restore procedures regularly
- Consider point-in-time recovery for critical deployments

**Performance:**
- Indexes automatically managed by migrations
- Regular `ANALYZE TABLE` recommended for query optimization
- Monitor slow query log for performance issues
- Consider read replicas for reporting workloads

### Sources
- "Firefly III Database Requirements" — https://docs.firefly-iii.org/explanation/more-information/database — retrieved 2025-01-09
- "MariaDB Official Docker Image" — https://hub.docker.com/_/mariadb — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/explanation/more-information/database":"sha256:pending","https://hub.docker.com/_/mariadb":"sha256:pending"},"sections":{"database-integration":"sha256:pending"}}}
-->