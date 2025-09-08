---
title: "Startup Issues"
description: "Common startup problems and their solutions."
tags: [troubleshooting, startup, firefly-iii]
search: { boost: 4, exclude: false }
icon: material/alert-circle
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Diagnose and resolve startup issues.

**Contents**
- [Common Symptoms](#common-symptoms)
- [Database Connection Issues](#database-connection-issues)
- [Application Key Problems](#application-key-problems)
- [Permission Errors](#permission-errors)
- [Sources](#sources)

### Common Symptoms

| Symptom | Likely Cause | Quick Fix |
|---------|--------------|-----------|
| Container exits immediately | Missing APP_KEY | Generate key: `php artisan key:generate` |
| "Connection refused" errors | Database not ready | Wait 30s, check `docker compose ps` |
| 500 Internal Server Error | Missing environment variables | Check `.env` file completeness |
| Permission denied on storage | File ownership issues | Fix permissions: `chown -R www-data:www-data storage` |
| Migration failures | Database connection or schema issues | Check DB credentials and connectivity |

### Database Connection Issues

**Symptoms:**
- "SQLSTATE[HY000] [2002] Connection refused"
- "SQLSTATE[28000] [1045] Access denied for user"
- Application hangs during startup

**Diagnosis:**
```bash
# Check database container status
docker compose ps firefly-iii-application.mariadb

# Test database connectivity
docker compose exec firefly-iii-application.mariadb \
  mysql -u application -papplication -e "SELECT 1"

# Check application logs for DB errors
docker compose logs firefly-iii-application.application | grep -i database
```

**Solutions:**
1. **Database not ready**: Wait for healthcheck to pass (up to 30s)
2. **Wrong credentials**: Verify `DB_*` variables in `.env` match container environment
3. **Network issues**: Ensure containers are on same Docker network
4. **Host connectivity**: Use container names, not localhost for DB_HOST

### Application Key Problems

**Symptoms:**
- "No application encryption key has been specified"
- "RuntimeException: No application encryption key"
- Immediate container exit

**Solution:**
```bash
# Generate application key
docker compose run --rm firefly-iii-application.application \
  php artisan key:generate --show

# Add to .env file
echo "APP_KEY=base64:GENERATED_KEY_HERE" >> .env

# Restart services
docker compose up -d
```

### Permission Errors

**Symptoms:**
- "Permission denied" when writing to storage
- File upload failures
- Log write errors

**Solutions:**
```bash
# Fix storage permissions
docker compose exec firefly-iii-application.application \
  chown -R www-data:www-data storage bootstrap/cache

# Set proper file permissions
docker compose exec firefly-iii-application.application \
  chmod -R 755 storage bootstrap/cache
```

### Sources
- "Firefly III Troubleshooting" — https://docs.firefly-iii.org/explanation/more-information/faq — retrieved 2025-01-09
- "Laravel Troubleshooting" — https://laravel.com/docs/10.x/deployment — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/explanation/more-information/faq":"sha256:pending","https://laravel.com/docs/10.x/deployment":"sha256:pending"},"sections":{"startup-troubleshooting":"sha256:pending"}}}
-->