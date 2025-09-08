---
title: "Runbook"
description: "Routine operations: start/stop, backups, restores, health checks."
tags: [operations, runbook, procedures]
search: { boost: 4, exclude: false }
icon: material/clipboard-text
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Day-to-day operational procedures for managing the Invoice Ninja deployment.

**Contents**
- [Daily Operations](#daily-operations)
- [System Startup and Shutdown](#system-startup-and-shutdown)
- [Health Checks](#health-checks)
- [Backup Procedures](#backup-procedures)
- [Restore Procedures](#restore-procedures)
- [User Management](#user-management)
- [Maintenance Tasks](#maintenance-tasks)
- [Emergency Procedures](#emergency-procedures)
- [Sources](#sources)

## Daily Operations

### Morning Checklist
Start-of-day verification procedures:

```bash
# 1. Check overall system health
make logs | grep -i error | tail -20

# 2. Verify all services are running
docker ps --filter "name=firefly-iii-application" --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# 3. Check application accessibility
make wait-ready URL=http://localhost:8080

# 4. Verify database connectivity
make run CMD="php artisan tinker --execute='DB::connection()->getPdo(); echo \"DB OK\";'"

# 5. Check queue processing
make run CMD="php artisan queue:stats"

# 6. Review disk space
df -h | grep -E "(docker|var|tmp)"
```

### End-of-Day Checklist
End-of-day maintenance and verification:

```bash
# 1. Process any pending queue jobs
make run CMD="php artisan queue:work redis --once --timeout=300"

# 2. Clear expired cache entries
make run CMD="php artisan cache:prune-stale-tags"

# 3. Check for failed jobs
make run CMD="php artisan queue:failed"

# 4. Review error logs
make logs SERVICE=firefly-iii-application.application | grep -i error | tail -10

# 5. Verify backup completion (if automated)
ls -la /backup/location/ | tail -5
```

## System Startup and Shutdown

### Complete System Startup
Procedure for starting the entire Invoice Ninja stack:

```bash
# 1. Ensure Docker network exists
docker network ls | grep webgrip || docker network create webgrip

# 2. Start all services
make start

# 3. Wait for services to be healthy
echo "Waiting for services to start..."
sleep 30

# 4. Verify database is ready
make run CMD="php artisan migrate:status"

# 5. Check cache connectivity
make run CMD="php artisan tinker --execute='Cache::put(\"health\", \"ok\", 60); echo Cache::get(\"health\");'"

# 6. Verify application is accessible
make wait-ready URL=http://localhost:8080

# 7. Start queue workers (if not automated)
# make run CMD="php artisan queue:work redis" &

echo "✅ System startup complete"
```

### Graceful System Shutdown
Procedure for safely shutting down the system:

```bash
# 1. Stop accepting new requests (if load balancer available)
# curl -X POST http://load-balancer/maintenance-mode

# 2. Wait for queue jobs to complete
echo "Waiting for queue jobs to complete..."
make run CMD="php artisan queue:restart"
sleep 60

# 3. Put application in maintenance mode
make run CMD="php artisan down --message='Scheduled maintenance'"

# 4. Stop application containers
make stop

# 5. Verify all containers are stopped
docker ps --filter "name=firefly-iii-application" --format "table {{.Names}}\t{{.Status}}"

echo "✅ System shutdown complete"
```

### Service-Specific Operations
Individual service management:

```bash
# Restart specific services
docker compose restart firefly-iii-application.application
docker compose restart firefly-iii-application.nginx
docker compose restart firefly-iii-application.mariadb
docker compose restart firefly-iii-application.redis

# Check individual service health
docker compose exec firefly-iii-application.mariadb mysqladmin ping
docker compose exec firefly-iii-application.redis redis-cli ping
curl -s http://localhost:8080/health
```

## Health Checks

### Application Health Verification
Comprehensive health check procedures:

```bash
#!/bin/bash
# health-check.sh - Comprehensive system health check

echo "🔍 Invoice Ninja Health Check - $(date)"
echo "================================================"

# 1. Container Health
echo "📦 Container Status:"
docker ps --filter "name=firefly-iii-application" --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
echo

# 2. Application Connectivity
echo "🌐 Application Connectivity:"
if curl -s -o /dev/null -w "%{http_code}" http://localhost:8080 | grep -q "200"; then
    echo "✅ Web application: OK"
else
    echo "❌ Web application: FAILED"
fi

# 3. Database Health
echo "🗄️  Database Health:"
if make run CMD="php artisan tinker --execute='DB::connection()->getPdo(); echo \"OK\";'" 2>/dev/null | grep -q "OK"; then
    echo "✅ Database connection: OK"
else
    echo "❌ Database connection: FAILED"
fi

# 4. Cache Health
echo "💾 Cache Health:"
if make run CMD="php artisan tinker --execute='Cache::put(\"test\", \"ok\", 10); echo Cache::get(\"test\");'" 2>/dev/null | grep -q "ok"; then
    echo "✅ Redis cache: OK"
else
    echo "❌ Redis cache: FAILED"
fi

# 5. Queue Health
echo "📋 Queue Health:"
QUEUE_SIZE=$(make run CMD="php artisan queue:stats" 2>/dev/null | grep -o "[0-9]\+ pending" | head -1 | cut -d' ' -f1)
if [[ $QUEUE_SIZE -lt 100 ]]; then
    echo "✅ Queue size: $QUEUE_SIZE (healthy)"
else
    echo "⚠️  Queue size: $QUEUE_SIZE (check processing)"
fi

# 6. Disk Space
echo "💿 Disk Space:"
DISK_USAGE=$(df -h | grep -E "(docker|var)" | awk '{print $5}' | sed 's/%//' | sort -n | tail -1)
if [[ $DISK_USAGE -lt 80 ]]; then
    echo "✅ Disk usage: ${DISK_USAGE}% (healthy)"
else
    echo "⚠️  Disk usage: ${DISK_USAGE}% (monitor closely)"
fi

# 7. Memory Usage
echo "🧠 Memory Usage:"
MEMORY_USAGE=$(docker stats --no-stream --format "table {{.Container}}\t{{.MemUsage}}" | grep firefly-iii-application)
echo "$MEMORY_USAGE"

echo "================================================"
echo "Health check completed - $(date)"
```

### Automated Health Monitoring
Set up automated health monitoring:

```bash
# Create health check cron job
# Add to crontab: 0 */6 * * * /path/to/health-check.sh >> /var/log/health-check.log 2>&1

# Health check with alerting
#!/bin/bash
if ! ./health-check.sh | grep -q "❌"; then
    echo "All health checks passed"
else
    echo "Health check failures detected - sending alert"
    # Send alert (email, Slack, etc.)
    # curl -X POST -H 'Content-type: application/json' \
    #   --data '{"text":"Invoice Ninja health check failed"}' \
    #   $SLACK_WEBHOOK_URL
fi
```

## Backup Procedures

### Database Backup
Complete database backup procedures:

```bash
#!/bin/bash
# database-backup.sh - Database backup script

BACKUP_DIR="/backup/database"
DATE=$(date +%Y%m%d_%H%M%S)
DB_HOST="firefly-iii-application.mariadb"
DB_NAME="application"
DB_USER="application"
DB_PASS="application"

echo "🗄️  Starting database backup - $(date)"

# Create backup directory
mkdir -p $BACKUP_DIR

# Full database backup
mysqldump \
  --host=$DB_HOST \
  --user=$DB_USER \
  --password=$DB_PASS \
  --single-transaction \
  --routines \
  --triggers \
  --add-drop-table \
  --compress \
  $DB_NAME > $BACKUP_DIR/invoice_ninja_db_$DATE.sql

# Verify backup
if [ -f "$BACKUP_DIR/invoice_ninja_db_$DATE.sql" ] && [ -s "$BACKUP_DIR/invoice_ninja_db_$DATE.sql" ]; then
    echo "✅ Database backup successful: invoice_ninja_db_$DATE.sql"
    
    # Compress backup
    gzip $BACKUP_DIR/invoice_ninja_db_$DATE.sql
    
    # Clean up old backups (keep last 7 days)
    find $BACKUP_DIR -name "invoice_ninja_db_*.sql.gz" -mtime +7 -delete
    
else
    echo "❌ Database backup failed"
    exit 1
fi

echo "Database backup completed - $(date)"
```

### File System Backup
Application files and uploads backup:

```bash
#!/bin/bash
# files-backup.sh - Application files backup

BACKUP_DIR="/backup/files"
DATE=$(date +%Y%m%d_%H%M%S)

echo "📁 Starting files backup - $(date)"

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup application volumes
docker run --rm \
  -v firefly-iii-application-application-public-data:/source/public:ro \
  -v firefly-iii-application-application-storage-data:/source/storage:ro \
  -v $BACKUP_DIR:/backup \
  alpine:latest \
  tar czf /backup/invoice_ninja_files_$DATE.tar.gz -C /source .

# Backup environment configuration
cp .env $BACKUP_DIR/env_$DATE.backup

# Verify backup
if [ -f "$BACKUP_DIR/invoice_ninja_files_$DATE.tar.gz" ]; then
    echo "✅ Files backup successful: invoice_ninja_files_$DATE.tar.gz"
    
    # Clean up old backups (keep last 7 days)
    find $BACKUP_DIR -name "invoice_ninja_files_*.tar.gz" -mtime +7 -delete
    find $BACKUP_DIR -name "env_*.backup" -mtime +7 -delete
    
else
    echo "❌ Files backup failed"
    exit 1
fi

echo "Files backup completed - $(date)"
```

### Complete Backup Script
Combined backup procedure:

```bash
#!/bin/bash
# full-backup.sh - Complete system backup

BACKUP_BASE="/backup"
DATE=$(date +%Y%m%d_%H%M%S)

echo "🔄 Starting complete backup - $(date)"

# Run database backup
./database-backup.sh

# Run files backup  
./files-backup.sh

# Create backup manifest
cat > $BACKUP_BASE/backup_manifest_$DATE.txt << EOF
Invoice Ninja Backup Manifest
============================
Date: $(date)
System: $(hostname)

Database Backup: $(ls -la $BACKUP_BASE/database/invoice_ninja_db_*$DATE*.sql.gz 2>/dev/null || echo "FAILED")
Files Backup: $(ls -la $BACKUP_BASE/files/invoice_ninja_files_*$DATE*.tar.gz 2>/dev/null || echo "FAILED")
Environment: $(ls -la $BACKUP_BASE/files/env_*$DATE*.backup 2>/dev/null || echo "FAILED")

Docker Images:
$(docker images | grep firefly-iii-application)

Volume Information:
$(docker volume ls | grep firefly-iii-application)
EOF

echo "✅ Complete backup finished - $(date)"
```

## Restore Procedures

### Database Restore
Restore database from backup:

```bash
#!/bin/bash
# database-restore.sh - Database restore procedure

BACKUP_FILE=$1
DB_HOST="firefly-iii-application.mariadb"
DB_NAME="application"
DB_USER="application"
DB_PASS="application"

if [ -z "$BACKUP_FILE" ]; then
    echo "Usage: $0 <backup_file.sql.gz>"
    echo "Available backups:"
    ls -la /backup/database/invoice_ninja_db_*.sql.gz
    exit 1
fi

echo "⚠️  WARNING: This will replace the current database!"
echo "Backup file: $BACKUP_FILE"
read -p "Continue? (yes/no): " CONFIRM

if [ "$CONFIRM" != "yes" ]; then
    echo "Restore cancelled"
    exit 0
fi

echo "🗄️  Starting database restore - $(date)"

# Stop application to prevent writes during restore
make run CMD="php artisan down --message='Database restore in progress'"

# Create current database backup before restore
echo "Creating safety backup..."
mysqldump \
  --host=$DB_HOST \
  --user=$DB_USER \
  --password=$DB_PASS \
  --single-transaction \
  $DB_NAME > /backup/safety_backup_$(date +%Y%m%d_%H%M%S).sql

# Restore from backup
echo "Restoring database..."
if [[ $BACKUP_FILE == *.gz ]]; then
    gunzip -c $BACKUP_FILE | mysql --host=$DB_HOST --user=$DB_USER --password=$DB_PASS $DB_NAME
else
    mysql --host=$DB_HOST --user=$DB_USER --password=$DB_PASS $DB_NAME < $BACKUP_FILE
fi

# Verify restore
if make run CMD="php artisan migrate:status" >/dev/null 2>&1; then
    echo "✅ Database restore successful"
    
    # Clear cache after restore
    make run CMD="php artisan cache:clear"
    make run CMD="php artisan config:clear"
    
    # Bring application back online
    make run CMD="php artisan up"
    
else
    echo "❌ Database restore failed - check logs"
    exit 1
fi

echo "Database restore completed - $(date)"
```

### File System Restore
Restore application files:

```bash
#!/bin/bash
# files-restore.sh - Files restore procedure

BACKUP_FILE=$1

if [ -z "$BACKUP_FILE" ]; then
    echo "Usage: $0 <backup_file.tar.gz>"
    echo "Available backups:"
    ls -la /backup/files/invoice_ninja_files_*.tar.gz
    exit 1
fi

echo "⚠️  WARNING: This will replace current application files!"
echo "Backup file: $BACKUP_FILE"
read -p "Continue? (yes/no): " CONFIRM

if [ "$CONFIRM" != "yes" ]; then
    echo "Restore cancelled"
    exit 0
fi

echo "📁 Starting files restore - $(date)"

# Stop application
make stop

# Restore files using temporary container
docker run --rm \
  -v firefly-iii-application-application-public-data:/target/public \
  -v firefly-iii-application-application-storage-data:/target/storage \
  -v $(dirname $BACKUP_FILE):/backup:ro \
  alpine:latest \
  sh -c "cd /target && tar xzf /backup/$(basename $BACKUP_FILE)"

if [ $? -eq 0 ]; then
    echo "✅ Files restore successful"
    
    # Restart application
    make start
    
    # Wait for startup
    sleep 30
    make wait-ready URL=http://localhost:8080
    
else
    echo "❌ Files restore failed"
    exit 1
fi

echo "Files restore completed - $(date)"
```

## User Management

### Create Admin User
Create administrative user account:

```bash
# Create admin user with make wrapper
make user:create EMAIL=admin@company.com PASS=secure_password

# Create user with custom attributes
make run CMD="php artisan ninja:create-account \
  --email=admin@company.com \
  --password=secure_password \
  --first-name=Admin \
  --last-name=User"

# Verify user creation
make run CMD="php artisan ninja:list-users"
```

### User Management Tasks
Common user management operations:

```bash
# Reset user password
make run CMD="php artisan ninja:reset-password --email=user@company.com"

# List all users
make run CMD="php artisan ninja:list-users"

# Create company
make run CMD="php artisan ninja:create-company --name='New Company' --email=contact@company.com"

# List companies
make run CMD="php artisan ninja:list-companies"

# Disable user account (via database)
make run CMD="php artisan tinker --execute='
  \$user = App\Models\User::where(\"email\", \"user@company.com\")->first();
  \$user->update([\"is_deleted\" => true]);
  echo \"User disabled\";
'"
```

## Maintenance Tasks

### Regular Maintenance Schedule
Scheduled maintenance tasks:

#### Daily Tasks
```bash
# Clear expired sessions and cache
make run CMD="php artisan cache:prune-stale-tags"

# Process failed queue jobs
make run CMD="php artisan queue:retry all"

# Check disk space
df -h | grep -E "(docker|var|tmp)"
```

#### Weekly Tasks
```bash
# Database optimization
make run CMD="php artisan ninja:optimize-db"

# Clear old log files
find /var/log -name "*.log" -mtime +7 -delete

# Update application cache
make run CMD="php artisan optimize"

# Backup verification
./health-check.sh
```

#### Monthly Tasks
```bash
# Full database backup
./full-backup.sh

# Security updates check
docker images | grep firefly-iii-application

# Performance review
make run CMD="php artisan ninja:performance-report"

# Cleanup old backups
find /backup -name "*.sql.gz" -mtime +30 -delete
find /backup -name "*.tar.gz" -mtime +30 -delete
```

### System Optimization
Performance optimization tasks:

```bash
# Clear all caches
make run CMD="php artisan optimize:clear"

# Rebuild optimized caches
make run CMD="php artisan optimize"

# Database table optimization
make run CMD="php artisan ninja:optimize-db"

# Queue processing optimization
make run CMD="php artisan queue:restart"
```

## Emergency Procedures

### Application Down
Emergency response for application outage:

```bash
# 1. Quick diagnostic
curl -I http://localhost:8080
docker ps | grep firefly-iii-application

# 2. Check logs for errors
make logs | grep -i error | tail -20

# 3. Restart services
make stop
make start

# 4. Verify recovery
make wait-ready URL=http://localhost:8080

# 5. Check data integrity
make run CMD="php artisan migrate:status"
```

### Database Issues
Emergency database recovery:

```bash
# 1. Check database connectivity
make run CMD="php artisan tinker --execute='DB::connection()->getPdo();'"

# 2. Restart database service
docker compose restart firefly-iii-application.mariadb

# 3. Check database integrity
make run CMD="php artisan ninja:check-db"

# 4. Emergency restore (if needed)
# ./database-restore.sh /backup/database/latest_backup.sql.gz
```

### Data Corruption
Emergency data recovery procedures:

```bash
# 1. Immediate shutdown
make run CMD="php artisan down --message='Emergency maintenance'"

# 2. Create emergency backup
mysqldump --all-databases > emergency_backup_$(date +%Y%m%d_%H%M%S).sql

# 3. Assess damage
make run CMD="php artisan ninja:check-db --verbose"

# 4. Restore from last known good backup
# ./database-restore.sh /backup/database/last_good_backup.sql.gz

# 5. Verify recovery and bring back online
make run CMD="php artisan up"
```

## Sources

- "Laravel Artisan Console" — https://laravel.com/docs/10.x/artisan — retrieved 2025-01-09
- "Docker Compose Operations" — https://docs.docker.com/compose/reference/ — retrieved 2025-01-09
- "MySQL Backup and Recovery" — https://dev.mysql.com/doc/refman/8.0/en/backup-and-recovery.html — retrieved 2025-01-09
- "Invoice Ninja Maintenance" — https://invoiceninja.github.io/en/self-host-maintenance/ — retrieved 2025-01-09
- "System Administration Best Practices" — https://www.debian.org/doc/manuals/debian-reference/ch09.en.html — retrieved 2025-01-09