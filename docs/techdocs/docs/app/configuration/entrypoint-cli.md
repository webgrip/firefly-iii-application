---
title: "Entrypoint & CLI"
description: "Official entrypoint behavior and supported CLI flags used here."
tags: [entrypoint, cli, artisan, commands]
search: { boost: 2, exclude: false }
icon: material/console
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Understand container entrypoint behavior and Laravel Artisan commands available for Invoice Ninja management.

**Contents**
- [Container Entrypoint](#container-entrypoint)
- [Laravel Artisan Commands](#laravel-artisan-commands)
- [Invoice Ninja Specific Commands](#invoice-ninja-specific-commands)
- [Database Management](#database-management)
- [Queue and Job Management](#queue-and-job-management)
- [Administrative Commands](#administrative-commands)
- [Sources](#sources)

## Container Entrypoint

### Default Container Behavior
The Invoice Ninja application container uses PHP-FPM as the primary entrypoint:

```dockerfile
# Container entrypoint (simplified)
ENTRYPOINT ["php-fpm"]
CMD ["-F", "-R"]
```

**Process Details:**
- **Primary Process**: PHP-FPM with FastCGI Process Manager
- **Worker Processes**: Configurable number of PHP worker processes
- **Signal Handling**: Graceful shutdown on SIGTERM
- **Health Monitoring**: Built-in FPM status and ping endpoints

### PHP-FPM Configuration
Key PHP-FPM runtime options:

| Option | Value | Purpose |
|--------|-------|---------|
| `-F` | Foreground mode | Run in foreground for container compatibility |
| `-R` | Allow running as root | Required for container initialization |
| `--nodaemonize` | No daemon mode | Keep process in foreground |
| `--force-stderr` | Log to stderr | Container log aggregation |

### Environment Variable Processing
Container entrypoint processes these environment variables:

```bash
# PHP Configuration
PHP_MEMORY_LIMIT=512M
PHP_MAX_EXECUTION_TIME=300
PHP_UPLOAD_MAX_FILESIZE=64M

# FPM Configuration  
FPM_PM_MAX_CHILDREN=50
FPM_PM_START_SERVERS=5
FPM_PM_MIN_SPARE_SERVERS=5
FPM_PM_MAX_SPARE_SERVERS=35
```

## Laravel Artisan Commands

### Command Structure
Laravel Artisan provides the CLI interface for Invoice Ninja management:

```bash
# General command format
docker compose exec firefly-iii-application.application php artisan <command> [options] [arguments]

# Or using make wrapper
make run CMD="php artisan <command> [options] [arguments]"
```

### Core Laravel Commands
Essential Laravel framework commands used in this deployment:

| Command | Purpose | Usage Example |
|---------|---------|---------------|
| `migrate` | Run database migrations | `php artisan migrate` |
| `migrate:rollback` | Rollback database migrations | `php artisan migrate:rollback` |
| `migrate:status` | Show migration status | `php artisan migrate:status` |
| `config:cache` | Cache configuration files | `php artisan config:cache` |
| `config:clear` | Clear configuration cache | `php artisan config:clear` |
| `route:cache` | Cache route definitions | `php artisan route:cache` |
| `route:clear` | Clear route cache | `php artisan route:clear` |
| `view:cache` | Cache Blade templates | `php artisan view:cache` |
| `view:clear` | Clear view cache | `php artisan view:clear` |

### Application Key Management
Laravel encryption key management:

```bash
# Generate new application key (first time setup)
php artisan key:generate

# Show current application key
php artisan tinker --execute="echo config('app.key');"
```

**Important Notes:**
- Application key is automatically generated if `APP_KEY` is empty
- Changing the key invalidates all encrypted data
- Backup the key before any changes

## Invoice Ninja Specific Commands

### User and Account Management
Invoice Ninja provides specific Artisan commands for user management:

```bash
# Create admin user account
php artisan ninja:create-account --email="admin@company.com" --password="secure_password"

# Create additional user
php artisan ninja:create-user --email="user@company.com" --password="user_password" --first-name="John" --last-name="Doe"

# Reset user password
php artisan ninja:reset-password --email="user@company.com"

# List all users
php artisan ninja:list-users
```

**Make Wrapper Usage:**
```bash
# Using the make target for user creation
make user:create EMAIL=admin@company.com PASS=secure_password

# Custom user creation with make run
make run CMD="php artisan ninja:create-account --email=admin@company.com --password=mypassword"
```

### Company and Tenant Management
Multi-tenant company management commands:

```bash
# Create new company
php artisan ninja:create-company --name="Company Name" --email="contact@company.com"

# List all companies
php artisan ninja:list-companies

# Migrate company data
php artisan ninja:migrate-company --company-id=1
```

### Data Import and Migration
Import data from other invoicing systems:

```bash
# Import from QuickBooks
php artisan ninja:import --type=quickbooks --file=/path/to/export.csv

# Import from FreshBooks
php artisan ninja:import --type=freshbooks --file=/path/to/export.xml

# Import from generic CSV
php artisan ninja:import --type=csv --file=/path/to/data.csv --mapping=/path/to/mapping.json
```

## Database Management

### Migration Commands
Database schema management:

```bash
# Run all pending migrations
php artisan migrate

# Run migrations with verbose output
php artisan migrate --verbose

# Rollback last migration batch
php artisan migrate:rollback

# Rollback specific number of batches
php artisan migrate:rollback --step=3

# Reset entire database (destructive)
php artisan migrate:reset

# Fresh migration (drop all tables and re-migrate)
php artisan migrate:fresh

# Seed database with sample data
php artisan db:seed
```

### Database Maintenance
Database optimization and maintenance:

```bash
# Check database connection
php artisan tinker --execute="DB::connection()->getPdo();"

# Optimize database tables
php artisan ninja:optimize-db

# Check database integrity
php artisan ninja:check-db

# Backup database to file
php artisan ninja:backup --output=/var/www/app/storage/backups/
```

## Queue and Job Management

### Queue Worker Commands
Background job processing:

```bash
# Start queue worker (blocking process)
php artisan queue:work

# Start queue worker with specific configuration
php artisan queue:work redis --sleep=3 --tries=3 --timeout=60

# Process only one job and exit
php artisan queue:work --once

# List failed jobs
php artisan queue:failed

# Retry all failed jobs
php artisan queue:retry all

# Retry specific failed job
php artisan queue:retry 1

# Clear all failed jobs
php artisan queue:flush
```

### Email Queue Management
Email-specific queue commands:

```bash
# Send pending emails
php artisan ninja:send-emails

# Process email queue
php artisan queue:work --queue=emails

# Check email queue status
php artisan queue:stats
```

## Administrative Commands

### Cache Management
Application cache control:

```bash
# Clear all caches
php artisan cache:clear

# Clear specific cache store
php artisan cache:clear --store=redis

# Cache application configuration
php artisan config:cache

# Cache routes for performance
php artisan route:cache

# Optimize for production (combines multiple optimizations)
php artisan optimize

# Clear all optimizations
php artisan optimize:clear
```

### System Information
Diagnostic and system information commands:

```bash
# Show Laravel version and environment
php artisan --version
php artisan env

# List all available commands
php artisan list

# Show detailed command help
php artisan help migrate

# Check system requirements
php artisan ninja:check-system

# Display application status
php artisan ninja:status
```

### Maintenance Mode
Application maintenance mode control:

```bash
# Put application in maintenance mode
php artisan down

# Put application in maintenance mode with message
php artisan down --message="Scheduled maintenance in progress"

# Allow specific IPs during maintenance
php artisan down --allow=192.168.1.1 --allow=10.0.0.0/8

# Bring application back online
php artisan up
```

## Container Interface

| Aspect | Value / Path | Notes |
|-------|---------------|-------|
| Primary Command | `php-fpm -F -R` | FastCGI Process Manager |
| Artisan CLI | `php artisan` | Laravel command-line interface |
| Working Directory | `/var/www/app` | Application root |
| Configuration | `/var/www/app/.env` | Environment variables |
| Process Owner | `www-data` (UID 1000) | Non-root user |
| Signal Handling | SIGTERM for graceful shutdown | Container lifecycle |

## Command Examples

### Initial Setup
```bash
# Complete initial setup sequence
make start                          # Start all containers
make wait-ready URL=http://localhost:8080  # Wait for services
make run CMD="php artisan migrate"  # Run database migrations
make user:create EMAIL=admin@company.com PASS=secure123  # Create admin user
```

### Daily Operations
```bash
# Check application status
make run CMD="php artisan ninja:status"

# Process background jobs
make run CMD="php artisan queue:work --once"

# Clear caches after updates
make run CMD="php artisan optimize:clear"
make run CMD="php artisan optimize"
```

### Troubleshooting
```bash
# Check database connectivity
make run CMD="php artisan tinker --execute='DB::connection()->getPdo();'"

# View recent logs
make logs SERVICE=firefly-iii-application.application

# Check configuration
make run CMD="php artisan config:show"
```

## Sources

- "Laravel Artisan Console" — https://laravel.com/docs/10.x/artisan — retrieved 2025-01-09
- "Invoice Ninja CLI Commands" — https://invoiceninja.github.io/en/self-host-installation/#artisan-commands — retrieved 2025-01-09
- "PHP-FPM Configuration" — https://www.php.net/manual/en/install.fpm.configuration.php — retrieved 2025-01-09
- "Docker Container Entrypoints" — https://docs.docker.com/engine/reference/builder/#entrypoint — retrieved 2025-01-09
- "Laravel Queue System" — https://laravel.com/docs/10.x/queues — retrieved 2025-01-09