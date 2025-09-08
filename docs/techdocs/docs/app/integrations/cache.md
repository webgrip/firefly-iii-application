---
title: "Cache Integration"
description: "Redis configuration for sessions and application caching."
tags: [cache, redis, firefly-iii]
search: { boost: 3, exclude: false }
icon: material/memory
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Document Redis cache integration for Firefly III.

**Contents**
- [Cache Requirements](#cache-requirements)
- [Container Interface](#container-interface)
- [Configuration](#configuration)
- [Sources](#sources)

### Cache Requirements

Firefly III can use Redis for:
- **Session Storage**: User session data persistence
- **Application Cache**: Query results, computed values
- **Queue Backend**: Background job processing
- **Rate Limiting**: API throttling and abuse prevention

**Benefits over file-based caching:**
- Better performance in multi-container deployments
- Shared cache across application instances
- Automatic expiration and memory management
- Atomic operations for counters and locks

### Container Interface

| Aspect | Value / Path | Notes |
|-------|---------------|-------|
| Image | webgrip/firefly-iii-application-redis:latest | Org-owned image based on official Redis |
| Ports | 6379:6379 | Standard Redis port |
| Healthcheck | `redis-cli ping` | Returns PONG when ready |
| Volumes | `/data` | Persistent Redis data (if persistence enabled) |
| Memory Policy | allkeys-lru | Evict least recently used keys when memory full |
| Persistence | RDB snapshots | Optional disk persistence for important cache data |

### Configuration

**Environment Variables:**
```bash
CACHE_DRIVER=redis
CACHE_HOST=firefly-iii-application.redis
CACHE_PORT=6379
CACHE_PASSWORD=  # Optional Redis password

SESSION_DRIVER=redis
SESSION_LIFETIME=120  # Minutes

QUEUE_CONNECTION=redis  # Optional queue backend
```

**Redis Configuration:**
- **Max Memory**: 256MB default (adjust based on usage)
- **Eviction Policy**: allkeys-lru (remove old cache entries)
- **Persistence**: Optional RDB snapshots for session persistence
- **Security**: No authentication by default (container network isolation)

**Performance Tuning:**
- Monitor memory usage and adjust max memory limit
- Use appropriate eviction policy for workload
- Consider Redis clustering for high availability
- Monitor cache hit rates and key expiration

**Cache Keys Pattern:**
- Sessions: `laravel_session:<session_id>`
- Application cache: `laravel_cache:<key>`
- Database cache: `laravel_cache:database:<query_hash>`
- Rate limiting: `laravel_cache:throttle:<key>`

### Sources
- "Redis Official Documentation" — https://redis.io/documentation — retrieved 2025-01-09
- "Laravel Redis Configuration" — https://laravel.com/docs/10.x/redis — retrieved 2025-01-09
- "Firefly III Caching" — https://docs.firefly-iii.org/explanation/more-information/performance — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://redis.io/documentation":"sha256:pending","https://laravel.com/docs/10.x/redis":"sha256:pending","https://docs.firefly-iii.org/explanation/more-information/performance":"sha256:pending"},"sections":{"cache-integration":"sha256:pending"}}}
-->