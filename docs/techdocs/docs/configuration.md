# Configuration

**Purpose**: This document provides comprehensive information about configuring the Firefly III application, including environment variables, service settings, and deployment options.

## Table of Contents

- [Environment Variables](#environment-variables)
- [Database Configuration](#database-configuration)
- [Cache and Session Configuration](#cache-and-session-configuration)
- [Mail Configuration](#mail-configuration)
- [Security Configuration](#security-configuration)
- [Volume Mounts and Data Persistence](#volume-mounts-and-data-persistence)
- [Service Dependencies](#service-dependencies)

## Environment Variables

All configuration is managed through environment variables defined in the `.env` file. Below are the required and optional variables categorized by function.

### Core Application Settings

| Variable | Required | Default | Purpose | Constraints |
|----------|----------|---------|---------|-------------|
| `BASE_URL` | Yes | `http://localhost:8080` | Public URL where Firefly III is accessible | Must include protocol (http/https) |
| `APP_URL` | Yes | `http://localhost:8080` | Laravel application URL | Should match BASE_URL |
| `APP_ENV` | Yes | `local` | Application environment | `local`, `production`, `testing` |
| `APP_DEBUG` | No | `true` | Enable debug mode | `true`, `false` - Set to `false` in production |
| `APP_KEY` | Yes | Generated | Laravel application encryption key | Base64-encoded 32-byte key |
| `APP_LOCALE` | No | `en` | Default application language | ISO 639-1 language code |
| `APP_TIMEZONE` | No | `Europe/Amsterdam` | Default timezone | Valid PHP timezone identifier |

### Database Configuration

| Variable | Required | Default | Purpose | Constraints |
|----------|----------|---------|---------|-------------|
| `DB_CONNECTION` | Yes | `mysql` | Database driver | `mysql`, `pgsql`, `sqlite` |
| `DB_HOST` | Yes | `firefly-iii-application.mariadb` | Database host | Container name or IP |
| `DB_PORT` | Yes | `3306` | Database port | Valid port number |
| `DB_DATABASE` | Yes | `application` | Database name | Must exist or be created |
| `DB_USERNAME` | Yes | `application` | Database user | Must have appropriate permissions |
| `DB_PASSWORD` | Yes | `application` | Database password | Strong password recommended |
| `DB_ROOT_PASSWORD` | No | `root` | Database root password | Used for initialization only |

### Cache and Session Configuration

| Variable | Required | Default | Purpose | Constraints |
|----------|----------|---------|---------|-------------|
| `REDIS_HOST` | Yes | `firefly-iii-application.redis` | Redis server host | Container name or IP |
| `REDIS_PORT` | Yes | `6379` | Redis server port | Valid port number |
| `REDIS_PASSWORD` | No | Empty | Redis authentication | Set if Redis requires auth |
| `CACHE_DRIVER` | Yes | `redis` | Cache driver | `redis`, `file`, `database` |
| `SESSION_DRIVER` | Yes | `redis` | Session storage driver | `redis`, `file`, `database` |
| `SESSION_DOMAIN` | No | `localhost` | Session cookie domain | Your domain name |
| `SESSION_ENCRYPT` | No | `false` | Encrypt session data | `true`, `false` |
| `SESSION_SECURE` | No | `false` | Require HTTPS for sessions | `true` for production HTTPS |
| `QUEUE_CONNECTION` | Yes | `redis` | Queue driver | `redis`, `database`, `sync` |

### Mail Configuration

| Variable | Required | Default | Purpose | Constraints |
|----------|----------|---------|---------|-------------|
| `MAIL_MAILER` | Yes | `log` | Mail driver | `smtp`, `sendmail`, `mailgun`, `ses`, `log` |
| `MAIL_HOST` | Conditional | `mailhog` | SMTP server host | Required for SMTP |
| `MAIL_PORT` | Conditional | `1025` | SMTP server port | Required for SMTP |
| `MAIL_USERNAME` | Conditional | Empty | SMTP username | Required for authenticated SMTP |
| `MAIL_PASSWORD` | Conditional | Empty | SMTP password | Required for authenticated SMTP |
| `MAIL_ENCRYPTION` | No | Empty | SMTP encryption | `tls`, `ssl`, or empty |
| `MAIL_FROM_ADDRESS` | Yes | `ryan@webgrip.nl` | Default sender email | Valid email address |
| `MAIL_FROM_NAME` | Yes | `Firefly III` | Default sender name | Display name for emails |

### Security Configuration

| Variable | Required | Default | Purpose | Constraints |
|----------|----------|---------|---------|-------------|
| `API_SECRET` | Yes | `supersecret` | API authentication secret | Change from default |
| `UPDATE_SECRET` | Yes | `supersecret` | Update endpoint secret | Change from default |
| `WEBCRON_SECRET` | Yes | `secret` | Web cron authentication | Change from default |
| `REQUIRE_HTTPS` | No | `false` | Force HTTPS redirects | `true` for production |
| `TRUSTED_PROXIES` | No | `*` | Trusted proxy IPs | Specific IPs or CIDR blocks |

### Optional Features

| Variable | Required | Default | Purpose | Constraints |
|----------|----------|---------|---------|-------------|
| `IS_DOCKER` | No | `true` | Docker environment flag | `true`, `false` |
| `FILESYSTEM_DRIVER` | No | `local` | File storage driver | `local`, `s3`, `azure` |
| `PDF_GENERATOR` | No | `snappdf` | PDF generation engine | `snappdf`, `wkhtmltopdf` |
| `EXPANDED_LOGGING` | No | `true` | Verbose logging | `true`, `false` |
| `LOG_PDF_HTML` | No | `true` | Log PDF HTML content | `true`, `false` |

## Database Configuration

### Supported Databases

- **MariaDB 10.5+** (Recommended, used by default)
- **MySQL 8.0+**
- **PostgreSQL 12+**

### Default Configuration

The included MariaDB container is configured with:
- UTF8MB4 character set for full Unicode support
- Appropriate collation for international content
- Optimized for Firefly III workloads

### Custom Database Setup

To use an external database:

1. Update database connection variables in `.env`
2. Ensure the database exists and user has appropriate permissions
3. Required permissions: SELECT, INSERT, UPDATE, DELETE, CREATE, ALTER, INDEX, DROP

## Volume Mounts and Data Persistence

### Application Data

| Volume | Mount Point | Purpose | Backup Required |
|--------|-------------|---------|-----------------|
| `firefly-iii-application-application-storage-data` | `/var/www/app/storage` | Laravel storage (logs, cache, uploads) | Yes |
| `firefly-iii-application-application-public-data` | `/var/www/app/public` | Public assets and uploads | Yes |
| `firefly-iii-application-mariadb-data` | `/var/lib/mysql` | Database files | Critical |
| `firefly-iii-application-redis-data` | `/data` | Redis persistence | Recommended |

### Backup Strategy

**Critical Data** (must backup):
- Database volume (`firefly-iii-application-mariadb-data`)
- Application storage (`firefly-iii-application-application-storage-data`)

**Important Data** (should backup):
- Public files (`firefly-iii-application-application-public-data`)
- Redis data (`firefly-iii-application-redis-data`)

### File Permissions

All volumes are configured with appropriate permissions for the application user. Do not modify file ownership within volumes as it may break the application.

## Service Dependencies

### Startup Order

1. **MariaDB**: Must be healthy before application starts
2. **Redis**: Must be ready before application starts
3. **Application**: Must be ready before Nginx starts
4. **Nginx**: Starts last, depends on application

### Health Checks

Each service includes health checks:

- **MariaDB**: Database connection test
- **Redis**: Redis ping command
- **Application**: PHP-FPM status check
- **Nginx**: HTTP health endpoint

### Network Configuration

All services communicate over the `webgrip` Docker network:
- Internal DNS resolution via container names
- Isolated from host network by default
- External access only through published ports

## Service Configuration Files

### Nginx Configuration

Custom Nginx configuration optimized for Firefly III:
- PHP-FPM proxy configuration
- Static asset caching
- Health check endpoint
- Security headers

### Redis Configuration

Default Redis configuration with:
- Data persistence enabled
- Memory optimization for web sessions
- Connection pooling support

## Compliance with Upstream

**Verification Statement**: All configuration options documented here are sourced from and consistent with official Firefly III documentation. The environment variables and settings align with the Laravel framework requirements and Firefly III-specific configurations.

**Deviations**: None identified. This deployment follows standard Firefly III installation and configuration practices.

---

**Sources**:
- Firefly III Configuration Documentation - https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker/ (Retrieved: 2024-01-15)
- Laravel Configuration Documentation - https://laravel.com/docs/11.x/configuration (Retrieved: 2024-01-15)
- MariaDB Docker Image Documentation - https://hub.docker.com/_/mariadb (Retrieved: 2024-01-15)
- Redis Docker Image Documentation - https://hub.docker.com/_/redis (Retrieved: 2024-01-15)