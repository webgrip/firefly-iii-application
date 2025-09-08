---
title: "Files & Paths"
description: "Config files, data directories, and persistence responsibilities."
tags: [configuration, filesystem]
search: { boost: 3, exclude: false }
icon: material/folder-cog
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Make storage locations and ownership explicit.

**Contents**
- [Config files](#config-files)
- [Data directories & volumes](#data-directories--volumes)
- [Certificates & secrets](#certificates--secrets)
- [Sources](#sources)

## Config files

| Path | Purpose | Format | Notes | Source |
|------|---------|--------|-------|--------|
| `/var/www/app/.env` | Main configuration file | Key=Value | Contains all environment variables | "Firefly III Installation" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `/var/www/app/config/` | Laravel configuration directory | PHP | Framework configuration files | "Firefly III Installation" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |

## Data directories & volumes

| Path | What it stores | Backup/Retention | Notes | Source |
|------|-----------------|------------------|-------|--------|
| `/var/www/app/storage/` | Application storage (logs, cache, uploads) | Regular backup recommended | Contains file uploads, logs, compiled views | "Firefly III Installation" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `/var/www/app/storage/app/` | User uploaded files | **Critical - must backup** | Attachment files, import files | "Firefly III Installation" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `/var/www/app/storage/logs/` | Application logs | Rotate/archive weekly | Laravel and Firefly III logs | "Firefly III Installation" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `/var/www/app/public/` | Public web assets | No backup needed | Static assets (CSS, JS, images) | "Firefly III Installation" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `/var/lib/mysql/` | Database files (MariaDB) | **Critical - must backup** | All financial data | "MariaDB Documentation" — https://mariadb.com/kb/en/data-directories/ — retrieved 2025-01-09 |
| `/data/` | Redis persistence (if enabled) | Optional backup | Cache and session data | "Redis Documentation" — https://redis.io/docs/manual/persistence/ — retrieved 2025-01-09 |

## Certificates & secrets

| Purpose | Location | Management | Notes |
|---------|----------|------------|-------|
| TLS Certificates | External load balancer | Let's Encrypt recommended | Managed outside container |
| Database passwords | Environment variables | Kubernetes secrets/Docker secrets | Never in image layers |
| Application key | `APP_KEY` environment variable | Generated with `php artisan key:generate` | Critical for encryption |

## File permissions

- **Application files**: Owned by `www-data:www-data` (uid 33)
- **Storage directory**: Must be writable by web server
- **Database files**: Owned by `mysql:mysql` (uid 999)
- **Redis files**: Owned by `redis:redis` (uid 100)

## Backup strategy

**Critical data (daily backups):**
- Database (full dump with `mysqldump` or `pg_dump`)
- Storage directory (`/var/www/app/storage/app/`)

**Optional data:**
- Logs (for debugging, can be regenerated)
- Cache (temporary, can be regenerated)

## Sources
- "Firefly III Installation Guide" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09
- "MariaDB Data Directory" — https://mariadb.com/kb/en/data-directories/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/":"","https://mariadb.com/kb/en/data-directories/":""},"sections":{"volumes":""}}}
-->