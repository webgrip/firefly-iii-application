---
title: "Upgrades & Rollback"
description: "Version upgrade procedures, rollback strategies, and validation steps."
tags: [upgrades, rollback, deployment]
search: { boost: 3, exclude: false }
icon: material/update
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Ensure safe and reliable version management.

**Contents**
- [Upgrade Strategy](#upgrade-strategy)
- [Pre-Upgrade Preparation](#pre-upgrade-preparation)
- [Upgrade Execution](#upgrade-execution)
- [Rollback Procedures](#rollback-procedures)
- [Validation & Testing](#validation--testing)
- [Sources](#sources)

## Upgrade Strategy

**Tagging Policy:**
- **Production**: Use specific version tags (e.g., `6.1.17`, `6.1.18`)
- **Staging**: Use release candidate tags (e.g., `6.1.18-rc1`)
- **Development**: Use specific commits or feature branches
- **Never**: Use `latest` tag in any environment

**Version Support Matrix:**

| Environment | Upgrade Frequency | Version Lag | Approval Required |
|-------------|-------------------|-------------|-------------------|
| Development | Immediate | Latest RC | Developer |
| Staging | Weekly | 1-2 versions behind latest | Team Lead |
| Production | Monthly | 2-4 versions behind latest | Change Board |

**Upgrade Types:**

| Type | Example | Database Changes | Downtime | Risk Level |
|------|---------|------------------|----------|------------|
| Patch | 6.1.17 → 6.1.18 | Rare | None | Low |
| Minor | 6.1.x → 6.2.0 | Possible | Minimal | Medium |
| Major | 6.x → 7.0 | Likely | Planned | High |

## Pre-Upgrade Preparation

**Backup Verification:**
```bash
# Create pre-upgrade backup
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups/pre-upgrade-${DATE}"
mkdir -p "${BACKUP_DIR}"

# Database backup
docker exec firefly-iii-application.mariadb mysqldump \
  -u application -p'${DB_PASSWORD}' application \
  > "${BACKUP_DIR}/database.sql"

# Storage backup
docker run --rm \
  -v firefly-iii-application-application-storage-data:/source:ro \
  -v "${BACKUP_DIR}":/backup \
  alpine:latest \
  tar czf /backup/storage.tar.gz -C /source .

# Configuration backup
cp .env "${BACKUP_DIR}/env.backup"
cp docker-compose.yml "${BACKUP_DIR}/compose.backup"

# Verify backup integrity
test -s "${BACKUP_DIR}/database.sql" || { echo "Database backup failed"; exit 1; }
test -s "${BACKUP_DIR}/storage.tar.gz" || { echo "Storage backup failed"; exit 1; }
```

**Environment Validation:**
```bash
# Check current version
docker exec firefly-iii-application.application \
  php artisan --version

# Verify system health
curl -f http://localhost:8080/health || { echo "System unhealthy"; exit 1; }

# Check resource availability
df -h | grep -E "(/$|/var/lib/docker)" | awk '$5 > "80%" {print "Disk space warning: " $0}'

# Verify database connectivity
docker exec firefly-iii-application.mariadb \
  mysql -u application -p'${DB_PASSWORD}' \
  -e "SELECT COUNT(*) FROM transactions;" || { echo "Database check failed"; exit 1; }
```

**Change Documentation:**
```bash
# Review changelog for breaking changes
curl -s https://api.github.com/repos/firefly-iii/firefly-iii/releases/latest | \
  jq -r '.body' > changelog.md

# Check for configuration changes
echo "Review configuration requirements in changelog.md"
echo "Update .env file if necessary"
echo "Verify all required environment variables are set"
```

## Upgrade Execution

**Docker Compose Upgrade:**
```bash
# Set target version
TARGET_VERSION="6.1.18"

# Update docker-compose.yml
sed -i.backup "s/tag: \".*\"/tag: \"${TARGET_VERSION}\"/" docker-compose.yml

# Pull new images
docker compose pull

# Stop application (keep database running)
docker compose stop firefly-iii-application.application firefly-iii-application.nginx

# Start application with new version
docker compose up -d firefly-iii-application.application firefly-iii-application.nginx

# Wait for startup
echo "Waiting for application startup..."
timeout 300 bash -c 'until curl -f http://localhost:8080/health; do sleep 5; done'
```

**Kubernetes Upgrade:**
```bash
# Update Helm values
TARGET_VERSION="6.1.18"
yq eval ".application.controllers.main.containers.app.image.tag = \"${TARGET_VERSION}\"" \
  -i ops/helm/application-application/values.yaml

# Helm upgrade with rollback capability
helm upgrade firefly-iii-application \
  ops/helm/application-application \
  --namespace firefly-iii-application \
  --wait \
  --timeout 10m \
  --atomic

# Verify deployment
kubectl get pods -n firefly-iii-application
kubectl logs -n firefly-iii-application deployment/firefly-iii-application
```

**Database Migrations:**
```bash
# Migrations run automatically on startup, but can be run manually
docker exec firefly-iii-application.application \
  php artisan migrate --force

# Check migration status
docker exec firefly-iii-application.application \
  php artisan migrate:status
```

## Rollback Procedures

**Immediate Rollback (Docker Compose):**
```bash
# Restore previous version
PREVIOUS_VERSION="6.1.17"  # Set to known good version

# Stop current version
docker compose down

# Restore configuration
cp docker-compose.yml.backup docker-compose.yml
cp .env.backup .env

# Update to previous version
sed -i "s/tag: \".*\"/tag: \"${PREVIOUS_VERSION}\"/" docker-compose.yml

# Restore database if needed
if [ "${RESTORE_DATABASE}" = "true" ]; then
  # Stop all services
  docker compose down
  
  # Restore database
  cat "${BACKUP_DIR}/database.sql" | \
    docker exec -i firefly-iii-application.mariadb \
    mysql -u application -p'${DB_PASSWORD}' application
fi

# Start previous version
docker compose up -d

# Verify rollback
timeout 300 bash -c 'until curl -f http://localhost:8080/health; do sleep 5; done'
```

**Kubernetes Rollback:**
```bash
# Helm rollback to previous release
helm rollback firefly-iii-application -n firefly-iii-application

# Or rollback to specific revision
helm history firefly-iii-application -n firefly-iii-application
helm rollback firefly-iii-application 3 -n firefly-iii-application

# Verify rollback
kubectl get pods -n firefly-iii-application
kubectl rollout status deployment/firefly-iii-application -n firefly-iii-application
```

**Database Rollback:**
```bash
# Database rollback (destructive - use with caution)
# Only if database schema changes are incompatible

# Stop application
docker compose stop firefly-iii-application.application

# Restore database from backup
docker exec -i firefly-iii-application.mariadb \
  mysql -u application -p'${DB_PASSWORD}' application \
  < "${BACKUP_DIR}/database.sql"

# Restore storage if needed
docker run --rm \
  -v firefly-iii-application-application-storage-data:/target \
  -v "${BACKUP_DIR}":/backup:ro \
  alpine:latest \
  tar xzf /backup/storage.tar.gz -C /target

# Start application
docker compose start firefly-iii-application.application
```

## Validation & Testing

**Post-Upgrade Validation:**
```bash
# Health check validation
curl -f http://localhost:8080/health | jq '.status' | grep -q "healthy"

# Version verification
CURRENT_VERSION=$(docker exec firefly-iii-application.application php artisan --version | grep -o 'v[0-9.]*')
echo "Current version: ${CURRENT_VERSION}"

# Database integrity check
docker exec firefly-iii-application.application \
  php artisan firefly-iii:verify

# Basic functionality test
curl -s http://localhost:8080/ | grep -q "Firefly III"
```

**Functional Testing:**
```bash
# Test critical user journeys
echo "Testing login functionality..."
# Add automated login test here

echo "Testing transaction creation..."
# Add automated transaction test here

echo "Testing data export..."
# Add automated export test here

echo "Testing import functionality..."
# Add automated import test here
```

**Performance Validation:**
```bash
# Response time check
RESPONSE_TIME=$(curl -o /dev/null -s -w '%{time_total}' http://localhost:8080/)
echo "Response time: ${RESPONSE_TIME}s"

# Memory usage check
MEMORY_USAGE=$(docker stats --no-stream --format "{{.MemUsage}}" firefly-iii-application.application)
echo "Memory usage: ${MEMORY_USAGE}"

# Database performance check
docker exec firefly-iii-application.mariadb \
  mysql -u application -p'${DB_PASSWORD}' \
  -e "SHOW GLOBAL STATUS LIKE 'Threads_connected';"
```

## Upgrade Checklist

**Pre-Upgrade:**
- [ ] Review changelog for breaking changes
- [ ] Create database backup
- [ ] Create storage backup
- [ ] Verify system health
- [ ] Schedule maintenance window
- [ ] Notify users of potential downtime

**During Upgrade:**
- [ ] Update configuration files
- [ ] Pull new container images
- [ ] Stop application services
- [ ] Start services with new version
- [ ] Monitor startup logs
- [ ] Verify health checks pass

**Post-Upgrade:**
- [ ] Validate version upgrade
- [ ] Test critical functionality
- [ ] Check performance metrics
- [ ] Verify all services healthy
- [ ] Document any issues encountered
- [ ] Notify users of completion

**Rollback (if needed):**
- [ ] Stop current version
- [ ] Restore previous configuration
- [ ] Restore database if necessary
- [ ] Start previous version
- [ ] Verify rollback successful
- [ ] Document rollback reason

## Emergency Procedures

**Critical Upgrade Failure:**
1. Immediately stop the application
2. Restore from backup
3. Start previous version
4. Verify system health
5. Notify stakeholders
6. Investigate root cause

**Data Corruption Detection:**
1. Stop application immediately
2. Do not restart until investigation complete
3. Restore from most recent clean backup
4. Document corruption details
5. Contact support if needed

## Sources
- "Firefly III Upgrade Guide" — https://docs.firefly-iii.org/how-to/firefly-iii/upgrade/ — retrieved 2025-01-09
- "Firefly III Installation Guide" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/upgrade/":"","https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/":""},"sections":{"upgrades-rollback":""}}}
-->