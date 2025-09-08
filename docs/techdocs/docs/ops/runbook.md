---
title: "Runbook"
description: "Routine operations: start/stop, backups, restores, health."
tags: [operations, runbook, firefly-iii]
search: { boost: 4, exclude: false }
icon: material/clipboard-text
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Safe day-2 operations for Firefly III deployment.

**Contents**
- [Start/Stop](#startstop)
- [Backups/Restores](#backupsrestores)
- [Health & Readiness](#health--readiness)
- [User Management](#user-management)
- [Sources](#sources)

### Start/Stop

**Start Services:**
```bash
# Start all services with health checks
make start

# Follow logs during startup
make logs

# Wait for application readiness (optional)
make wait-ready URL=http://localhost:8080/login
```

**Stop Services:**
```bash
# Graceful shutdown
make stop

# Force stop and cleanup volumes (data loss!)
docker compose down --volumes
```

**Service Dependencies:**
1. MariaDB starts first (30s startup period)
2. Redis starts second (5s startup period)  
3. Application waits for DB + Redis healthy
4. Nginx starts after application is ready

### Backups/Restores

**Database Backup:**
```bash
# Create backup
docker compose exec firefly-iii-application.mariadb mysqldump \
  -u application -papplication application > backup-$(date +%Y%m%d).sql

# Automated backup (add to crontab)
0 2 * * * cd /path/to/repo && make backup
```

**File Storage Backup:**
```bash
# Backup uploaded files and storage
docker run --rm -v firefly-iii-application-application-storage-data:/data \
  -v $(pwd):/backup alpine tar czf /backup/storage-$(date +%Y%m%d).tar.gz /data
```

**Restore Database:**
```bash
# Stop application first
docker compose stop firefly-iii-application.application

# Restore database
docker compose exec firefly-iii-application.mariadb mysql \
  -u application -papplication application < backup-20250109.sql

# Restart application
docker compose start firefly-iii-application.application
```

### Health & Readiness

**Application Health Checks:**
- Healthcheck endpoint: `http://localhost:8080/health` (returns 200 when ready)
- Container health: `docker ps` shows "healthy" status
- Database connectivity: Automatic Laravel health check
- Redis connectivity: Session storage functional test

**Health Check Commands:**
```bash
# Quick health check
curl -f http://localhost:8080/health || echo "Service unhealthy"

# Detailed container status
docker compose ps

# Application logs for errors
docker compose logs firefly-iii-application.application | grep -i error
```

**Readiness Criteria:**
- HTTP 200 response from `/health` endpoint
- Database migrations completed successfully
- Redis connection established
- File permissions correct for storage directories
- All environment variables properly loaded

**Timeout Guidance:**
- Initial startup: Allow 60-90 seconds for full readiness
- Health check interval: 30 seconds
- Database timeout: 10 seconds
- Redis timeout: 5 seconds

### User Management

**Create Initial Admin User:**
```bash
# Create admin user (customize EMAIL and PASS)
make user:create EMAIL=admin@example.com PASS=securepassword123

# Or directly via artisan
docker compose exec firefly-iii-application.application \
  php artisan firefly-iii:create-user --email=admin@example.com
```

**Reset User Password:**
```bash
# Reset password via command line
docker compose exec firefly-iii-application.application \
  php artisan firefly-iii:reset-password --email=user@example.com
```

### Sources
- "Firefly III Docker Installation" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09
- "Firefly III Maintenance" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/upgrade — retrieved 2025-01-09
- "Firefly III Command Line" — https://docs.firefly-iii.org/references/firefly-iii/command-line — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker":"sha256:pending","https://docs.firefly-iii.org/how-to/firefly-iii/installation/upgrade":"sha256:pending","https://docs.firefly-iii.org/references/firefly-iii/command-line":"sha256:pending"},"sections":{"runbook":"sha256:pending"}}}
-->