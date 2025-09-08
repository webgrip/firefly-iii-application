---
title: "Cache Integration"
description: "Redis cache configuration, session storage, and performance optimization."
tags: [cache, redis, integration, performance]
search: { boost: 3, exclude: false }
icon: material/cached
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Document cache integration for session storage and performance optimization.

**Contents**
- [Supported cache drivers](#supported-cache-drivers)
- [Redis configuration](#redis-configuration)
- [Container interface](#container-interface)
- [Performance considerations](#performance-considerations)
- [Sources](#sources)

## Supported cache drivers

Firefly III supports multiple cache backends:

| Driver | Use Case | Performance | Persistence | Source |
|--------|----------|-------------|-------------|--------|
| `redis` | **Production recommended** | High | Optional | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `file` | Development/testing | Medium | Yes | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `database` | Small installations | Low | Yes | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `memcached` | Alternative to Redis | High | No | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |

**This repository uses Redis for:**
- Application cache (`CACHE_DRIVER=redis`)
- Session storage (`SESSION_DRIVER=redis`)
- Queue backend (`QUEUE_CONNECTION=redis`)

## Redis configuration

**Environment variables:**

| Variable | Purpose | Example | Source |
|----------|---------|---------|--------|
| `REDIS_HOST` | Redis hostname | firefly-iii-application.redis | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `REDIS_PORT` | Redis port | 6379 | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `REDIS_PASSWORD` | Redis password | (from secrets) | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `REDIS_CLIENT` | PHP Redis client | phpredis | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `REDIS_PREFIX` | Key prefix | firefly-iii-application_ | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |

**Redis client configuration:**
- **Client**: `phpredis` (recommended over `predis` for performance)
- **Serialization**: PHP serialization for complex objects
- **Compression**: Disabled by default
- **Persistence**: Optional RDB snapshots for session recovery

## Container interface

| Aspect | Value / Path | Notes |
|--------|---------------|-------|
| Ports | 6379/tcp | Standard Redis port |
| Healthcheck | `redis-cli ping` | Returns PONG when healthy |
| Volumes | `/data` | Optional persistence for sessions |
| Configuration | `/usr/local/etc/redis/redis.conf` | Custom Redis configuration |
| Memory Policy | `allkeys-lru` | Evict least recently used keys when memory full |

**Docker Compose configuration:**
```yaml
firefly-iii-application.redis:
  image: webgrip/firefly-iii-application-redis:latest
  volumes:
    - firefly-iii-application-redis-data:/data
  healthcheck:
    test: ["CMD", "redis-cli", "ping"]
    interval: 30s
    timeout: 10s
    retries: 5
    start_period: 5s
```

**Redis configuration options:**
```conf
# Memory management
maxmemory 256mb
maxmemory-policy allkeys-lru

# Persistence (optional)
save 900 1
save 300 10
save 60 10000

# Performance
tcp-keepalive 300
timeout 0
```

## Performance considerations

**Memory usage:**
- **Sessions**: ~1-10KB per active user session
- **Cache**: Variable based on application usage
- **Queues**: Depends on background job volume
- **Recommended**: 256MB minimum, 512MB for high traffic

**Key expiration:**
- **Sessions**: Configured via `SESSION_LIFETIME` (default: 120 minutes)
- **Cache**: TTL varies by cached content (views, routes, config)
- **Queues**: Jobs removed after processing

**Monitoring:**
```bash
# Check memory usage
redis-cli info memory

# Monitor key count by type
redis-cli eval "return redis.call('keys', ARGV[1])" 0 "firefly-iii-application_*"

# Check active connections
redis-cli info clients
```

**Cache warming:**
- Application caches are populated on first access
- Route and configuration caching happens at startup
- No manual cache warming required

## Troubleshooting

**Common issues:**
1. **Connection refused**: Check Redis container health and network connectivity
2. **Memory pressure**: Monitor Redis memory usage and adjust limits
3. **Session loss**: Check Redis persistence configuration
4. **Slow performance**: Monitor Redis CPU usage and connection count

**Debug commands:**
```bash
# Test Redis connectivity
docker exec firefly-iii-application.application php -r "
  $redis = new Redis();
  $redis->connect('firefly-iii-application.redis', 6379);
  echo $redis->ping() ? 'Connected' : 'Failed';
"

# Clear all cache
docker exec firefly-iii-application.application php artisan cache:clear

# View Redis logs
docker logs firefly-iii-application.redis
```

## Sources
- "Firefly III Configuration Reference" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09
- "Redis Official Documentation" — https://redis.io/docs/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/":"","https://redis.io/docs/":""},"sections":{"cache-integration":""}}}
-->