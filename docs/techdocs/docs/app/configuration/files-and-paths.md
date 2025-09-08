---
title: "Files & Paths"
description: "Config files, data directories, and persistence responsibilities."
tags: [configuration, filesystem, storage]
search: { boost: 3, exclude: false }
icon: material/folder-cog
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Understand file system layout, persistent storage, and configuration file locations in the containerized Invoice Ninja deployment.

**Contents**
- [Container File System Layout](#container-file-system-layout)
- [Persistent Storage](#persistent-storage)
- [Configuration Files](#configuration-files)
- [Log Files](#log-files)
- [Temporary Files](#temporary-files)
- [Sources](#sources)

## Container File System Layout

### Application Directory Structure
The Invoice Ninja application is installed in `/var/www/app` within the container:

```
/var/www/app/
├── app/                    # Laravel application logic
├── bootstrap/              # Framework bootstrap files
├── config/                 # Laravel configuration files
├── database/              # Database migrations and seeders
├── public/                # Web-accessible files (PERSISTENT)
│   ├── index.php          # Application entry point
│   ├── css/               # Compiled CSS assets
│   ├── js/                # Compiled JavaScript assets
│   └── uploads/           # User uploaded files
├── resources/             # Raw assets and views
├── routes/                # Application routes
├── storage/               # Application storage (PERSISTENT)
│   ├── app/               # Application file storage
│   ├── framework/         # Framework cache and sessions
│   └── logs/              # Application log files
├── vendor/                # Composer dependencies
├── .env                   # Environment configuration (MOUNTED)
├── artisan                # Laravel command-line interface
└── composer.json          # PHP dependency definitions
```

### System Directories
| Path | Purpose | Persistent | Notes |
|------|---------|------------|-------|
| `/var/www/app/public` | Web-accessible assets and uploads | Yes | Shared with Nginx container |
| `/var/www/app/storage` | App data, cache, sessions, logs | Yes | Performance-critical for database |
| `/var/www/app/.env` | Environment configuration | Mounted | Host file mounted as volume |
| `/tmp` | Temporary files and PHP uploads | No | Cleared on container restart |
| `/var/log` | System and web server logs | No | Use centralized logging |

## Persistent Storage

### Volume Mounts (Docker Compose)
The application requires persistent storage for data that must survive container restarts:

```yaml
volumes:
  # Application public files (shared with nginx)
  - firefly-iii-application-application-public-data:/var/www/app/public
  
  # Application storage (logs, cache, sessions, uploads)
  - firefly-iii-application-application-storage-data:/var/www/app/storage
  
  # Environment configuration (mounted from host)
  - ./.env:/var/www/app/.env
```

### Storage Requirements
| Volume | Size Estimate | Growth Rate | Backup Priority |
|--------|---------------|-------------|-----------------|
| Public files | 50-100 MB | Low (assets only) | Medium |
| Storage data | 1-10 GB | High (documents, logs) | Critical |
| Database | 100 MB - 50 GB | High (transaction data) | Critical |
| Redis cache | 50-500 MB | None (volatile) | None |

### File Permissions
All application files must be owned by the web server user:

```bash
# Container file ownership
USER_ID=1000
GROUP_ID=1000

# Application files
chown -R $USER_ID:$GROUP_ID /var/www/app/
chmod -R 755 /var/www/app/
chmod -R 775 /var/www/app/storage/
chmod -R 775 /var/www/app/public/
```

## Configuration Files

### Environment Configuration
| File | Location | Purpose | Mount Type |
|------|----------|---------|------------|
| `.env` | `/var/www/app/.env` | Application environment variables | Host file mount |
| `.env.example` | `/var/www/app/.env.example` | Template for environment setup | Baked into image |

**Environment File Format:**
```bash
# Core application settings
APP_ENV=production
APP_DEBUG=false
APP_URL=https://invoices.company.com

# Database connection
DB_CONNECTION=mysql
DB_HOST=production-mariadb
DB_DATABASE=invoiceninja
DB_USERNAME=ninja_user
DB_PASSWORD=secure_password

# Cache and sessions
CACHE_DRIVER=redis
SESSION_DRIVER=redis
REDIS_HOST=production-redis
```

### Laravel Configuration
Laravel configuration files are located in `/var/www/app/config/`:

| File | Purpose | Customization |
|------|---------|---------------|
| `app.php` | Core application settings | Environment-based via .env |
| `database.php` | Database connections | Configured via DB_* env vars |
| `cache.php` | Cache configuration | Configured via CACHE_* env vars |
| `mail.php` | Email configuration | Configured via MAIL_* env vars |
| `session.php` | Session management | Configured via SESSION_* env vars |
| `filesystems.php` | File storage configuration | Configured via FILESYSTEM_* env vars |

### Web Server Configuration
Nginx configuration is handled in the separate nginx container:

| File | Location | Purpose |
|------|----------|---------|
| `nginx.conf` | `/etc/nginx/nginx.conf` | Main Nginx configuration |
| `default.conf` | `/etc/nginx/conf.d/default.conf` | Virtual host configuration |

## Log Files

### Application Logs
Laravel application logs are stored in `/var/www/app/storage/logs/`:

| File | Purpose | Rotation | Level |
|------|---------|----------|-------|
| `laravel.log` | Main application log | Daily | INFO, WARNING, ERROR |
| `laravel-YYYY-MM-DD.log` | Daily log files | Automatic | Configurable via LOG_LEVEL |

### Log Configuration
```bash
# Logging environment variables
LOG_CHANNEL=stack           # Log to multiple channels
EXPANDED_LOGGING=true       # Enable detailed logging
LOG_LEVEL=info              # Minimum log level
LOG_PDF_HTML=true           # Log PDF generation details
```

### Log Format
Application logs use structured JSON format for parsing:

```json
{
  "timestamp": "2025-01-09T10:30:00Z",
  "level": "INFO",
  "message": "Invoice created successfully",
  "context": {
    "invoice_id": "INV-2025-001",
    "client_id": 123,
    "amount": 1500.00
  },
  "extra": {
    "user_id": 456,
    "ip_address": "192.168.1.100"
  }
}
```

### External Log Aggregation
For production deployments, logs should be forwarded to centralized systems:

- **Stdout/Stderr**: Docker log driver forwards to external systems
- **Log Files**: Mount storage volume and use log shippers (Filebeat, Fluentd)
- **Structured Logging**: JSON format enables easy parsing and filtering

## Temporary Files

### PHP Temporary Files
PHP temporary files for uploads and processing:

| Path | Purpose | Cleanup |
|------|---------|---------|
| `/tmp/php_uploads/` | File upload staging | Automatic after request |
| `/tmp/php_sessions/` | PHP session files (if not using Redis) | Based on session timeout |
| `/var/tmp/` | Long-term temporary files | Manual cleanup required |

### Application Cache
Framework and application cache files:

| Path | Purpose | Cleanup |
|------|---------|---------|
| `/var/www/app/storage/framework/cache/` | Application data cache | TTL-based expiration |
| `/var/www/app/storage/framework/views/` | Compiled Blade templates | Cleared on deployment |
| `/var/www/app/storage/framework/sessions/` | File-based sessions (if not Redis) | TTL-based expiration |

### PDF Generation
Invoice Ninja generates PDF files for invoices and reports:

| Path | Purpose | Cleanup |
|------|---------|---------|
| `/tmp/pdf_generation/` | PDF rendering workspace | Immediate after generation |
| `/var/www/app/storage/app/pdf/` | Temporary PDF storage | Configurable TTL |

## Container Interface

| Aspect | Value / Path | Notes |
|-------|---------------|-------|
| Working Directory | `/var/www/app` | Application root directory |
| Document Root | `/var/www/app/public` | Nginx document root |
| Log Output | `/var/www/app/storage/logs/` | Also STDOUT for container logs |
| Upload Directory | `/var/www/app/public/uploads/` | User file uploads |
| Cache Directory | `/var/www/app/storage/framework/cache/` | Application cache |
| Session Storage | Redis or `/var/www/app/storage/framework/sessions/` | Configurable backend |

## Backup Considerations

### Critical Data
Files that must be included in backups:

1. **Database Content**: All invoice, client, and financial data
2. **Uploaded Files**: Client documents, logos, attachments
3. **Configuration**: Environment files and custom configurations
4. **SSL Certificates**: If stored locally rather than external provider

### Non-Critical Data
Files that can be regenerated and don't need backup:

1. **Application Cache**: Can be rebuilt from source data
2. **Compiled Assets**: Generated during deployment
3. **Log Files**: Archived via log management system
4. **PHP Sessions**: Short-lived user session data

### Backup Strategy
```bash
# Database backup
mysqldump -h $DB_HOST -u $DB_USERNAME -p$DB_PASSWORD $DB_DATABASE > backup.sql

# File system backup
tar -czf invoice_ninja_files.tar.gz \
  /var/lib/docker/volumes/firefly-iii-application-application-public-data/ \
  /var/lib/docker/volumes/firefly-iii-application-application-storage-data/

# Environment configuration
cp .env env_backup_$(date +%Y%m%d).env
```

## Sources

- "Laravel Directory Structure" — https://laravel.com/docs/10.x/structure — retrieved 2025-01-09
- "Invoice Ninja File Storage" — https://invoiceninja.github.io/en/self-host-installation/#file-storage — retrieved 2025-01-09
- "Docker Volume Management" — https://docs.docker.com/storage/volumes/ — retrieved 2025-01-09
- "PHP Configuration" — https://www.php.net/manual/en/configuration.file.php — retrieved 2025-01-09
- "Nginx Configuration Guide" — https://nginx.org/en/docs/beginners_guide.html — retrieved 2025-01-09