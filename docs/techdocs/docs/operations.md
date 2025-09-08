# Operations

**Purpose**: This document provides comprehensive operational procedures for managing the Firefly III application in development and production environments, including deployment, monitoring, backup, and maintenance tasks.

## Table of Contents

- [Daily Operations](#daily-operations)
- [Deployment Procedures](#deployment-procedures)
- [Backup and Recovery](#backup-and-recovery)
- [Monitoring and Health Checks](#monitoring-and-health-checks)
- [Maintenance Tasks](#maintenance-tasks)
- [Upgrade Procedures](#upgrade-procedures)
- [Rollback Procedures](#rollback-procedures)

## Daily Operations

### Service Management

**Start Services**:
```bash
make start
```

**Stop Services**:
```bash
make stop
```

**View Service Status**:
```bash
docker compose ps
```

**Restart Specific Service**:
```bash
docker compose restart firefly-iii-application.application
```

### Log Management

**View All Logs**:
```bash
make logs
```

**View Specific Service Logs**:
```bash
make logs SERVICE=firefly-iii-application.application
```

**Follow Real-time Logs**:
```bash
docker compose logs -f firefly-iii-application.application
```

**Log Rotation**:
Logs are automatically rotated by Docker. Configure log rotation in production:
```yaml
logging:
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"
```

### Health Monitoring

**Quick Health Check**:
```bash
# Check application health
curl -f http://localhost:8080/health

# Check all services
docker compose ps
```

**Service Health Details**:
```bash
# Check individual service health
docker inspect firefly-iii-application.mariadb --format='{{.State.Health.Status}}'
```

## Deployment Procedures

### Local Development Deployment

**Initial Deployment**:
```bash
# 1. Clone repository
git clone <repository-url>
cd firefly-iii-application

# 2. Configure environment
cp .env.example .env
# Edit .env with appropriate values

# 3. Start services
make start

# 4. Wait for initialization
make wait-ready URL=http://localhost:8080/health

# 5. Create initial user
make user:create EMAIL=admin@example.com PASS=securepassword
```

### Production Deployment

**Pre-deployment Checklist**:
- [ ] Environment variables configured for production
- [ ] Database backup completed
- [ ] SSL certificates prepared
- [ ] DNS configuration updated
- [ ] Firewall rules configured
- [ ] Monitoring alerts configured

**Production Deployment Steps**:
```bash
# 1. Update environment for production
APP_ENV=production
APP_DEBUG=false
REQUIRE_HTTPS=true

# 2. Set secure secrets
APP_KEY=<generated-key>
API_SECRET=<random-secret>
UPDATE_SECRET=<random-secret>
WEBCRON_SECRET=<random-secret>

# 3. Configure database
DB_HOST=<production-db-host>
DB_PASSWORD=<secure-password>

# 4. Deploy with production settings
make start

# 5. Verify deployment
make wait-ready URL=https://your-domain.com/health
```

### Container Image Management

**Image Policy**: This repository builds and uses only organization-owned images:
- `webgrip/firefly-iii-application:latest`
- `webgrip/firefly-iii-application-nginx:latest`
- `webgrip/firefly-iii-application-mariadb:latest`
- `webgrip/firefly-iii-application-redis:latest`

**Image Building**:
```bash
# Build all images
docker compose build

# Build specific image
docker compose build firefly-iii-application.application

# Tag for registry
docker tag webgrip/firefly-iii-application:latest webgrip/firefly-iii-application:v1.0.0
```

**Image Registry Operations**:
```bash
# Push to registry
docker push webgrip/firefly-iii-application:v1.0.0

# Pull latest images
docker compose pull
```

## Backup and Recovery

### Automated Backup Strategy

**Database Backup**:
```bash
# Create database backup
docker compose exec firefly-iii-application.mariadb \
  mysqldump -u root -p${DB_ROOT_PASSWORD} ${DB_DATABASE} > backup-$(date +%Y%m%d).sql

# Automated daily backup script
#!/bin/bash
BACKUP_DIR="/var/backups/firefly-iii"
DATE=$(date +%Y%m%d-%H%M%S)
mkdir -p $BACKUP_DIR

docker compose exec firefly-iii-application.mariadb \
  mysqldump -u root -p${DB_ROOT_PASSWORD} ${DB_DATABASE} > $BACKUP_DIR/db-$DATE.sql

# Backup application data
docker run --rm -v firefly-iii-application-application-storage-data:/data \
  -v $BACKUP_DIR:/backup alpine tar czf /backup/storage-$DATE.tar.gz -C /data .
```

**Volume Backup**:
```bash
# Backup all persistent volumes
for volume in firefly-iii-application-mariadb-data \
              firefly-iii-application-application-storage-data \
              firefly-iii-application-application-public-data \
              firefly-iii-application-redis-data; do
  docker run --rm -v $volume:/data -v $(pwd)/backups:/backup \
    alpine tar czf /backup/$volume-$(date +%Y%m%d).tar.gz -C /data .
done
```

### Recovery Procedures

**Database Recovery**:
```bash
# Stop application
docker compose stop firefly-iii-application.application

# Restore database
docker compose exec firefly-iii-application.mariadb \
  mysql -u root -p${DB_ROOT_PASSWORD} ${DB_DATABASE} < backup-20240115.sql

# Start application
docker compose start firefly-iii-application.application
```

**Volume Recovery**:
```bash
# Stop services
make stop

# Restore volume
docker run --rm -v firefly-iii-application-mariadb-data:/data \
  -v $(pwd)/backups:/backup alpine \
  tar xzf /backup/firefly-iii-application-mariadb-data-20240115.tar.gz -C /data

# Start services
make start
```

### Backup Retention Policy

**Recommended Schedule**:
- **Daily**: Database backup (retain 30 days)
- **Weekly**: Full volume backup (retain 12 weeks)
- **Monthly**: Complete system backup (retain 12 months)
- **Before upgrades**: Complete backup (retain until upgrade verified)

## Monitoring and Health Checks

### Application Monitoring

**Health Endpoints**:
- Application health: `http://localhost:8080/health`
- Nginx health: `http://localhost:8080/health`
- Database: Connection test via application

**Key Metrics to Monitor**:
- HTTP response times
- Database connection pool status
- Redis memory usage
- Disk space usage
- Container CPU and memory usage

**Monitoring Script Example**:
```bash
#!/bin/bash
# Basic monitoring script

# Check HTTP response
if ! curl -f -s http://localhost:8080/health > /dev/null; then
  echo "ALERT: Application health check failed"
fi

# Check container status
if ! docker compose ps | grep -q "Up"; then
  echo "ALERT: One or more containers are down"
fi

# Check disk space
if [ $(df /var/lib/docker | awk 'NR==2 {print $5}' | sed 's/%//') -gt 80 ]; then
  echo "ALERT: Docker disk usage over 80%"
fi
```

### Log Monitoring

**Error Detection**:
```bash
# Monitor for errors in logs
docker compose logs firefly-iii-application.application | grep -i error

# Monitor database errors
docker compose logs firefly-iii-application.mariadb | grep -i error

# Watch for specific patterns
docker compose logs -f firefly-iii-application.application | grep -E "(ERROR|CRITICAL|FATAL)"
```

## Maintenance Tasks

### Database Maintenance

**Weekly Tasks**:
```bash
# Optimize database tables
docker compose exec firefly-iii-application.mariadb \
  mysql -u root -p${DB_ROOT_PASSWORD} -e "OPTIMIZE TABLE ${DB_DATABASE}.*;"

# Check database integrity
docker compose exec firefly-iii-application.mariadb \
  mysql -u root -p${DB_ROOT_PASSWORD} -e "CHECK TABLE ${DB_DATABASE}.*;"
```

**Monthly Tasks**:
```bash
# Analyze table statistics
docker compose exec firefly-iii-application.mariadb \
  mysql -u root -p${DB_ROOT_PASSWORD} -e "ANALYZE TABLE ${DB_DATABASE}.*;"

# Review slow query log
docker compose exec firefly-iii-application.mariadb \
  mysql -u root -p${DB_ROOT_PASSWORD} -e "SELECT * FROM mysql.slow_log LIMIT 10;"
```

### Cache Maintenance

**Redis Cache Management**:
```bash
# Clear application cache
make run CMD="php artisan cache:clear"

# Clear specific cache keys
docker compose exec firefly-iii-application.redis redis-cli FLUSHDB

# Monitor Redis memory usage
docker compose exec firefly-iii-application.redis redis-cli INFO memory
```

### Container Maintenance

**Image Updates**:
```bash
# Pull latest base images
docker compose pull

# Rebuild with latest base images
docker compose build --no-cache

# Remove unused images
docker image prune -f
```

**Volume Cleanup**:
```bash
# Remove unused volumes (be careful!)
docker volume prune

# Check volume usage
docker system df
```

## Upgrade Procedures

### Application Upgrades

**Pre-upgrade Checklist**:
- [ ] Current backup completed
- [ ] Read upstream release notes
- [ ] Test upgrade in staging environment
- [ ] Plan rollback procedure
- [ ] Schedule maintenance window

**Upgrade Steps**:
```bash
# 1. Create backup
make backup

# 2. Pull latest images
docker compose pull

# 3. Stop services
make stop

# 4. Start with new images
make start

# 5. Run database migrations (if required)
make run CMD="php artisan migrate"

# 6. Verify application
make wait-ready URL=http://localhost:8080/health

# 7. Test functionality
curl -s http://localhost:8080/api/version
```

### Version Management

**Tagging Strategy**:
- Use semantic versioning for releases
- Tag images with specific versions
- Maintain latest tag for development

**Version Verification**:
```bash
# Check application version
make run CMD="php artisan --version"

# Check container image versions
docker compose images
```

## Rollback Procedures

### Quick Rollback

**Emergency Rollback** (when current version fails):
```bash
# 1. Stop current services
make stop

# 2. Restore from backup
docker run --rm -v firefly-iii-application-mariadb-data:/data \
  -v $(pwd)/backups:/backup alpine \
  tar xzf /backup/db-backup-previous.tar.gz -C /data

# 3. Use previous image version
docker compose pull <previous-version>

# 4. Start services
make start

# 5. Verify functionality
make wait-ready URL=http://localhost:8080/health
```

### Planned Rollback

**Controlled Rollback**:
```bash
# 1. Document reason for rollback
echo "Rollback reason: $(date)" >> rollback.log

# 2. Create current state backup (in case rollback fails)
make backup

# 3. Stop services
make stop

# 4. Switch to previous version
git checkout <previous-release-tag>

# 5. Restore data if needed
# (only if database schema changes occurred)

# 6. Start services
make start

# 7. Verify and test
make wait-ready URL=http://localhost:8080/health
```

### Rollback Verification

**Post-rollback Testing**:
- [ ] Application loads correctly
- [ ] User authentication works
- [ ] Database queries function
- [ ] File uploads work
- [ ] API endpoints respond
- [ ] Background jobs process

---

**Sources**:
- Firefly III Maintenance Guide - https://docs.firefly-iii.org/how-to/firefly-iii/upgrade/ (Retrieved: 2024-01-15)
- Docker Operations Best Practices - https://docs.docker.com/config/containers/logging/ (Retrieved: 2024-01-15)
- Laravel Deployment Documentation - https://laravel.com/docs/11.x/deployment (Retrieved: 2024-01-15)