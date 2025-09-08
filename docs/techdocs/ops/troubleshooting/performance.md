---
title: "Performance Troubleshooting"
description: "Performance issues and optimization strategies."
tags: [troubleshooting, performance, optimization]
search: { boost: 3, exclude: false }
icon: material/speedometer
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Diagnose and resolve performance issues.

**Contents**
- [Slow Response Times](#slow-response-times)
- [High Resource Usage](#high-resource-usage)
- [Database Performance](#database-performance)
- [Cache Performance](#cache-performance)
- [Sources](#sources)

## Slow Response Times

**Symptom:** Pages load slowly or time out

| Issue | Cause | Diagnosis | Solution | Source |
|-------|-------|-----------|----------|--------|
| High CPU usage | Resource exhaustion | Monitor with `docker stats` | Scale resources or optimize code | "Performance Tuning" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Memory pressure | Insufficient RAM | Check memory usage patterns | Increase memory limits | "Container Resources" — docker-compose.yml — retrieved 2025-01-09 |
| Disk I/O bottleneck | Slow storage | Monitor disk utilization | Use faster storage or optimize queries | "Storage Performance" — https://docs.docker.com/storage/ — retrieved 2025-01-09 |

**Performance Monitoring:**
```bash
# Monitor container resources
docker stats firefly-iii-application.application --no-stream

# Check system load
uptime

# Monitor disk I/O
iostat -x 1 5

# Check memory usage
free -h

# Monitor network
iftop -t -s 10
```

## High Resource Usage

**Resource optimization strategies:**

| Resource | Monitoring | Optimization | Source |
|----------|------------|--------------|--------|
| CPU | `docker stats`, `top` | Optimize queries, enable caching | "PHP Performance" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Memory | `docker stats`, `free` | Tune PHP memory limits, optimize cache | "Memory Tuning" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Disk | `df`, `iostat` | Archive old data, optimize storage | "Storage Management" — docs/techdocs/app/configuration/files-and-paths.md — retrieved 2025-01-09 |

## Database Performance

**Database optimization checks:**

| Issue | Symptom | Solution | Source |
|-------|---------|----------|--------|
| Slow queries | High response times | Analyze and optimize slow queries | "MariaDB Performance" — https://mariadb.com/kb/en/optimization-and-tuning/ — retrieved 2025-01-09 |
| Lock contention | Blocking transactions | Optimize transaction scope and indexing | "Database Optimization" — docs/techdocs/app/integrations/database.md — retrieved 2025-01-09 |
| Table bloat | Large table sizes | Regular maintenance and optimization | "Database Maintenance" — docs/techdocs/ops/runbook.md — retrieved 2025-01-09 |

**Database Performance Commands:**
```bash
# Check slow queries
docker exec firefly-iii-application.mariadb \
  mysql -u application -p'${DB_PASSWORD}' \
  -e "SELECT * FROM information_schema.processlist WHERE time > 30;"

# Check table sizes
docker exec firefly-iii-application.mariadb \
  mysql -u application -p'${DB_PASSWORD}' \
  -e "SELECT table_name, ROUND(((data_length + index_length) / 1024 / 1024), 2) AS 'Size (MB)' 
      FROM information_schema.tables WHERE table_schema = 'application' 
      ORDER BY (data_length + index_length) DESC LIMIT 10;"

# Optimize tables
docker exec firefly-iii-application.mariadb \
  mysql -u application -p'${DB_PASSWORD}' \
  -e "OPTIMIZE TABLE application.transactions, application.accounts, application.transaction_journals;"
```

## Cache Performance

**Redis performance optimization:**

| Metric | Command | Target | Action if Poor | Source |
|--------|---------|--------|----------------|--------|
| Hit ratio | `redis-cli info stats` | >90% | Review cache strategy | "Redis Performance" — https://redis.io/docs/manual/performance/ — retrieved 2025-01-09 |
| Memory usage | `redis-cli info memory` | <80% of limit | Increase memory or eviction policy | "Cache Configuration" — docs/techdocs/app/integrations/cache.md — retrieved 2025-01-09 |
| Response time | Custom monitoring | <5ms | Check network and memory | "Redis Monitoring" — https://redis.io/docs/manual/admin/ — retrieved 2025-01-09 |

**Cache Performance Commands:**
```bash
# Check Redis performance
docker exec firefly-iii-application.redis redis-cli info stats | grep -E "(hits|misses)"

# Monitor Redis memory
docker exec firefly-iii-application.redis redis-cli info memory

# Check Redis latency
docker exec firefly-iii-application.redis redis-cli --latency-history

# Monitor Redis operations
docker exec firefly-iii-application.redis redis-cli monitor
```

## Sources
- "Firefly III Performance Guide" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09
- "Docker Performance Tuning" — https://docs.docker.com/config/containers/resource_constraints/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/":"","https://docs.docker.com/config/containers/resource_constraints/":""},"sections":{"performance-troubleshooting":""}}}
-->