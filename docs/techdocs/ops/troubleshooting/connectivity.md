---
title: "Connectivity Troubleshooting"
description: "Network connectivity issues and resolution steps."
tags: [troubleshooting, connectivity, network]
search: { boost: 3, exclude: false }
icon: material/network-off
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Resolve network and connectivity issues.

**Contents**
- [External Access Issues](#external-access-issues)
- [Service Communication](#service-communication)
- [Database Connectivity](#database-connectivity)
- [Cache Connectivity](#cache-connectivity)
- [Sources](#sources)

## External Access Issues

**Symptom:** Cannot access Firefly III from browser

| Issue | Symptom | Diagnosis | Solution | Source |
|-------|---------|-----------|----------|--------|
| Connection refused | Browser can't reach site | Service not running or port blocked | Check service status and firewall | "Firefly III Installation" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| 502 Bad Gateway | Nginx shows 502 error | PHP-FPM not responding | Restart application container | "Nginx Troubleshooting" — ops/docker/nginx/ — retrieved 2025-01-09 |
| Slow loading | Pages load very slowly | Resource exhaustion or network issues | Check container resources and network | "Performance Guide" — docs/techdocs/ops/troubleshooting/performance.md — retrieved 2025-01-09 |

**Diagnostic Commands:**
```bash
# Test external connectivity
curl -v http://localhost:8080/

# Check port binding
netstat -tulpn | grep :8080

# Test from different network
curl -v http://$(hostname -I | awk '{print $1}'):8080/

# Check firewall status
sudo ufw status verbose
```

## Service Communication

**Inter-container communication issues:**

| Service Pair | Error | Diagnosis | Solution | Source |
|--------------|-------|-----------|----------|--------|
| App → Database | Connection refused | Network or service issue | Check container network and database health | "Docker Networking" — docker-compose.yml — retrieved 2025-01-09 |
| App → Redis | Redis timeout | Redis not responding | Verify Redis container status | "Redis Configuration" — ops/docker/redis/ — retrieved 2025-01-09 |
| Nginx → App | 502/504 errors | PHP-FPM not responding | Check application container logs | "Nginx Configuration" — ops/docker/nginx/ — retrieved 2025-01-09 |

**Service Communication Tests:**
```bash
# Test database connectivity from app
docker exec firefly-iii-application.application nc -zv firefly-iii-application.mariadb 3306

# Test Redis connectivity from app
docker exec firefly-iii-application.application nc -zv firefly-iii-application.redis 6379

# Test app connectivity from nginx
docker exec firefly-iii-application.nginx nc -zv firefly-iii-application.application 9000

# Check container network
docker network inspect webgrip
```

## Database Connectivity

**Common database connection issues:**

| Issue | Symptom | Resolution | Source |
|-------|---------|------------|--------|
| Max connections reached | "Too many connections" | Increase max_connections or fix connection leaks | "MariaDB Tuning" — https://mariadb.com/kb/en/server-system-variables/#max_connections — retrieved 2025-01-09 |
| Connection timeout | Queries hang | Check wait_timeout and interactive_timeout | "MariaDB Configuration" — ops/docker/mariadb/ — retrieved 2025-01-09 |
| Authentication failure | Access denied | Verify credentials and host permissions | "Database Setup" — docs/techdocs/app/integrations/database.md — retrieved 2025-01-09 |

## Cache Connectivity

**Redis connection troubleshooting:**

| Issue | Symptom | Resolution | Source |
|-------|---------|------------|--------|
| Memory full | Redis OOM errors | Increase memory limit or implement eviction | "Redis Memory" — https://redis.io/docs/manual/eviction/ — retrieved 2025-01-09 |
| Connection drops | Intermittent failures | Check tcp-keepalive settings | "Redis Configuration" — ops/docker/redis/ — retrieved 2025-01-09 |
| Slow responses | High latency | Monitor Redis performance metrics | "Cache Performance" — docs/techdocs/app/integrations/cache.md — retrieved 2025-01-09 |

## Sources
- "Firefly III Installation Guide" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09
- "Docker Networking" — https://docs.docker.com/network/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/":"","https://docs.docker.com/network/":""},"sections":{"connectivity-troubleshooting":""}}}
-->