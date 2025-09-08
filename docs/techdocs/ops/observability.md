---
title: "Observability"
description: "Metrics, logs, traces setup and monitoring guidelines."
tags: [observability, monitoring, logging]
search: { boost: 3, exclude: false }
icon: material/chart-line
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Enable effective monitoring and troubleshooting.

**Contents**
- [Logging Strategy](#logging-strategy)
- [Metrics Collection](#metrics-collection)
- [Health Monitoring](#health-monitoring)
- [Alerting](#alerting)
- [Sources](#sources)

## Logging Strategy

**Log Destinations:**

| Component | Log Location | Format | Rotation | Purpose | Source |
|-----------|--------------|--------|----------|---------|--------|
| Firefly III Application | `/var/www/app/storage/logs/laravel.log` | JSON | Daily | Application events, errors | "Firefly III Logging" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Nginx Access | stdout | Combined log format | Docker log driver | HTTP request tracking | "Nginx Configuration" — ops/docker/nginx/ — retrieved 2025-01-09 |
| Nginx Error | stderr | Standard error format | Docker log driver | Web server errors | "Nginx Configuration" — ops/docker/nginx/ — retrieved 2025-01-09 |
| MariaDB | `/var/log/mysql/error.log` | MySQL format | Size-based | Database errors | "MariaDB Configuration" — ops/docker/mariadb/ — retrieved 2025-01-09 |
| Redis | stdout | Redis format | Docker log driver | Cache operations | "Redis Configuration" — ops/docker/redis/ — retrieved 2025-01-09 |

**Structured Logging Configuration:**
```bash
# Laravel logging configuration
LOG_CHANNEL=stderr
LOG_LEVEL=info
LOG_STACK=single

# JSON format for structured logs
LOG_JSON_FORMAT=true
```

**Log Level Guidelines:**

| Level | When to Use | Examples |
|-------|-------------|----------|
| DEBUG | Development debugging | SQL queries, cache operations |
| INFO | Normal operations | User login, transaction creation |
| WARNING | Recoverable errors | Failed external API calls, validation warnings |
| ERROR | Application errors | Unhandled exceptions, database errors |
| CRITICAL | System failures | Database unavailable, filesystem full |

**Centralized Logging (Production):**
```yaml
# Example logging sidecar for Kubernetes
- name: log-shipper
  image: fluent/fluent-bit:2.0
  volumeMounts:
    - name: app-logs
      mountPath: /var/log/app
    - name: fluent-bit-config
      mountPath: /fluent-bit/etc
```

## Metrics Collection

**Application Metrics:**

| Metric | Type | Description | Labels | Source |
|--------|------|-------------|--------|--------|
| `firefly_transactions_total` | Counter | Total transactions created | type, user_id | "Firefly III Metrics" — Application Events — retrieved 2025-01-09 |
| `firefly_active_users` | Gauge | Currently active users | - | "Firefly III Metrics" — Application Events — retrieved 2025-01-09 |
| `firefly_account_balance` | Gauge | Account balances by currency | account_type, currency | "Firefly III Metrics" — Application Events — retrieved 2025-01-09 |
| `firefly_budget_utilization` | Gauge | Budget spending percentage | budget_id, period | "Firefly III Metrics" — Application Events — retrieved 2025-01-09 |
| `firefly_import_jobs_total` | Counter | Data import jobs | status, format | "Firefly III Metrics" — Application Events — retrieved 2025-01-09 |
| `firefly_rule_executions_total` | Counter | Rule engine executions | rule_id, outcome | "Firefly III Metrics" — Application Events — retrieved 2025-01-09 |

**Infrastructure Metrics:**

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `container_cpu_usage_percent` | Gauge | Container CPU utilization | container_name |
| `container_memory_usage_bytes` | Gauge | Container memory usage | container_name |
| `mysql_connections_active` | Gauge | Active database connections | - |
| `redis_memory_usage_bytes` | Gauge | Redis memory consumption | - |
| `http_requests_total` | Counter | HTTP requests received | method, status_code |
| `http_request_duration_seconds` | Histogram | HTTP request latency | method, status_code |

**Prometheus Configuration:**
```yaml
# prometheus.yml snippet
scrape_configs:
  - job_name: 'firefly-iii'
    static_configs:
      - targets: ['localhost:8080']
    metrics_path: '/metrics'
    scrape_interval: 30s

  - job_name: 'mysql-exporter'
    static_configs:
      - targets: ['localhost:9104']

  - job_name: 'redis-exporter'
    static_configs:
      - targets: ['localhost:9121']
```

**Custom Metrics Implementation:**
```php
// Example application metric
use Prometheus\Counter;

class TransactionCreatedListener
{
    private Counter $transactionCounter;
    
    public function handle(TransactionCreated $event): void
    {
        $this->transactionCounter
            ->labels($event->transaction->type, $event->user_id)
            ->inc();
    }
}
```

## Health Monitoring

**Health Check Endpoints:**

| Endpoint | Purpose | Response Format | Checks Performed | Source |
|----------|---------|-----------------|------------------|--------|
| `/health` | Basic health status | JSON | HTTP 200/503, basic connectivity | "Health Check Implementation" — src/index.js — retrieved 2025-01-09 |
| `/health/detailed` | Comprehensive health | JSON | Database, Redis, filesystem, external APIs | "Health Check Implementation" — src/index.js — retrieved 2025-01-09 |
| `/metrics` | Prometheus metrics | Prometheus format | Application and system metrics | "Metrics Implementation" — Custom — retrieved 2025-01-09 |

**Health Check Response Format:**
```json
{
  "status": "healthy",
  "timestamp": "2025-01-09T10:30:00Z",
  "checks": {
    "database": {
      "status": "healthy",
      "response_time_ms": 12,
      "connection_pool": {
        "active": 3,
        "max": 10
      }
    },
    "redis": {
      "status": "healthy", 
      "response_time_ms": 2,
      "memory_usage_mb": 45
    },
    "filesystem": {
      "status": "healthy",
      "storage_usage_percent": 65
    }
  }
}
```

**Container Health Checks:**
```yaml
# Docker Compose health check
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:80/health"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 60s
```

**Readiness vs Liveness:**
- **Liveness**: Basic process health (HTTP 200 response)
- **Readiness**: Full system health (database, cache, dependencies)
- **Startup**: Initial system initialization (longer timeout)

## Alerting

**Critical Alerts (Immediate Response):**

| Alert | Condition | Severity | Action Required |
|-------|-----------|----------|-----------------|
| Application Down | HTTP health check fails for 3 minutes | Critical | Immediate investigation |
| Database Unavailable | Database connection failures > 50% | Critical | Database recovery |
| High Error Rate | Error rate > 5% for 5 minutes | High | Code/configuration review |
| Memory Exhaustion | Container memory > 90% for 10 minutes | High | Resource scaling |

**Warning Alerts (Next Business Day):**

| Alert | Condition | Severity | Action Required |
|-------|-----------|----------|-----------------|
| Slow Response Times | P95 latency > 2s for 15 minutes | Medium | Performance optimization |
| Disk Space Low | Storage usage > 80% | Medium | Cleanup or expansion |
| High Transaction Volume | 50% above normal for 1 hour | Medium | Capacity planning |

**Alert Manager Configuration:**
```yaml
# alertmanager.yml
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 12h
  receiver: 'web.hook'

receivers:
  - name: 'web.hook'
    webhook_configs:
      - url: 'https://hooks.slack.com/services/...'
        send_resolved: true
```

**Prometheus Alert Rules:**
```yaml
# alerts.yml
groups:
  - name: firefly-iii
    rules:
      - alert: FireflyIIIDown
        expr: up{job="firefly-iii"} == 0
        for: 3m
        labels:
          severity: critical
        annotations:
          summary: "Firefly III is down"
          description: "Firefly III has been down for more than 3 minutes"

      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: high
        annotations:
          summary: "High error rate detected"
```

## Dashboards

**Grafana Dashboard Panels:**

1. **Overview Dashboard:**
   - Service health status
   - Request rate and latency
   - Error rate trends
   - Active user count

2. **Application Dashboard:**
   - Transaction creation rate
   - Account balance trends
   - Budget utilization
   - Import job success rate

3. **Infrastructure Dashboard:**
   - Container resource usage
   - Database performance metrics
   - Cache hit rates
   - Network traffic

**Dashboard Configuration:**
```json
{
  "dashboard": {
    "title": "Firefly III Overview",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{status}}"
          }
        ]
      }
    ]
  }
}
```

## Troubleshooting Workflows

**Performance Issues:**
1. Check dashboard for resource utilization
2. Review slow query logs
3. Analyze request patterns
4. Check for memory leaks
5. Verify cache performance

**Error Investigation:**
1. Check error rate dashboard
2. Review application logs
3. Correlate with deployment events
4. Check external service status
5. Verify configuration changes

**Capacity Planning:**
1. Monitor growth trends
2. Analyze peak usage patterns
3. Project future requirements
4. Plan scaling activities

## Sources
- "Firefly III Installation Guide" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09
- "Laravel Logging Documentation" — https://laravel.com/docs/logging — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/":"","https://laravel.com/docs/logging":""},"sections":{"observability":""}}}
-->