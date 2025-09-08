---
title: "Files & Paths"
description: "Config files, data directories, and persistence responsibilities."
tags: [configuration, filesystem, firefly-iii]
search: { boost: 3, exclude: false }
icon: material/folder-cog
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Make storage locations and ownership explicit.

**Contents**
- [Config files](#config-files)
- [Data directories & volumes](#data-directories--volumes)
- [Certificates & secrets](#certificates--secrets)
- [Sources](#sources)

### Config files

| Path | Purpose | Format | Notes | Source |
|------|---------|--------|-------|--------|
| `/var/www/app/.env` | Application configuration | Key=Value | Laravel environment file | "Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |
| `/var/www/app/config/` | PHP configuration files | PHP arrays | Laravel config cache location | "Laravel Configuration" — https://laravel.com/docs/10.x/configuration — retrieved 2025-01-09 |
| `/var/www/app/storage/logs/` | Application logs | JSON/Text | Configurable via LOG_CHANNEL | "Logging" — https://docs.firefly-iii.org/explanation/more-information/logs — retrieved 2025-01-09 |

### Data directories & volumes

| Path | What it stores | Backup/Retention | Notes | Source |
|------|-----------------|------------------|-------|--------|
| `/var/www/app/storage/app/` | User uploads, attachments, exports | Daily backup, 7 year retention | Financial document storage | "File Storage" — https://docs.firefly-iii.org/how-to/firefly-iii/features/attachments — retrieved 2025-01-09 |
| `/var/www/app/storage/logs/` | Application log files | Weekly rotation, 90 day retention | Debug and audit information | "Logging" — https://docs.firefly-iii.org/explanation/more-information/logs — retrieved 2025-01-09 |
| `/var/www/app/storage/framework/cache/` | Application cache files | No backup needed | Temporary performance data | "Caching" — https://laravel.com/docs/10.x/cache — retrieved 2025-01-09 |
| `/var/www/app/storage/framework/sessions/` | User session data | No backup needed | Temporary session storage (when using file driver) | "Sessions" — https://laravel.com/docs/10.x/session — retrieved 2025-01-09 |
| `/var/www/app/public/` | Web-accessible files | No backup needed | Static assets and compiled frontend | "Public Assets" — https://docs.firefly-iii.org/explanation/more-information/architecture — retrieved 2025-01-09 |
| `/var/www/app/bootstrap/cache/` | Configuration cache | No backup needed | Laravel optimization cache | "Optimization" — https://laravel.com/docs/10.x/deployment — retrieved 2025-01-09 |

### Certificates & secrets

| Path | Purpose | Security Level | Notes | Source |
|------|---------|---------------|-------|--------|
| `/var/www/app/.env` | Environment variables including secrets | High | Contains database passwords, API keys | "Environment Security" — https://docs.firefly-iii.org/explanation/more-information/security — retrieved 2025-01-09 |
| `/etc/ssl/certs/` | System SSL certificates | Medium | For HTTPS connections to external services | "SSL Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |

**File Ownership & Permissions:**
- Application files: `www-data:www-data` (33:33)
- Storage directories: `755` permissions
- Log files: `644` permissions  
- Configuration files: `600` permissions (sensitive)
- Cache directories: `755` permissions

**Backup Strategy:**
- **Critical**: `/var/www/app/storage/app/` (user data, attachments)
- **Important**: Application logs for compliance/audit
- **Skip**: Cache and session directories (regenerated)

**Volume Mapping:**
```yaml
volumes:
  - firefly-iii-application-storage-data:/var/www/app/storage
  - firefly-iii-application-public-data:/var/www/app/public
```

**Container Persistence:**
- Storage data: Persisted across container restarts
- Public assets: Shared with Nginx container (read-only)
- Database: Separate MariaDB volume
- Cache: Redis container or file-based

### Sources
- "Firefly III Docker Installation" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09
- "Laravel Filesystem" — https://laravel.com/docs/10.x/filesystem — retrieved 2025-01-09
- "Docker Storage Best Practices" — https://docs.docker.com/storage/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker":"sha256:pending","https://laravel.com/docs/10.x/filesystem":"sha256:pending","https://docs.docker.com/storage/":"sha256:pending"},"sections":{"volumes":"sha256:pending"}}}
-->