---
title: "Startup Troubleshooting"
description: "Common startup issues and resolution steps."
tags: [troubleshooting, startup, containers]
search: { boost: 4, exclude: false }
icon: material/play-circle-outline
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Resolve common startup and initialization issues.

**Contents**
- [Container Startup Issues](#container-startup-issues)
- [Database Connection Problems](#database-connection-problems)
- [Application Initialization](#application-initialization)
- [Configuration Issues](#configuration-issues)
- [Sources](#sources)

## Container Startup Issues

**Symptom:** Containers fail to start or exit immediately

| Issue | Symptom | Cause | Solution | Source |
|-------|---------|-------|----------|--------|
| Container exits with code 125 | `docker ps` shows no containers | Invalid configuration or missing image | Check docker-compose.yml syntax, verify image availability | "Docker Troubleshooting" — https://docs.docker.com/engine/reference/run/ — retrieved 2025-01-09 |
| Port binding errors | `port already in use` error | Another service using same port | Stop conflicting service or change port mapping | "Docker Troubleshooting" — https://docs.docker.com/engine/reference/run/ — retrieved 2025-01-09 |
| Volume mount failures | Permission denied errors | Incorrect file permissions | Fix file ownership: `sudo chown -R 1000:1000 ./data` | "Docker Troubleshooting" — https://docs.docker.com/engine/reference/run/ — retrieved 2025-01-09 |
| Memory/CPU limits | Container killed (OOMKilled) | Insufficient resources | Increase memory limits or optimize application | "Docker Troubleshooting" — https://docs.docker.com/engine/reference/run/ — retrieved 2025-01-09 |

**Diagnostic Commands:**
```bash
# Check container status
docker ps -a | grep firefly-iii-application

# View container logs
docker logs firefly-iii-application.application --tail 50

# Check resource usage
docker stats firefly-iii-application.application --no-stream

# Inspect container configuration
docker inspect firefly-iii-application.application | jq '.State'

# Check for port conflicts
netstat -tulpn | grep :8080
```

**Common Startup Fixes:**
```bash
# Clear stopped containers
docker compose down --remove-orphans

# Rebuild containers if needed
docker compose build --no-cache

# Pull latest images
docker compose pull

# Start with verbose logging
docker compose up -d && docker compose logs -f
```

## Database Connection Problems

**Symptom:** Application fails to connect to database

| Issue | Error Message | Diagnosis | Solution | Source |
|-------|---------------|-----------|----------|--------|
| Database not ready | `Connection refused` | Database container not started | Wait for database health check or restart database | "Firefly III Installation" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Wrong credentials | `Access denied` | Incorrect username/password | Verify DB_USERNAME and DB_PASSWORD in .env | "Firefly III Installation" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Database doesn't exist | `Unknown database` | Database not created | Check MARIADB_DATABASE environment variable | "Firefly III Installation" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Character set issues | `Incorrect string value` | Wrong character set | Ensure utf8mb4 character set in database | "Firefly III Installation" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |

**Database Diagnostic Commands:**
```bash
# Check database container health
docker exec firefly-iii-application.mariadb \
  mysql -u root -p'${DB_ROOT_PASSWORD}' \
  -e "SHOW DATABASES;"

# Test application database connection
docker exec firefly-iii-application.mariadb \
  mysql -u application -p'${DB_PASSWORD}' application \
  -e "SELECT 1;"

# Check database character set
docker exec firefly-iii-application.mariadb \
  mysql -u application -p'${DB_PASSWORD}' \
  -e "SELECT @@character_set_database, @@collation_database;"

# Verify database user permissions
docker exec firefly-iii-application.mariadb \
  mysql -u root -p'${DB_ROOT_PASSWORD}' \
  -e "SHOW GRANTS FOR 'application'@'%';"
```

**Database Connection Fixes:**
```bash
# Reset database container
docker compose stop firefly-iii-application.mariadb
docker compose start firefly-iii-application.mariadb

# Wait for database to be ready
timeout 60 bash -c 'until docker exec firefly-iii-application.mariadb mysql -u application -p"${DB_PASSWORD}" -e "SELECT 1" >/dev/null 2>&1; do sleep 2; done'

# Recreate database with correct settings
docker exec firefly-iii-application.mariadb \
  mysql -u root -p'${DB_ROOT_PASSWORD}' \
  -e "DROP DATABASE IF EXISTS application; CREATE DATABASE application CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

## Application Initialization

**Symptom:** Application starts but fails during initialization

| Issue | Log Message | Cause | Solution | Source |
|-------|-------------|-------|----------|--------|
| Missing APP_KEY | `No application encryption key` | APP_KEY not set | Generate: `docker exec app php artisan key:generate` | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Migration failures | `Migration failed` | Database schema issues | Check database permissions and run migrations manually | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Cache/config issues | `Configuration cache` errors | Stale cache files | Clear cache: `php artisan config:clear` | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Permission errors | `Permission denied` | File system permissions | Fix storage permissions: `chown -R www-data:www-data storage` | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |

**Application Diagnostic Commands:**
```bash
# Check Laravel environment
docker exec firefly-iii-application.application php artisan env

# Verify configuration
docker exec firefly-iii-application.application php artisan config:show

# Check migration status
docker exec firefly-iii-application.application php artisan migrate:status

# Verify file permissions
docker exec firefly-iii-application.application ls -la storage/

# Test application configuration
docker exec firefly-iii-application.application php artisan firefly-iii:verify
```

**Application Initialization Fixes:**
```bash
# Generate application key
docker exec firefly-iii-application.application php artisan key:generate

# Clear all caches
docker exec firefly-iii-application.application php artisan cache:clear
docker exec firefly-iii-application.application php artisan config:clear
docker exec firefly-iii-application.application php artisan route:clear
docker exec firefly-iii-application.application php artisan view:clear

# Run migrations manually
docker exec firefly-iii-application.application php artisan migrate --force

# Fix storage permissions
docker exec firefly-iii-application.application chown -R www-data:www-data storage bootstrap/cache
```

## Configuration Issues

**Symptom:** Application starts but behaves incorrectly

| Issue | Symptom | Common Cause | Solution | Source |
|-------|---------|--------------|----------|--------|
| Wrong base URL | Links point to localhost | APP_URL misconfigured | Set correct APP_URL in .env | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Timezone issues | Wrong timestamps | APP_TIMEZONE not set | Set APP_TIMEZONE=Europe/Amsterdam | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Email not working | No emails sent | MAIL_* variables wrong | Configure SMTP settings correctly | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Cache not working | Slow performance | Redis connection failed | Check REDIS_HOST and REDIS_PORT | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |

**Configuration Validation:**
```bash
# Validate critical environment variables
docker exec firefly-iii-application.application env | grep -E "(APP_|DB_|REDIS_)" | sort

# Test Redis connection
docker exec firefly-iii-application.application \
  php -r "
    \$redis = new Redis();
    \$redis->connect('firefly-iii-application.redis', 6379);
    echo \$redis->ping() ? 'Redis OK' : 'Redis Failed';
  "

# Test email configuration (development only)
docker exec firefly-iii-application.application \
  php artisan tinker --execute="Mail::raw('Test', function(\$m) { \$m->to('test@example.com')->subject('Test'); });"

# Verify URL configuration
curl -I http://localhost:8080/ | grep -i location
```

## Startup Sequence Troubleshooting

**Normal Startup Sequence:**
1. Database container starts and initializes
2. Redis container starts
3. Application container starts
4. Database migrations run
5. Cache warming occurs
6. Nginx starts and serves traffic

**Startup Timeline Issues:**

| Step | Timeout | Failure Mode | Debug Command |
|------|---------|--------------|---------------|
| Database ready | 30s | Connection refused | `docker logs firefly-iii-application.mariadb` |
| Redis ready | 5s | Connection refused | `docker logs firefly-iii-application.redis` |
| Migrations | 60s | SQL errors | `docker exec app php artisan migrate:status` |
| App startup | 90s | PHP errors | `docker logs firefly-iii-application.application` |
| Health check | 60s | HTTP 503 | `curl -v http://localhost:8080/health` |

**Startup Sequence Debug:**
```bash
# Monitor startup sequence
watch -n 2 'docker ps --format "table {{ '{' }}.Names{{ '}' }}\t{{ '{' }}.Status{{ '}' }}\t{{ '{' }}.Ports{{ '}' }}"'

# Check dependency order
docker compose config | grep -A 5 depends_on

# Test service dependencies manually
docker exec firefly-iii-application.application nc -zv firefly-iii-application.mariadb 3306
docker exec firefly-iii-application.application nc -zv firefly-iii-application.redis 6379
```

## Quick Recovery Scripts

**Full Stack Restart:**
```bash
#!/bin/bash
# full-restart.sh
docker compose down
docker compose pull
docker compose up -d
echo "Waiting for services to be ready..."
timeout 300 bash -c 'until curl -f http://localhost:8080/health >/dev/null 2>&1; do sleep 5; done'
echo "Services ready!"
```

**Database Reset (Development Only):**
```bash
#!/bin/bash
# reset-database.sh
docker compose stop firefly-iii-application.application
docker compose exec firefly-iii-application.mariadb \
  mysql -u root -p"${DB_ROOT_PASSWORD}" \
  -e "DROP DATABASE application; CREATE DATABASE application CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
docker compose start firefly-iii-application.application
```

## Sources
- "Firefly III Installation Guide" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09
- "Docker Troubleshooting Guide" — https://docs.docker.com/engine/reference/run/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/":"","https://docs.docker.com/engine/reference/run/":""},"sections":{"startup-troubleshooting":""}}}
-->