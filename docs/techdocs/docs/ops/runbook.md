---
title: "Runbook"
description: "Routine operations: start/stop, backups, restores, health."
tags: [operations, runbook]
search: { boost: 4, exclude: false }
icon: material/clipboard-text
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Safe day-2 operations.

**Contents**
- [Start/Stop](#startstop)
- [Backups/Restores](#backupsrestores)
- [Health & Readiness](#health--readiness)
- [Maintenance Tasks](#maintenance-tasks)
- [Sources](#sources)

## Start/Stop

**Starting the stack:**
```bash
# Bootstrap environment (first time only)
make bootstrap

# Start all services
make start

# Verify services are running
docker ps | grep firefly-iii-application

# Check logs for any startup issues
make logs
```

**Stopping the stack:**
```bash
# Graceful shutdown
make stop

# Force stop if needed (last resort)
docker compose down --remove-orphans
```

**Individual service management:**
```bash
# Restart specific service
docker compose restart firefly-iii-application.application

# View logs for specific service  
make logs SERVICE=firefly-iii-application.application

# Execute commands in application container
make enter CMD="/bin/bash"
```

## Backups/Restores

**Database backup (daily recommended):**
```bash
# Create timestamped backup
DATE=$(date +%Y%m%d_%H%M%S)
docker exec firefly-iii-application.mariadb mysqldump \
  -u application -p'${DB_PASSWORD}' application \
  > "backup_firefly_${DATE}.sql"

# Compress backup
gzip "backup_firefly_${DATE}.sql"

# Verify backup integrity
zcat "backup_firefly_${DATE}.sql.gz" | head -20
```

**File storage backup:**
```bash
# Backup user uploads and application storage
docker run --rm \
  -v firefly-iii-application-application-storage-data:/source:ro \
  -v $(pwd):/backup \
  alpine:latest \
  tar czf /backup/storage_backup_$(date +%Y%m%d).tar.gz -C /source .
```

**Database restore:**
```bash
# Stop application first
docker compose stop firefly-iii-application.application

# Restore database
zcat backup_firefly_YYYYMMDD_HHMMSS.sql.gz | \
  docker exec -i firefly-iii-application.mariadb mysql \
  -u application -p'${DB_PASSWORD}' application

# Restart application
docker compose start firefly-iii-application.application
```

**Storage restore:**
```bash
# Stop application
docker compose stop firefly-iii-application.application

# Restore files
docker run --rm \
  -v firefly-iii-application-application-storage-data:/target \
  -v $(pwd):/backup:ro \
  alpine:latest \
  tar xzf /backup/storage_backup_YYYYMMDD.tar.gz -C /target

# Restart application
docker compose start firefly-iii-application.application
```

## Health & Readiness

**Health check endpoints:**
- **Application**: `http://localhost:8080/health` (via nginx)
- **Direct PHP-FPM**: Not exposed externally
- **Database**: Health check via container status
- **Redis**: Health check via container status

**Container health verification:**
```bash
# Check all container health status
docker ps --format "table {{ '{' }}.Names{{ '}' }}\t{{ '{' }}.Status{{ '}' }}\t{{ '{' }}.Ports{{ '}' }}"

# Detailed health check
docker inspect firefly-iii-application.application | grep -A 5 Health

# Manual health check
curl -f http://localhost:8080/health || echo "Health check failed"
```

**Health check commands:**
```bash
# Application health (internal)
docker exec firefly-iii-application.application \
  php artisan firefly-iii:verify

# Database connectivity
docker exec firefly-iii-application.mariadb \
  mariadb --user=application --password='${DB_PASSWORD}' \
  --execute="SELECT 1"

# Redis connectivity  
docker exec firefly-iii-application.redis redis-cli ping
```

**Expected health check responses:**
- **HTTP 200**: Application healthy and database connected
- **HTTP 503**: Application starting or database unavailable
- **Connection refused**: Service not running or network issue

## Maintenance Tasks

**Cache clearing (if performance issues):**
```bash
# Clear application cache
docker exec firefly-iii-application.application php artisan cache:clear

# Clear Redis cache entirely (destructive)
docker exec firefly-iii-application.redis redis-cli FLUSHALL
```

**Database maintenance:**
```bash
# Check database size
docker exec firefly-iii-application.mariadb \
  mysql -u application -p'${DB_PASSWORD}' \
  -e "SELECT table_name AS 'Table', 
      ROUND(((data_length + index_length) / 1024 / 1024), 2) AS 'Size (MB)' 
      FROM information_schema.TABLES 
      WHERE table_schema = 'application' 
      ORDER BY (data_length + index_length) DESC;"

# Optimize database tables
docker exec firefly-iii-application.mariadb \
  mysql -u application -p'${DB_PASSWORD}' \
  -e "OPTIMIZE TABLE application.transactions, application.accounts, application.transaction_journals;"
```

**Log rotation:**
```bash
# Check log sizes
docker exec firefly-iii-application.application \
  du -sh storage/logs/

# Archive old logs (keep last 7 days)
docker exec firefly-iii-application.application \
  find storage/logs/ -name "*.log" -mtime +7 -delete
```

**Security updates:**
```bash
# Check for container image updates
docker compose pull

# Update and restart (after testing in staging)
docker compose down
docker compose up -d
```

**User management:**
```bash
# Create admin user
make user:create EMAIL=admin@example.com PASS=secure_password

# Reset user password (Laravel tinker)
docker exec -it firefly-iii-application.application php artisan tinker
>>> $user = User::where('email', 'admin@example.com')->first();
>>> $user->password = Hash::make('new_password');
>>> $user->save();
```

## Monitoring & Alerts

**Key metrics to monitor:**
- Container health status
- HTTP response times and error rates
- Database connection pool utilization
- Redis memory usage and hit rates
- Disk space for volumes
- Transaction processing rates

**Log monitoring:**
```bash
# Watch for errors in real-time
make logs | grep -i error

# Check for specific error patterns
docker logs firefly-iii-application.application 2>&1 | grep -i "exception\|error\|failed"
```

**Performance monitoring:**
```bash
# Check resource usage
docker stats firefly-iii-application.application

# Database performance
docker exec firefly-iii-application.mariadb \
  mysql -u application -p'${DB_PASSWORD}' \
  -e "SHOW PROCESSLIST;"
```

## Emergency Procedures

**Application unresponsive:**
1. Check container health: `docker ps`
2. Review logs: `make logs`
3. Restart application: `docker compose restart firefly-iii-application.application`
4. If database issues, restart database: `docker compose restart firefly-iii-application.mariadb`

**Database corruption:**
1. Stop application immediately
2. Restore from latest backup
3. Verify data integrity
4. Restart application

**Storage full:**
1. Archive old logs and backups
2. Clean up temporary files
3. Extend storage if needed
4. Monitor space regularly

## Sources
- "Firefly III Installation Guide" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09
- "Firefly III Administration" — https://docs.firefly-iii.org/how-to/firefly-iii/administration/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/":"","https://docs.firefly-iii.org/how-to/firefly-iii/administration/":""},"sections":{"runbook":""}}}
-->