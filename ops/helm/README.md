# Firefly III Application - Helm Charts

This directory contains Helm charts for deploying the Firefly III personal finance management application to Kubernetes.

## Chart Structure

### Main Application Chart
- **Location**: `firefly-iii-application/`
- **Purpose**: Deploys the complete Firefly III application stack
- **Dependencies**: Uses `bjw-s/app-template` v4.2.0 with optional Redis and MariaDB

### Secrets Chart
- **Location**: `../secrets/firefly-iii-application-secrets/`
- **Purpose**: Manages application secrets separately for security
- **Requirement**: Must be installed first before main application

## Prerequisites

- Kubernetes cluster with Traefik ingress controller
- Helm 3.x installed
- kubectl configured for your cluster

## Installation

### 1. Configure Secrets

First, create your secrets configuration:

```bash
cp ops/secrets/firefly-iii-application-secrets/values.dec.yaml.example \
   ops/secrets/firefly-iii-application-secrets/values.dec.yaml
```

Edit the `values.dec.yaml` file and configure all required secrets:

```yaml
namespace: firefly-iii-application

# Database secrets
db-password: "your-secure-db-password"
db-root-password: "your-secure-root-password"
redis-password: "your-secure-redis-password"

# Firefly III application secrets
app-key: "base64:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
api-secret: "your-api-secret-for-external-integrations"
update-secret: "your-update-secret-for-webhook-authentication"
webcron-secret: "your-webcron-secret-for-scheduled-tasks"
user-password: "your-secure-admin-password"
```

### 2. Install Secrets Chart

```bash
helm upgrade --install firefly-iii-application-secrets \
  ops/secrets/firefly-iii-application-secrets \
  --values ops/secrets/firefly-iii-application-secrets/values.dec.yaml \
  -n firefly-iii-application --create-namespace
```

### 3. Configure Application

Edit `ops/helm/firefly-iii-application/values.yaml` to customize:

- Hostname and URL for your environment
- Resource limits and requests
- Storage class and sizes
- Ingress configuration

### 4. Install Main Application

```bash
helm upgrade --install firefly-iii-application \
  ops/helm/firefly-iii-application \
  -n firefly-iii-application
```

## Secrets Reference

| Secret | Purpose | Required |
|--------|---------|----------|
| `app-key` | Laravel application encryption key | ✅ |
| `db-password` | Database user password | ✅ |
| `db-root-password` | Database root password | ✅ |
| `redis-password` | Redis authentication password | ✅ |
| `api-secret` | API authentication for external integrations | ✅ |
| `update-secret` | Webhook authentication for update notifications | ✅ |
| `webcron-secret` | Authentication for scheduled task execution | ✅ |
| `user-password` | Initial admin user password | ✅ |

### Generating Secrets

**APP_KEY**: Generate a base64 encoded 32-character string:
```bash
echo "base64:$(openssl rand -base64 32)"
```

**Other secrets**: Generate random secure strings:
```bash
openssl rand -base64 32
```

## Environment Variables

The application configures these key environment variables:

### Core Application
- `APP_NAME`: firefly-iii-application
- `APP_ENV`: production
- `APP_DEBUG`: false
- `APP_URL`: Your configured external URL
- `APP_TIMEZONE`: Europe/Amsterdam

### Database Configuration
- `DB_CONNECTION`: mysql
- `DB_HOST`: firefly-iii-application-mariadb
- `DB_DATABASE`: firefly-iii-application
- `DB_USERNAME`: firefly-iii-application

### Cache and Sessions
- `SESSION_DRIVER`: redis
- `CACHE_DRIVER`: redis
- `QUEUE_CONNECTION`: redis
- `REDIS_HOST`: firefly-iii-application-redis-master

### Security
- `REQUIRE_HTTPS`: true
- `TRUSTED_PROXIES`: ** (trust all proxies in Kubernetes)

## Dependencies

The chart uses these Helm chart dependencies:

| Chart | Version | Repository | Purpose |
|-------|---------|------------|---------|
| `bjw-s/app-template` | 4.2.0 | https://bjw-s-labs.github.io/helm-charts | Main application deployment |
| `bitnami/redis` | 22.0.7 | https://charts.bitnami.com/bitnami | Cache and session storage |
| `bitnami/mariadb` | 22.0.0 | https://charts.bitnami.com/bitnami | Database backend |

## Ingress Configuration

The chart is configured for Traefik ingress controller with:

- Automatic HTTPS redirect middleware
- Host-based routing
- TLS termination at ingress

## Storage

The application requires persistent storage for:

- **Application uploads**: 10Gi persistent volume
- **Database data**: 20Gi persistent volume  
- **Redis data**: 8Gi persistent volume

## Monitoring and Health Checks

The application includes:

- **Readiness probe**: HTTP check on `/` (port 8080)
- **Liveness probe**: HTTP check on `/` (port 8080)
- **Resource limits**: CPU and memory constraints

## Troubleshooting

### Check Pod Status
```bash
kubectl get pods -n firefly-iii-application
```

### View Logs
```bash
kubectl logs -f deployment/firefly-iii-application -n firefly-iii-application
```

### Check Secrets
```bash
kubectl get secrets -n firefly-iii-application
kubectl describe secret firefly-iii-application-secrets -n firefly-iii-application
```

### Verify Configuration
```bash
kubectl describe deployment firefly-iii-application -n firefly-iii-application
```

## Upgrading

To upgrade the application:

1. Update the image tag in `values.yaml`
2. Run the helm upgrade command:

```bash
helm upgrade firefly-iii-application \
  ops/helm/firefly-iii-application \
  -n firefly-iii-application
```

## Uninstalling

To remove the application:

```bash
# Remove main application
helm uninstall firefly-iii-application -n firefly-iii-application

# Remove secrets (this will delete all data!)
helm uninstall firefly-iii-application-secrets -n firefly-iii-application

# Remove namespace
kubectl delete namespace firefly-iii-application
```

⚠️ **Warning**: This will permanently delete all data!