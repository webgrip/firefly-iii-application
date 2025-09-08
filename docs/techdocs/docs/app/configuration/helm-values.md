---
title: "Helm — Used Values"
description: "Only the values from values.yaml that this repo actually uses."
tags: [helm, configuration]
search: { boost: 2, exclude: false }
icon: material/ship-wheel
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Keep Helm docs tight and truthful.

**Contents**
- [Values we set](#values-we-set)
- [Sources](#sources)

## Values we set

| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `namespace` | firefly-iii-application | Kubernetes namespace for deployment | "Helm Chart Values" — ops/helm/application-application/values.yaml — retrieved 2025-01-09 |
| `_shared_config.hostname` | firefly-iii-application.staging.k8s.webgrip.nl | External hostname for ingress | "Helm Chart Values" — ops/helm/application-application/values.yaml — retrieved 2025-01-09 |
| `_shared_config.url` | https://firefly-iii-application.staging.k8s.webgrip.nl | Full external URL | "Helm Chart Values" — ops/helm/application-application/values.yaml — retrieved 2025-01-09 |
| `application.controllers.main.containers.app.image.repository` | docker.io/webgrip/firefly-iii-application | Container image repository | "Helm Chart Values" — ops/helm/application-application/values.yaml — retrieved 2025-01-09 |
| `application.controllers.main.containers.app.image.tag` | latest | Container image tag (use specific versions in production) | "Helm Chart Values" — ops/helm/application-application/values.yaml — retrieved 2025-01-09 |
| `application.controllers.main.containers.app.resources.requests.cpu` | 250m | CPU request for application container | "Helm Chart Values" — ops/helm/application-application/values.yaml — retrieved 2025-01-09 |
| `application.controllers.main.containers.app.resources.requests.memory` | 512Mi | Memory request for application container | "Helm Chart Values" — ops/helm/application-application/values.yaml — retrieved 2025-01-09 |
| `application.controllers.main.containers.app.resources.limits.cpu` | 1000m | CPU limit for application container | "Helm Chart Values" — ops/helm/application-application/values.yaml — retrieved 2025-01-09 |
| `application.controllers.main.containers.app.resources.limits.memory` | 1Gi | Memory limit for application container | "Helm Chart Values" — ops/helm/application-application/values.yaml — retrieved 2025-01-09 |
| `application.controllers.main.containers.web.image.repository` | nginxinc/nginx-unprivileged | Nginx container image | "Helm Chart Values" — ops/helm/application-application/values.yaml — retrieved 2025-01-09 |
| `application.controllers.main.containers.web.image.tag` | 1.29.1-alpine3.22-perl | Nginx container image tag | "Helm Chart Values" — ops/helm/application-application/values.yaml — retrieved 2025-01-09 |
| `mariadb.enabled` | true | Enable MariaDB database | "Helm Chart Values" — ops/helm/application-application/values.yaml — retrieved 2025-01-09 |
| `mariadb.database` | firefly-iii-application | Database name | "Helm Chart Values" — ops/helm/application-application/values.yaml — retrieved 2025-01-09 |
| `mariadb.username` | firefly-iii-application | Database username | "Helm Chart Values" — ops/helm/application-application/values.yaml — retrieved 2025-01-09 |
| `redis.enabled` | true | Enable Redis cache | "Helm Chart Values" — ops/helm/application-application/values.yaml — retrieved 2025-01-09 |

## Environment Variables Set via Helm

**Application Configuration:**
- `APP_NAME`: firefly-iii-application
- `APP_ENV`: production
- `APP_DEBUG`: false
- `APP_URL`: Uses shared hostname configuration
- `APP_LOCALE`: en
- `APP_TIMEZONE`: Europe/Amsterdam

**Database Configuration:**
- `DB_CONNECTION`: mysql
- `DB_HOST`: firefly-iii-application-mariadb
- `DB_PORT`: 3306
- `DB_DATABASE`: firefly-iii-application
- `DB_USERNAME`: firefly-iii-application

**Cache Configuration:**
- `SESSION_DRIVER`: redis
- `CACHE_DRIVER`: redis
- `QUEUE_CONNECTION`: redis
- `REDIS_HOST`: firefly-iii-application-redis-master
- `REDIS_PORT`: 6379
- `REDIS_PREFIX`: firefly-iii-application_

**Security Configuration:**
- `REQUIRE_HTTPS`: true
- `TRUSTED_PROXIES`: ** (trust all proxies)
- `DB_STRICT`: false

**Additional Configuration:**
- `IS_DOCKER`: true (indicates container environment)
- `FILESYSTEM_DRIVER`: local (file storage backend)
- `EXPANDED_LOGGING`: true (enhanced logging for troubleshooting)
- `PDF_GENERATOR`: snappdf (PDF generation engine)

**Secrets (referenced from Kubernetes secrets):**
- `APP_KEY`: Application encryption key used for Laravel encryption and CSRF protection
- `DB_PASSWORD`: Database password for MariaDB connection
- `REDIS_PASSWORD`: Password for Redis cache and session storage
- `API_SECRET`: Secret token for API authentication and external integrations
- `UPDATE_SECRET`: Security token for webhook authentication and automated update notifications
- `WEBCRON_SECRET`: Authentication token for scheduled task execution and cron job webhooks

## Sources
- "Helm Chart Values" — ops/helm/application-application/values.yaml — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"ops/helm/application-application/values.yaml":""},"sections":{"helm-values":""}}}
-->