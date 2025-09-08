# Deployment

**Purpose**: This document provides comprehensive guidance for deploying the Firefly III application in various environments, from local development to production cloud deployments.

## Table of Contents

- [Deployment Overview](#deployment-overview)
- [Local Development Deployment](#local-development-deployment)
- [Staging Environment](#staging-environment)
- [Production Deployment](#production-deployment)
- [Cloud Deployment Options](#cloud-deployment-options)
- [Container Registry Management](#container-registry-management)
- [Environment-Specific Configuration](#environment-specific-configuration)
- [Monitoring and Health Checks](#monitoring-and-health-checks)

## Deployment Overview

### Deployment Environments

| Environment | Purpose | Configuration | Access |
|-------------|---------|---------------|--------|
| **Development** | Local development and testing | Minimal security, debug enabled | Localhost only |
| **Staging** | Pre-production testing | Production-like, test data | Internal access |
| **Production** | Live application | Maximum security, performance optimized | Public access |

### Image Policy

**Organization-Owned Images**: This repository builds and maintains all container images:
- `webgrip/firefly-iii-application:latest` - Main application
- `webgrip/firefly-iii-application-nginx:latest` - Web server
- `webgrip/firefly-iii-application-mariadb:latest` - Database
- `webgrip/firefly-iii-application-redis:latest` - Cache/sessions

**Versioning Strategy**:
- `latest` - Development builds
- `v1.2.3` - Semantic versioned releases
- `main-abc123` - Git commit-based tags

## Local Development Deployment

### Prerequisites

```bash
# Required software
docker --version          # 20.10+
docker compose version    # 2.0+
make --version            # GNU Make
git --version             # Git

# System requirements
# - 4GB RAM minimum
# - 10GB disk space
# - Modern CPU (x64 or ARM64)
```

### Quick Start

```bash
# 1. Clone repository
git clone https://github.com/webgrip/firefly-iii-application.git
cd firefly-iii-application

# 2. Environment setup
cp .env.example .env

# 3. Generate application key
echo "APP_KEY=base64:$(openssl rand -base64 32)" >> .env

# 4. Start services
make start

# 5. Verify deployment
make wait-ready URL=http://localhost:8080/health

# 6. Access application
open http://localhost:8080
```

### Development Configuration

**Environment Variables** (`.env`):
```env
# Development-specific settings
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8080
BASE_URL=http://localhost:8080

# Local database settings
DB_HOST=firefly-iii-application.mariadb
DB_CONNECTION=mysql
DB_DATABASE=application
DB_USERNAME=application
DB_PASSWORD=application

# Development mail (logs only)
MAIL_MAILER=log
```

**Development Commands**:
```bash
# Start in development mode
make start

# View logs in real-time
make logs

# Access application container
make enter

# Run application commands
make run CMD="php artisan --version"

# Create test user
make user:create EMAIL=dev@example.com PASS=password
```

## Staging Environment

### Staging Setup

**Purpose**: Staging environment mirrors production for final testing:
- Production-like configuration
- Real SSL certificates
- Performance testing
- Integration testing

**Configuration Differences from Development**:
```env
# Staging-specific settings
APP_ENV=staging
APP_DEBUG=false
APP_URL=https://staging.yourdomain.com
BASE_URL=https://staging.yourdomain.com

# Production-like database
DB_HOST=staging-db.internal
DB_PASSWORD=secure_staging_password

# Real mail service
MAIL_MAILER=smtp
MAIL_HOST=smtp.yourdomain.com
MAIL_PORT=587
MAIL_ENCRYPTION=tls
```

**Staging Deployment Process**:
```bash
# 1. Deploy to staging server
scp -r . staging-server:/opt/firefly-iii-staging/

# 2. SSH to staging server
ssh staging-server

# 3. Configure environment
cd /opt/firefly-iii-staging
cp .env.staging .env

# 4. Deploy with staging configuration
make start

# 5. Run staging tests
./scripts/staging-tests.sh

# 6. Verify functionality
curl -f https://staging.yourdomain.com/health
```

## Production Deployment

### Pre-Production Checklist

**Infrastructure Requirements**:
- [ ] Server with 8GB+ RAM, 4+ CPU cores
- [ ] 50GB+ SSD storage
- [ ] SSL certificates configured
- [ ] Firewall rules configured
- [ ] Backup solution in place
- [ ] Monitoring solution configured
- [ ] Database backup strategy defined

**Security Requirements**:
- [ ] All default passwords changed
- [ ] API secrets generated
- [ ] Database access restricted
- [ ] HTTPS enforced
- [ ] Security headers configured
- [ ] Access logs enabled
- [ ] Intrusion detection configured

### Production Environment Configuration

**Critical Environment Variables**:
```env
# Production application settings
APP_ENV=production
APP_DEBUG=false
APP_URL=https://yourdomain.com
BASE_URL=https://yourdomain.com

# Security settings
REQUIRE_HTTPS=true
SESSION_SECURE=true
SESSION_ENCRYPT=true
SESSION_DOMAIN=yourdomain.com

# Strong secrets (generate unique values)
APP_KEY=base64:$(openssl rand -base64 32)
API_SECRET=$(openssl rand -hex 32)
UPDATE_SECRET=$(openssl rand -hex 32)
WEBCRON_SECRET=$(openssl rand -hex 16)

# Production database
DB_HOST=prod-db.internal
DB_PASSWORD=very_secure_production_password

# Production mail service
MAIL_MAILER=smtp
MAIL_HOST=smtp.yourdomain.com
MAIL_PORT=587
MAIL_ENCRYPTION=tls
MAIL_USERNAME=noreply@yourdomain.com
MAIL_PASSWORD=mail_service_password
```

### Production Deployment Process

**Step 1: Server Preparation**
```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Create application directory
sudo mkdir -p /opt/firefly-iii
sudo chown $USER:$USER /opt/firefly-iii
```

**Step 2: Application Deployment**
```bash
# Clone application
cd /opt/firefly-iii
git clone https://github.com/webgrip/firefly-iii-application.git .

# Configure production environment
cp .env.example .env
# Edit .env with production values

# Set up secrets management
make secrets:init
# Configure secrets in ops/secrets/firefly-iii-application-secrets/values.dec.yaml
make secrets:encrypt SECRETS_DIR=./ops/secrets/firefly-iii-application-secrets
```

**Step 3: SSL/TLS Configuration**
```bash
# Install Certbot for Let's Encrypt
sudo apt install certbot python3-certbot-nginx

# Obtain SSL certificate
sudo certbot certonly --standalone -d yourdomain.com

# Configure Nginx for HTTPS
# Update ops/docker/nginx/config/default.conf with SSL configuration
```

**Step 4: Production Deployment**
```bash
# Build production images
docker compose build

# Start services
docker compose up -d

# Wait for initialization
make wait-ready URL=https://yourdomain.com/health

# Create admin user
make user:create EMAIL=admin@yourdomain.com PASS=secure_admin_password

# Verify deployment
curl -f https://yourdomain.com/api/about
```

## Cloud Deployment Options

### AWS Deployment

**ECS Deployment**:
```bash
# Install AWS CLI and ECS CLI
pip install awscli
ecs-cli configure --cluster firefly-iii --region us-west-2

# Create ECS task definition
# Use docker-compose.yml as base for ECS task definition

# Deploy to ECS
ecs-cli compose --project-name firefly-iii service up
```

**Elastic Beanstalk Deployment**:
```bash
# Install EB CLI
pip install awsebcli

# Initialize Elastic Beanstalk application
eb init firefly-iii --platform docker

# Deploy application
eb create production
eb deploy
```

### Azure Deployment

**Container Instances**:
```bash
# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Login to Azure
az login

# Create resource group
az group create --name firefly-iii-rg --location eastus

# Deploy container group
az container create \
  --resource-group firefly-iii-rg \
  --name firefly-iii-app \
  --image webgrip/firefly-iii-application:latest \
  --dns-name-label firefly-iii-unique \
  --ports 80
```

### Google Cloud Deployment

**Cloud Run Deployment**:
```bash
# Install Google Cloud SDK
curl https://sdk.cloud.google.com | bash

# Authenticate
gcloud auth login

# Deploy to Cloud Run
gcloud run deploy firefly-iii \
  --image webgrip/firefly-iii-application:latest \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated
```

### Kubernetes Deployment

**Using Helm Charts**:
```bash
# Install Helm
curl https://get.helm.sh/helm-v3.10.0-linux-amd64.tar.gz | tar xz
sudo mv linux-amd64/helm /usr/local/bin/

# Deploy with Helm
cd ops/helm/firefly-iii-application
helm dependency update
helm install firefly-iii . --namespace firefly-iii --create-namespace

# Verify deployment
kubectl get pods -n firefly-iii
kubectl get services -n firefly-iii
```

## Container Registry Management

### Building and Pushing Images

**Local Build Process**:
```bash
# Build all images
docker compose build

# Tag for registry
docker tag webgrip/firefly-iii-application:latest \
  registry.yourdomain.com/firefly-iii-application:v1.0.0

# Push to registry
docker push registry.yourdomain.com/firefly-iii-application:v1.0.0
```

**CI/CD Integration**:
```yaml
# .github/workflows/build-and-deploy.yml
name: Build and Deploy
on:
  push:
    branches: [main]
    tags: ['v*']

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build images
        run: docker compose build
        
      - name: Push to registry
        run: |
          echo ${{ secrets.REGISTRY_PASSWORD }} | docker login -u ${{ secrets.REGISTRY_USERNAME }} --password-stdin
          docker compose push
```

### Registry Security

**Image Scanning**:
```bash
# Scan images for vulnerabilities
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image webgrip/firefly-iii-application:latest

# Sign images (using Docker Content Trust)
export DOCKER_CONTENT_TRUST=1
docker push webgrip/firefly-iii-application:v1.0.0
```

## Environment-Specific Configuration

### Configuration Management

**Environment Variables by Environment**:

| Variable | Development | Staging | Production |
|----------|-------------|---------|------------|
| `APP_ENV` | `local` | `staging` | `production` |
| `APP_DEBUG` | `true` | `false` | `false` |
| `MAIL_MAILER` | `log` | `smtp` | `smtp` |
| `REQUIRE_HTTPS` | `false` | `true` | `true` |
| `DB_HOST` | `firefly-iii-application.mariadb` | `staging-db.internal` | `prod-db.internal` |

**Configuration Templates**:
```bash
# Development
cp .env.example .env.development

# Staging
cp .env.example .env.staging
# Customize for staging

# Production
cp .env.example .env.production
# Customize for production with secure values
```

### Resource Allocation

**Container Resources by Environment**:

| Service | Development | Staging | Production |
|---------|-------------|---------|------------|
| **Application** | 512MB / 0.5 CPU | 1GB / 1 CPU | 2GB / 2 CPU |
| **Database** | 512MB / 0.5 CPU | 2GB / 1 CPU | 4GB / 2 CPU |
| **Redis** | 128MB / 0.1 CPU | 256MB / 0.5 CPU | 512MB / 1 CPU |
| **Nginx** | 64MB / 0.1 CPU | 128MB / 0.5 CPU | 256MB / 1 CPU |

## Monitoring and Health Checks

### Health Check Endpoints

**Application Health Checks**:
```bash
# Basic health check
curl -f http://localhost:8080/health

# Detailed health check (includes database, cache)
curl -f http://localhost:8080/health/detailed

# API health check
curl -f http://localhost:8080/api/health
```

**Database Health Checks**:
```bash
# Database connection test
docker compose exec firefly-iii-application.mariadb \
  mysql -u application -p${DB_PASSWORD} -e "SELECT 1"

# Database performance check
docker compose exec firefly-iii-application.mariadb \
  mysql -u application -p${DB_PASSWORD} -e "SHOW STATUS LIKE 'Threads%'"
```

### Monitoring Integration

**Prometheus Metrics** (if available):
```bash
# Application metrics endpoint
curl http://localhost:8080/metrics

# Custom metrics collection
# Configure in Firefly III settings
```

**Log Aggregation**:
```bash
# Configure log forwarding to centralized logging
# Example: ELK Stack, Splunk, or cloud logging services

# Structured logging configuration
LOG_CHANNEL=stack
LOG_LEVEL=info
LOG_STDERR_FORMATTER=Monolog\Formatter\JsonFormatter
```

### Deployment Verification

**Post-Deployment Checklist**:
- [ ] All containers running and healthy
- [ ] Database migrations completed successfully
- [ ] SSL certificates valid and properly configured
- [ ] Health checks passing
- [ ] Application accessible via web browser
- [ ] API endpoints responding correctly
- [ ] Mail service functioning
- [ ] Backup processes working
- [ ] Monitoring alerts configured
- [ ] Security scans completed without critical issues

**Automated Verification Script**:
```bash
#!/bin/bash
# deployment-verification.sh

set -e

echo "Starting deployment verification..."

# Check container health
if ! docker compose ps | grep -q "Up"; then
    echo "ERROR: Some containers are not running"
    exit 1
fi

# Check application health
if ! curl -f -s "${BASE_URL}/health" > /dev/null; then
    echo "ERROR: Application health check failed"
    exit 1
fi

# Check database connectivity
if ! make run CMD="php artisan tinker --execute='DB::connection()->getPdo();'" > /dev/null 2>&1; then
    echo "ERROR: Database connection failed"
    exit 1
fi

# Check API functionality
if ! curl -f -s "${BASE_URL}/api/about" > /dev/null; then
    echo "ERROR: API health check failed"
    exit 1
fi

echo "Deployment verification completed successfully!"
```

---

**Sources**:
- Firefly III Installation Documentation - https://docs.firefly-iii.org/how-to/firefly-iii/installation/ (Retrieved: 2024-01-15)
- Docker Deployment Best Practices - https://docs.docker.com/develop/dev-best-practices/ (Retrieved: 2024-01-15)
- Laravel Deployment Guide - https://laravel.com/docs/11.x/deployment (Retrieved: 2024-01-15)
- Container Security Best Practices - https://kubernetes.io/docs/concepts/security/ (Retrieved: 2024-01-15)