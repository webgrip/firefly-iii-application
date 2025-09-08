---
title: "Helm — Used Values"
description: "Only the values from values.yaml that this repo actually uses."
tags: [helm, configuration, kubernetes]
search: { boost: 2, exclude: false }
icon: material/helm
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Document the Helm chart values actually used in this Invoice Ninja deployment for Kubernetes environments.

**Contents**
- [Chart Configuration](#chart-configuration)
- [Application Container](#application-container)
- [Web Server Container](#web-server-container)
- [Database Configuration](#database-configuration)
- [Redis Configuration](#redis-configuration)
- [Networking and Ingress](#networking-and-ingress)
- [Storage and Persistence](#storage-and-persistence)
- [Sources](#sources)

## Chart Configuration

### Global Settings
| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `namespace` | `firefly-iii-application` | Kubernetes namespace for all resources | Cluster organization |
| `_shared_config.hostname` | `firefly-iii-application.staging.k8s.webgrip.nl` | FQDN for external access | DNS configuration |
| `_shared_config.url` | `https://firefly-iii-application.staging.k8s.webgrip.nl` | Complete application URL with protocol | Ingress and SSL |

**Configuration Pattern:**
```yaml
namespace: firefly-iii-application

_shared_config:
  hostname: &hostname firefly-iii-application.staging.k8s.webgrip.nl
  url: &url https://firefly-iii-application.staging.k8s.webgrip.nl
```

## Application Container

### Image Configuration
| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `application.controllers.main.containers.app.image.repository` | `docker.io/webgrip/firefly-iii-application` | Custom built Invoice Ninja image | WebGrip registry |
| `application.controllers.main.containers.app.image.tag` | `"latest"` | Use specific version tags for production | Container versioning |
| `application.controllers.main.containers.app.image.pullPolicy` | `Always` | Ensure latest image version | Development setting |

### Security Context
| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `application.controllers.main.pod.securityContext.runAsNonRoot` | `true` | Security requirement: no root execution | Security policy |
| `application.controllers.main.pod.securityContext.fsGroup` | `1500` | File system group ID | User management |
| `application.controllers.main.containers.app.securityContext.runAsUser` | `1500` | Non-root user ID | Security hardening |
| `application.controllers.main.containers.app.securityContext.allowPrivilegeEscalation` | `false` | Prevent privilege escalation | Security hardening |

### Environment Variables
Critical application environment configuration:

| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `env[].name: APP_ENV` | `production` | Laravel environment mode | Laravel framework |
| `env[].name: APP_DEBUG` | `"false"` | Disable debug in production | Security requirement |
| `env[].name: APP_URL` | Uses `*url` reference | Must match ingress configuration | URL consistency |
| `env[].name: APP_TIMEZONE` | `Europe/Amsterdam` | Application timezone | Localization |
| `env[].name: REQUIRE_HTTPS` | `"true"` | Force HTTPS redirects | Security requirement |
| `env[].name: TRUSTED_PROXIES` | `"**"` | Trust all proxies (behind ingress) | Network configuration |

### Resource Management
| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `resources.requests.cpu` | `"250m"` | Guaranteed CPU allocation | Performance planning |
| `resources.requests.memory` | `"512Mi"` | Guaranteed memory allocation | Performance planning |
| `resources.limits.cpu` | `"1000m"` | Maximum CPU usage | Resource limiting |
| `resources.limits.memory` | `"1Gi"` | Maximum memory usage | Resource limiting |

### Health Checks
Application readiness probe configuration:

```yaml
probes:
  readiness:
    enabled: true
    custom: true
    spec:
      exec:
        command:
          - sh
          - -lc
          - >
            php -v >/dev/null &&
            nc -z 127.0.0.1 9000 &&
            nc -z firefly-iii-application-mariadb 3306 &&
            nc -z firefly-iii-application-redis-master 6379
      initialDelaySeconds: 10
      periodSeconds: 10
```

**Health Check Components:**
- **PHP Runtime**: Verify PHP interpreter is functional
- **FastCGI**: Check PHP-FPM is accepting connections on port 9000
- **Database**: Verify MariaDB connectivity on port 3306
- **Cache**: Verify Redis connectivity on port 6379

## Web Server Container

### Nginx Configuration
| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `containers.web.image.repository` | `nginxinc/nginx-unprivileged` | Official unprivileged Nginx image | Security requirement |
| `containers.web.image.tag` | `1.29.1-alpine3.22-perl` | Specific version with Alpine base | Version pinning |
| `containers.web.securityContext.runAsUser` | `101` | Nginx unprivileged user | Security hardening |
| `containers.web.securityContext.readOnlyRootFilesystem` | `true` | Immutable root filesystem | Security hardening |

### Web Server Resources
| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `resources.requests.cpu` | `"100m"` | Minimal CPU for static serving | Resource efficiency |
| `resources.requests.memory` | `"128Mi"` | Minimal memory footprint | Resource efficiency |
| `resources.limits.cpu` | `"300m"` | Maximum CPU for web serving | Performance limit |
| `resources.limits.memory` | `"256Mi"` | Maximum memory for web serving | Performance limit |

### Nginx Health Checks
| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `probes.liveness.spec.httpGet.path` | `/health` | Simple health endpoint | Nginx configuration |
| `probes.liveness.spec.httpGet.port` | `http` | Uses named port reference | Service mapping |
| `probes.liveness.initialDelaySeconds` | `5` | Quick startup for web server | Performance tuning |
| `probes.readiness.initialDelaySeconds` | `2` | Fast readiness detection | Performance tuning |

## Database Configuration

### MariaDB Settings
| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `mariadb.enabled` | `true` | Enable MariaDB subchart | Database requirement |
| `mariadb.image.repository` | `bitnamilegacy/mariadb` | Bitnami MariaDB image | Upstream dependency |
| `mariadb.image.tag` | `"12.0.2"` | Specific MariaDB version | Version pinning |
| `mariadb.architecture` | `standalone` | Single instance deployment | Deployment model |
| `mariadb.auth.database` | `firefly-iii-application` | Database name | Application requirement |
| `mariadb.auth.username` | `firefly-iii-application` | Database user | Application requirement |
| `mariadb.auth.existingSecret` | `firefly-iii-application-secrets` | Secret for credentials | Security requirement |

### Database Resources
| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `mariadb.primary.resources.requests.cpu` | `"100m"` | Guaranteed CPU for database | Performance planning |
| `mariadb.primary.resources.requests.memory` | `"256Mi"` | Guaranteed memory for database | Performance planning |
| `mariadb.primary.resources.limits.cpu` | `"300m"` | Maximum database CPU | Resource limiting |
| `mariadb.primary.resources.limits.memory` | `"512Mi"` | Maximum database memory | Resource limiting |

### Database Storage
| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `mariadb.primary.persistence.enabled` | `true` | Persistent storage required | Data durability |
| `mariadb.primary.persistence.size` | `2Gi` | Database storage allocation | Capacity planning |
| `mariadb.primary.persistence.storageClass` | `do-block-storage` | DigitalOcean block storage | Cloud provider |

## Redis Configuration

### Redis Settings
| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `redis.enabled` | `true` | Enable Redis subchart | Cache requirement |
| `redis.image.repository` | `bitnamilegacy/redis` | Bitnami Redis image | Upstream dependency |
| `redis.image.tag` | `"8.2.1"` | Specific Redis version | Version pinning |
| `redis.architecture` | `standalone` | Single instance deployment | Deployment model |
| `redis.auth.enabled` | `true` | Enable Redis authentication | Security requirement |
| `redis.auth.existingSecret` | `firefly-iii-application-secrets` | Secret for credentials | Security requirement |

### Redis Security
| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `redis.master.disableCommands` | `[FLUSHALL, CONFIG, KEYS]` | Disable dangerous commands | Security hardening |
| `redis.master.persistence.enabled` | `true` | Persistent Redis data | Data durability |
| `redis.master.persistence.size` | `1Gi` | Redis storage allocation | Capacity planning |

### Redis Resources
| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `redis.resources.requests.cpu` | `"50m"` | Minimal CPU for cache | Resource efficiency |
| `redis.resources.requests.memory` | `"128Mi"` | Minimal memory for cache | Resource efficiency |
| `redis.resources.limits.cpu` | `"150m"` | Maximum cache CPU | Performance limit |
| `redis.resources.limits.memory` | `"256Mi"` | Maximum cache memory | Performance limit |

## Networking and Ingress

### Service Configuration
| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `service.main.controller` | `main` | Links service to controller | Kubernetes mapping |
| `service.main.ports.http.port` | `80` | External service port | Standard HTTP |
| `service.main.ports.http.targetPort` | `http` | Maps to container port name | Port mapping |

### Ingress Configuration
| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `ingress.main.enabled` | `true` | Enable external access | Internet exposure |
| `ingress.main.className` | `ingress-traefik` | Traefik ingress controller | Load balancer |
| `ingress.main.annotations["cert-manager.io/cluster-issuer"]` | `letsencrypt-traefik` | Automatic TLS certificates | SSL automation |
| `ingress.main.hosts[0].host` | Uses `*hostname` reference | Domain name for access | DNS configuration |
| `ingress.main.tls[0].secretName` | `letsencrypt-firefly-iii-application` | TLS certificate secret | SSL storage |

## Storage and Persistence

### Application Storage
| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `persistence.public.type` | `persistentVolumeClaim` | Kubernetes PVC | Storage type |
| `persistence.public.size` | `1Gi` | Public files storage | Capacity planning |
| `persistence.public.storageClass` | `do-block-storage` | DigitalOcean block storage | Cloud provider |
| `persistence.storage.size` | `2Gi` | Application data storage | Capacity planning |

### Volume Mounts
Application storage is mounted to both containers:

**Application Container Mounts:**
- `/var/www/app/public` - Public web files (read-write)
- `/var/www/app/storage` - Application data (read-write)

**Web Server Container Mounts:**
- `/var/www/app/public` - Public web files (read-only)
- `/var/www/app/storage` - Application data (read-only)

### ConfigMap Storage
| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `persistence.nginx-conf.type` | `configMap` | Nginx configuration | Configuration management |
| `persistence.nginx-conf.name` | `firefly-iii-application` | ConfigMap name | Resource naming |
| `persistence.nginx-tmp.type` | `emptyDir` | Temporary Nginx files | Runtime storage |

## Container Interface

| Aspect | Value / Path | Notes |
|-------|---------------|-------|
| Application Port | 9000 (FastCGI) | PHP-FPM internal port |
| Web Server Port | 8080 → 80 | Nginx serves on 8080, exposed as 80 |
| Health Endpoints | `/health` (Nginx), Custom exec (App) | Liveness and readiness |
| Storage Mounts | `/var/www/app/public`, `/var/www/app/storage` | Persistent data |
| Configuration | Environment variables + ConfigMap | Externalized config |
| Security | Non-root users, read-only filesystems | Hardened containers |

## Sources

- "Helm Chart Values Reference" — https://helm.sh/docs/chart_template_guide/values_files/ — retrieved 2025-01-09
- "Kubernetes Resource Management" — https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/ — retrieved 2025-01-09
- "Nginx Unprivileged Container" — https://hub.docker.com/r/nginxinc/nginx-unprivileged — retrieved 2025-01-09
- "Bitnami MariaDB Helm Chart" — https://github.com/bitnami/charts/tree/main/bitnami/mariadb — retrieved 2025-01-09
- "Bitnami Redis Helm Chart" — https://github.com/bitnami/charts/tree/main/bitnami/redis — retrieved 2025-01-09