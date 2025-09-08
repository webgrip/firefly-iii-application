---
title: "Database Integration"
description: "Database requirements, supported backends, and connection configuration."
tags: [database, integration, mariadb, mysql, postgresql]
search: { boost: 3, exclude: false }
icon: material/database
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Document database integration requirements and configuration.

**Contents**
- [Supported databases](#supported-databases)
- [Connection configuration](#connection-configuration)
- [Container interface](#container-interface)
- [Migration and maintenance](#migration-and-maintenance)
- [Sources](#sources)

## Supported databases

Firefly III supports multiple database backends:

| Database | Version | Notes | Source |
|----------|---------|-------|--------|
| MySQL | 8.0+ | Recommended for new installations | "Firefly III Requirements" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| MariaDB | 10.9+ | Used in this repository deployment | "Firefly III Requirements" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| PostgreSQL | 13+ | Alternative to MySQL/MariaDB | "Firefly III Requirements" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| SQLite | 3.8+ | Development only, not recommended for production | "Firefly III Requirements" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |

**Character Set Requirements:**
- MySQL/MariaDB: `utf8mb4` character set with `utf8mb4_unicode_ci` collation
- PostgreSQL: UTF-8 encoding

## Connection configuration

**Environment variables for database connection:**

| Variable | Purpose | Example |
|----------|---------|---------|
| `DB_CONNECTION` | Database driver | mysql, pgsql, sqlite |
| `DB_HOST` | Database hostname | firefly-iii-application.mariadb |
| `DB_PORT` | Database port | 3306 (MySQL/MariaDB), 5432 (PostgreSQL) |
| `DB_DATABASE` | Database name | firefly-iii-application |
| `DB_USERNAME` | Database user | firefly-iii-application |
| `DB_PASSWORD` | Database password | (from secrets) |

**Connection string format (internal use):**
```
mysql://username:password@host:port/database?charset=utf8mb4
```

## Container interface

| Aspect | Value / Path | Notes |
|--------|---------------|-------|
| Ports | 3306/tcp (MariaDB) | Must match upstream defaults |
| Healthcheck | `mariadb --user=<user> --password=<pass> --execute="SELECT 1"` | As per official image guidance |
| Volumes | `/var/lib/mysql` | Data and retention responsibilities |
| Character Set | `utf8mb4` with `utf8mb4_unicode_ci` collation | Required for proper emoji/Unicode support |
| Configuration | `/etc/mysql/mariadb.conf.d/` | Custom configuration files |

**Docker Compose configuration:**
```yaml
firefly-iii-application.mariadb:
  image: webgrip/firefly-iii-application-mariadb:latest
  environment:
    MARIADB_DATABASE: ${DB_DATABASE:-application}
    MARIADB_USER: ${DB_USERNAME:-application} 
    MARIADB_PASSWORD: ${DB_PASSWORD:-application}
    MARIADB_ROOT_PASSWORD: ${DB_ROOT_PASSWORD:-root}
    MARIADB_CHARACTER_SET_SERVER: utf8mb4
    MARIADB_COLLATION_SERVER: utf8mb4_unicode_ci
  healthcheck:
    test: ["CMD", "mariadb", "--user=${DB_USERNAME:-application}", "--password=${DB_PASSWORD:-application}", "--execute=SELECT 1"]
    interval: 10s
    timeout: 5s
    retries: 10
    start_period: 30s
```

## Migration and maintenance

**Database initialization:**
1. Container starts with empty database
2. Firefly III runs migrations automatically on first startup
3. Subsequent startups check for pending migrations

**Backup procedures:**
```bash
# Full database backup
docker exec firefly-iii-application.mariadb mysqldump \
  -u application -p application > firefly_backup.sql

# Restore from backup
docker exec -i firefly-iii-application.mariadb mysql \
  -u application -p application < firefly_backup.sql
```

**Migration commands:**
```bash
# Manual migration (if needed)
docker exec firefly-iii-application.application \
  php artisan migrate

# Check migration status
docker exec firefly-iii-application.application \
  php artisan migrate:status
```

**Performance tuning (MariaDB):**
- `innodb_buffer_pool_size`: 70-80% of available RAM
- `max_connections`: Based on expected concurrent users
- `innodb_log_file_size`: 256MB for write-heavy workloads

## Sources
- "Firefly III Installation Requirements" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09
- "MariaDB Official Image" — https://hub.docker.com/_/mariadb — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/":"","https://hub.docker.com/_/mariadb":""},"sections":{"database-integration":""}}}
-->