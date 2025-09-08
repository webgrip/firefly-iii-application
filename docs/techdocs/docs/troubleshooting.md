# Troubleshooting

**Purpose**: This document provides solutions for common issues encountered when running the Firefly III application, including diagnostic procedures and resolution steps.

## Table of Contents

- [Common Issues](#common-issues)
- [Service-Specific Issues](#service-specific-issues)
- [Performance Issues](#performance-issues)
- [Data Issues](#data-issues)
- [Network and Connectivity Issues](#network-and-connectivity-issues)
- [Diagnostic Tools](#diagnostic-tools)
- [Getting Help](#getting-help)

## Common Issues

### Application Won't Start

**Symptoms**: 
- Containers fail to start
- Error messages in logs
- Services in "Exited" status

**Diagnostic Steps**:
```bash
# Check container status
docker compose ps

# Check logs for errors
make logs

# Check specific service logs
docker compose logs firefly-iii-application.application
```

**Common Causes and Solutions**:

1. **Port Already in Use**:
   ```bash
   # Check what's using port 8080
   sudo netstat -tulpn | grep :8080
   
   # Solution: Stop conflicting service or change port
   # Edit docker-compose.yml to use different port
   ```

2. **Missing Environment Variables**:
   ```bash
   # Check for required variables
   cat .env | grep -E "(APP_KEY|DB_PASSWORD)"
   
   # Solution: Copy from .env.example and set values
   cp .env.example .env
   vim .env
   ```

3. **Docker Daemon Issues**:
   ```bash
   # Check Docker status
   systemctl status docker
   
   # Restart Docker if needed
   sudo systemctl restart docker
   ```

### Database Connection Failures

**Symptoms**:
- "Connection refused" errors
- "Access denied" errors
- Application times out during startup

**Diagnostic Steps**:
```bash
# Check MariaDB container status
docker compose ps firefly-iii-application.mariadb

# Check MariaDB logs
docker compose logs firefly-iii-application.mariadb

# Test database connection
docker compose exec firefly-iii-application.mariadb \
  mysql -u application -p${DB_PASSWORD} -e "SELECT 1"
```

**Solutions**:

1. **Container Not Ready**:
   ```bash
   # Wait for MariaDB to initialize (first run takes longer)
   docker compose logs -f firefly-iii-application.mariadb | grep "ready for connections"
   ```

2. **Wrong Credentials**:
   ```bash
   # Verify credentials in .env match docker-compose.yml
   grep -E "(DB_USERNAME|DB_PASSWORD)" .env
   ```

3. **Database Not Created**:
   ```bash
   # Create database manually if needed
   docker compose exec firefly-iii-application.mariadb \
     mysql -u root -p${DB_ROOT_PASSWORD} -e "CREATE DATABASE ${DB_DATABASE};"
   ```

### Application Errors

**Symptoms**:
- 500 Internal Server Error
- Laravel error pages
- Blank white pages

**Diagnostic Steps**:
```bash
# Check application logs
docker compose logs firefly-iii-application.application

# Check Laravel logs inside container
make enter CMD="tail -f storage/logs/laravel.log"

# Check Nginx error logs
docker compose logs firefly-iii-application.nginx
```

**Solutions**:

1. **Missing APP_KEY**:
   ```bash
   # Generate application key
   echo "APP_KEY=base64:$(openssl rand -base64 32)" >> .env
   
   # Restart application
   docker compose restart firefly-iii-application.application
   ```

2. **Permission Issues**:
   ```bash
   # Fix storage permissions
   make run CMD="chown -R www-data:www-data storage"
   make run CMD="chmod -R 775 storage"
   ```

3. **Cache Issues**:
   ```bash
   # Clear application cache
   make run CMD="php artisan cache:clear"
   make run CMD="php artisan config:clear"
   make run CMD="php artisan view:clear"
   ```

## Service-Specific Issues

### Nginx Issues

**Symptoms**:
- Cannot access application via browser
- Nginx returns 502 Bad Gateway
- Static assets not loading

**Diagnostic Steps**:
```bash
# Check Nginx status
docker compose ps firefly-iii-application.nginx

# Check Nginx configuration
docker compose exec firefly-iii-application.nginx nginx -t

# Check upstream connection
docker compose exec firefly-iii-application.nginx \
  curl -f firefly-iii-application.application:9000/status
```

**Solutions**:

1. **PHP-FPM Not Responding**:
   ```bash
   # Check PHP-FPM status
   docker compose ps firefly-iii-application.application
   
   # Restart application container
   docker compose restart firefly-iii-application.application
   ```

2. **Configuration Errors**:
   ```bash
   # Validate Nginx config
   docker compose exec firefly-iii-application.nginx nginx -t
   
   # Restart Nginx if config is valid
   docker compose restart firefly-iii-application.nginx
   ```

### Redis Issues

**Symptoms**:
- Session data lost
- Cache not working
- Connection timeouts

**Diagnostic Steps**:
```bash
# Check Redis status
docker compose ps firefly-iii-application.redis

# Test Redis connection
docker compose exec firefly-iii-application.redis redis-cli ping

# Check Redis memory usage
docker compose exec firefly-iii-application.redis redis-cli info memory
```

**Solutions**:

1. **Redis Out of Memory**:
   ```bash
   # Clear Redis cache
   docker compose exec firefly-iii-application.redis redis-cli flushdb
   
   # Increase memory limits in docker-compose.yml
   ```

2. **Connection Issues**:
   ```bash
   # Verify Redis configuration in .env
   grep REDIS_ .env
   
   # Restart Redis
   docker compose restart firefly-iii-application.redis
   ```

### MariaDB Issues

**Symptoms**:
- Slow database queries
- Connection pool exhausted
- Database corruption errors

**Diagnostic Steps**:
```bash
# Check database status
docker compose exec firefly-iii-application.mariadb \
  mysql -u root -p${DB_ROOT_PASSWORD} -e "SHOW STATUS LIKE 'Threads%'"

# Check for locked tables
docker compose exec firefly-iii-application.mariadb \
  mysql -u root -p${DB_ROOT_PASSWORD} -e "SHOW PROCESSLIST"

# Check database size
docker compose exec firefly-iii-application.mariadb \
  mysql -u root -p${DB_ROOT_PASSWORD} -e \
  "SELECT table_schema AS 'Database', ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)' FROM information_schema.tables GROUP BY table_schema"
```

**Solutions**:

1. **Connection Pool Issues**:
   ```bash
   # Increase max_connections (temporarily)
   docker compose exec firefly-iii-application.mariadb \
     mysql -u root -p${DB_ROOT_PASSWORD} -e "SET GLOBAL max_connections = 500"
   ```

2. **Slow Queries**:
   ```bash
   # Enable slow query log
   docker compose exec firefly-iii-application.mariadb \
     mysql -u root -p${DB_ROOT_PASSWORD} -e "SET GLOBAL slow_query_log = 'ON'"
   
   # Check slow queries
   docker compose exec firefly-iii-application.mariadb \
     mysql -u root -p${DB_ROOT_PASSWORD} -e "SELECT * FROM mysql.slow_log LIMIT 10"
   ```

## Performance Issues

### Slow Response Times

**Symptoms**:
- Pages load slowly
- API calls timeout
- High CPU usage

**Diagnostic Steps**:
```bash
# Check container resource usage
docker stats

# Check system resources
top
df -h

# Monitor HTTP response times
curl -w "@curl-format.txt" -o /dev/null -s http://localhost:8080/
```

**Solutions**:

1. **Enable Opcache**:
   ```bash
   # Verify opcache is enabled
   make run CMD="php -m | grep -i opcache"
   
   # Check opcache status
   make run CMD="php artisan tinker --execute='dd(opcache_get_status());'"
   ```

2. **Optimize Database**:
   ```bash
   # Run database optimization
   docker compose exec firefly-iii-application.mariadb \
     mysql -u root -p${DB_ROOT_PASSWORD} -e "OPTIMIZE TABLE ${DB_DATABASE}.*"
   
   # Update table statistics
   docker compose exec firefly-iii-application.mariadb \
     mysql -u root -p${DB_ROOT_PASSWORD} -e "ANALYZE TABLE ${DB_DATABASE}.*"
   ```

3. **Increase Container Resources**:
   ```yaml
   # In docker-compose.yml, add resource limits
   services:
     firefly-iii-application.application:
       deploy:
         resources:
           limits:
             memory: 2G
             cpus: '1.0'
   ```

### High Memory Usage

**Symptoms**:
- Containers killed by OOM
- System becomes unresponsive
- Swap usage high

**Diagnostic Steps**:
```bash
# Check memory usage by container
docker stats --no-stream

# Check system memory
free -h

# Check for memory leaks in logs
docker compose logs firefly-iii-application.application | grep -i memory
```

**Solutions**:

1. **Optimize PHP Memory**:
   ```bash
   # Check current memory limit
   make run CMD="php -r 'echo ini_get(\"memory_limit\").PHP_EOL;'"
   
   # Adjust memory limit if needed (in PHP configuration)
   ```

2. **Clear Caches**:
   ```bash
   # Clear all caches
   make run CMD="php artisan cache:clear"
   make run CMD="php artisan route:clear"
   make run CMD="php artisan config:clear"
   ```

## Data Issues

### Data Corruption

**Symptoms**:
- Database errors in logs
- Missing or incorrect data
- Application crashes

**Diagnostic Steps**:
```bash
# Check database integrity
docker compose exec firefly-iii-application.mariadb \
  mysql -u root -p${DB_ROOT_PASSWORD} -e "CHECK TABLE ${DB_DATABASE}.*"

# Check for foreign key issues
docker compose exec firefly-iii-application.mariadb \
  mysql -u root -p${DB_ROOT_PASSWORD} -e "SELECT * FROM information_schema.TABLE_CONSTRAINTS WHERE CONSTRAINT_TYPE = 'FOREIGN KEY'"
```

**Solutions**:

1. **Repair Tables**:
   ```bash
   # Repair corrupted tables
   docker compose exec firefly-iii-application.mariadb \
     mysql -u root -p${DB_ROOT_PASSWORD} -e "REPAIR TABLE ${DB_DATABASE}.table_name"
   ```

2. **Restore from Backup**:
   ```bash
   # Stop application
   docker compose stop firefly-iii-application.application
   
   # Restore database
   docker compose exec firefly-iii-application.mariadb \
     mysql -u root -p${DB_ROOT_PASSWORD} ${DB_DATABASE} < backup.sql
   
   # Start application
   docker compose start firefly-iii-application.application
   ```

### Migration Issues

**Symptoms**:
- Migration fails
- Database schema mismatch
- "Column not found" errors

**Diagnostic Steps**:
```bash
# Check migration status
make run CMD="php artisan migrate:status"

# Check database schema
docker compose exec firefly-iii-application.mariadb \
  mysql -u root -p${DB_ROOT_PASSWORD} -e "DESCRIBE ${DB_DATABASE}.migrations"
```

**Solutions**:

1. **Run Migrations**:
   ```bash
   # Run pending migrations
   make run CMD="php artisan migrate"
   
   # Force migrations if needed (be careful!)
   make run CMD="php artisan migrate --force"
   ```

2. **Reset Database** (development only):
   ```bash
   # Fresh migration (destroys data!)
   make run CMD="php artisan migrate:fresh"
   ```

## Network and Connectivity Issues

### External Access Issues

**Symptoms**:
- Cannot access application from outside
- DNS resolution fails
- SSL certificate errors

**Diagnostic Steps**:
```bash
# Check port binding
netstat -tulpn | grep :8080

# Test external access
curl -I http://your-domain.com:8080

# Check firewall rules
sudo ufw status
```

**Solutions**:

1. **Firewall Configuration**:
   ```bash
   # Allow HTTP traffic
   sudo ufw allow 8080/tcp
   
   # For HTTPS
   sudo ufw allow 443/tcp
   ```

2. **SSL/TLS Setup**:
   ```bash
   # Configure HTTPS in production
   # Update REQUIRE_HTTPS=true in .env
   # Configure SSL certificates in Nginx
   ```

## Diagnostic Tools

### Container Diagnostics

```bash
# Check container health
docker compose ps

# Inspect container details
docker inspect firefly-iii-application.application

# Check container logs
docker compose logs --tail=100 firefly-iii-application.application

# Execute commands in container
make enter
make run CMD="command"
```

### System Diagnostics

```bash
# Check system resources
htop
df -h
free -h

# Check Docker resource usage
docker system df
docker stats

# Check network connectivity
ping google.com
nslookup your-domain.com
```

### Application Diagnostics

```bash
# Check Laravel status
make run CMD="php artisan about"

# Check database connection
make run CMD="php artisan tinker --execute='DB::connection()->getPdo();'"

# Check cache status
make run CMD="php artisan cache:table"

# Check queue status
make run CMD="php artisan queue:monitor"
```

## Getting Help

### Log Collection

Before seeking help, collect relevant logs:

```bash
# Create diagnostic bundle
mkdir -p debug-logs/$(date +%Y%m%d)
cd debug-logs/$(date +%Y%m%d)

# Collect container logs
docker compose logs > docker-compose.log

# Collect system info
docker version > docker-info.txt
docker compose version >> docker-info.txt
uname -a >> system-info.txt
df -h >> system-info.txt
free -h >> system-info.txt

# Collect configuration (remove sensitive data!)
cp ../../.env env-config.txt
sed -i 's/PASSWORD=.*/PASSWORD=***REDACTED***/g' env-config.txt
```

### Support Channels

1. **Official Firefly III Documentation**: https://docs.firefly-iii.org/
2. **Firefly III GitHub Issues**: https://github.com/firefly-iii/firefly-iii/issues
3. **Community Forums**: https://www.reddit.com/r/FireflyIII/
4. **Discord Server**: https://discord.gg/firefly-iii

### Issue Reporting Template

When reporting issues, include:
- **Environment**: OS, Docker version, container versions
- **Configuration**: Relevant environment variables (sanitized)
- **Steps to Reproduce**: Exact steps that cause the issue
- **Expected Behavior**: What should happen
- **Actual Behavior**: What actually happens
- **Logs**: Relevant log excerpts
- **Additional Context**: Any other relevant information

---

**Sources**:
- Firefly III Troubleshooting Guide - https://docs.firefly-iii.org/how-to/firefly-iii/troubleshooting/ (Retrieved: 2024-01-15)
- Laravel Troubleshooting Documentation - https://laravel.com/docs/11.x/errors (Retrieved: 2024-01-15)
- Docker Troubleshooting Guide - https://docs.docker.com/config/troubleshooting/ (Retrieved: 2024-01-15)