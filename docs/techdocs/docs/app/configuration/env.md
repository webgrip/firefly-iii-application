---
title: "Environment Variables"
description: "Required env vars with purpose, defaults, and constraints."
tags: [configuration, environment, variables]
search: { boost: 4, exclude: false }
icon: material/form-textbox
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Configure Invoice Ninja through environment variables used in this containerized deployment.

**Contents**
- [Core Application Settings](#core-application-settings)
- [Database Configuration](#database-configuration)
- [Cache and Session Management](#cache-and-session-management)
- [Mail Configuration](#mail-configuration)
- [Security Settings](#security-settings)
- [Invoice Ninja Specific](#invoice-ninja-specific)
- [Sources](#sources)

## Core Application Settings

### Basic Application Configuration
| Name | Purpose | Default | Constraints / Format | Source |
|------|---------|---------|----------------------|--------|
| `BASE_URL` | Public URL for accessing the application | `http://localhost:8080` | Valid URL format | Laravel config |
| `APP_ENV` | Application environment mode | `local` | `local`, `production`, `staging` | Laravel framework |
| `APP_URL` | Application URL used for link generation | `http://localhost:8080` | Must match BASE_URL | Laravel routing |
| `APP_DEBUG` | Enable debug mode and detailed error output | `true` | `true`, `false` | Laravel debugging |
| `APP_LOCALE` | Default application language | `en` | ISO 639-1 language code | Laravel localization |
| `APP_TIMEZONE` | Application timezone for date/time handling | `Europe/Amsterdam` | Valid timezone identifier | PHP timezone |
| `APP_KEY` | Laravel application encryption key | `""` (auto-generated) | Base64 encoded 32-byte key | Laravel encryption |

**Security Notes:**
- `APP_KEY` is auto-generated on first run if empty
- `APP_DEBUG` must be `false` in production environments
- `APP_ENV=production` enables optimizations and security features

## Database Configuration

### MariaDB Connection Settings
| Name | Purpose | Default | Constraints / Format | Source |
|------|---------|---------|----------------------|--------|
| `DB_CONNECTION` | Database driver type | `mysql` | `mysql`, `pgsql`, `sqlite` | Laravel database |
| `DB_HOST` | Database server hostname | `firefly-iii-application.mariadb` | Container name or IP | Docker Compose |
| `DB_PORT` | Database server port | `3306` | Valid port number | MariaDB default |
| `DB_DATABASE` | Database name | `application` | Valid database identifier | Container config |
| `DB_USERNAME` | Database user for application | `application` | Valid MySQL username | Container config |
| `DB_PASSWORD` | Database password | `application` | Strong password required for production | Container config |
| `DB_ROOT_PASSWORD` | MariaDB root password | `root` | Strong password required for production | Container config |

**Connection Example:**
```bash
# Production-ready database configuration
DB_CONNECTION=mysql
DB_HOST=production-mariadb.example.com
DB_PORT=3306
DB_DATABASE=invoiceninja_prod
DB_USERNAME=ninja_app
DB_PASSWORD=secure_random_password_here
```

## Cache and Session Management

### Redis Configuration
| Name | Purpose | Default | Constraints / Format | Source |
|------|---------|---------|----------------------|--------|
| `REDIS_HOST` | Redis server hostname | `firefly-iii-application.redis` | Container name or IP | Docker Compose |
| `REDIS_PORT` | Redis server port | `6379` | Valid port number | Redis default |
| `REDIS_PASSWORD` | Redis authentication password | `""` (no auth) | Strong password for production | Redis security |
| `CACHE_DRIVER` | Cache storage backend | `redis` | `redis`, `file`, `database` | Laravel cache |
| `CACHE_HOST` | Cache server hostname | `firefly-iii-application.redis` | Must match REDIS_HOST | Cache config |
| `SESSION_DRIVER` | Session storage backend | `redis` | `redis`, `file`, `database` | Laravel sessions |
| `SESSION_DOMAIN` | Cookie domain for sessions | `localhost` | Valid domain name | Session security |
| `SESSION_ENCRYPT` | Encrypt session data | `false` | `true`, `false` | Session security |
| `SESSION_SECURE` | HTTPS-only session cookies | `false` | `true` for production HTTPS | Session security |
| `QUEUE_CONNECTION` | Queue backend for background jobs | `redis` | `redis`, `database`, `sync` | Laravel queues |

**Redis Performance Notes:**
- Redis handles sessions, cache, and queue data
- Multiple Redis instances can be used for different purposes
- `SESSION_SECURE=true` required when using HTTPS

## Mail Configuration

### SMTP and Email Settings
| Name | Purpose | Default | Constraints / Format | Source |
|------|---------|---------|----------------------|--------|
| `MAIL_MAILER` | Mail sending method | `log` | `smtp`, `sendmail`, `mailgun`, `postmark`, `log` | Laravel mail |
| `MAIL_HOST` | SMTP server hostname | `mailhog` | Valid hostname or IP | SMTP config |
| `MAIL_PORT` | SMTP server port | `1025` | Valid port number | SMTP config |
| `MAIL_USERNAME` | SMTP authentication username | `""` | Valid email or username | SMTP auth |
| `MAIL_PASSWORD` | SMTP authentication password | `""` | SMTP password | SMTP auth |
| `MAIL_ENCRYPTION` | SMTP encryption method | `""` | `tls`, `ssl`, `""` (none) | SMTP security |
| `MAIL_FROM_ADDRESS` | Default sender email address | `ryan@webgrip.nl` | Valid email address | Email headers |
| `MAIL_FROM_NAME` | Default sender name | `"Invoice Ninja"` | Display name | Email headers |

**Production SMTP Example:**
```bash
# Production SMTP configuration
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=invoices@company.com
MAIL_PASSWORD=app_specific_password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=invoices@company.com
MAIL_FROM_NAME="Company Invoices"
```

## Security Settings

### API and Authentication
| Name | Purpose | Default | Constraints / Format | Source |
|------|---------|---------|----------------------|--------|
| `API_SECRET` | API authentication secret | `supersecret` | Strong random string | Invoice Ninja API |
| `UPDATE_SECRET` | Software update authentication | `supersecret` | Strong random string | Invoice Ninja updates |
| `WEBCRON_SECRET` | Web-based cron job authentication | `secret` | Strong random string | Invoice Ninja cron |
| `ERROR_EMAIL` | Email for error notifications | `ryan@webgrip.nl` | Valid email address | Error handling |
| `TRUSTED_PROXIES` | Trusted proxy IP addresses | `*` | IP addresses or `*` | Laravel proxies |

**Security Best Practices:**
- Use strong, unique secrets for each environment
- `TRUSTED_PROXIES=*` only safe behind proper load balancer
- Rotate secrets regularly and store securely

## Invoice Ninja Specific

### Application Features
| Name | Purpose | Default | Constraints / Format | Source |
|------|---------|---------|----------------------|--------|
| `NINJA_ENVIRONMENT` | Invoice Ninja deployment type | `selfhost` | `selfhost`, `hosted` | Invoice Ninja config |
| `IS_DOCKER` | Enable Docker-specific optimizations | `true` | `true`, `false` | Container config |
| `LOCAL_DOWNLOAD` | Download files locally vs cloud | `false` | `true`, `false` | File handling |
| `FILESYSTEM_DRIVER` | File storage backend | `local` | `local`, `s3`, `spaces` | Laravel filesystem |
| `EXPANDED_LOGGING` | Enable detailed logging | `true` | `true`, `false` | Debugging |
| `LOG_PDF_HTML` | Log PDF generation HTML | `true` | `true`, `false` | PDF debugging |
| `PDF_GENERATOR` | PDF generation engine | `snappdf` | `snappdf`, `wkhtmltopdf` | PDF generation |

### Optional External Integrations
| Name | Purpose | Default | Constraints / Format | Source |
|------|---------|---------|----------------------|--------|
| `POSTMARK_SECRET` | Postmark email service API key | `""` | Postmark API key | Email service |
| `GOOGLE_CLIENT_ID` | Google OAuth client ID | `""` | Google OAuth credentials | Authentication |
| `GOOGLE_CLIENT_SECRET` | Google OAuth client secret | `""` | Google OAuth credentials | Authentication |
| `GOOGLE_OAUTH_REDIRECT` | Google OAuth redirect URL | `""` | Valid callback URL | Authentication |

**External Service Examples:**
```bash
# Postmark email service
MAIL_MAILER=postmark
POSTMARK_SECRET=your_postmark_api_key_here

# Google OAuth integration  
GOOGLE_CLIENT_ID=your_google_client_id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your_google_client_secret
GOOGLE_OAUTH_REDIRECT=https://yourdomain.com/auth/google
```

## Container Interface

| Aspect | Value / Path | Notes |
|-------|---------------|-------|
| Ports | 9000 (app), 8080 (nginx), 3306 (db), 6379 (redis) | Internal container networking |
| Volumes | `/var/www/app/public`, `/var/www/app/storage` | Persistent application data |
| Health Check | HTTP GET to `/health` or application root | Nginx and application status |
| Log Output | STDOUT/STDERR with JSON formatting | Centralized log collection |
| Restart Policy | `always` for all services | Automatic recovery |

## Sources

- "Invoice Ninja Environment Configuration" — https://invoiceninja.github.io/en/self-host-installation/#environment — retrieved 2025-01-09
- "Laravel Configuration Documentation" — https://laravel.com/docs/10.x/configuration — retrieved 2025-01-09
- "Laravel Database Configuration" — https://laravel.com/docs/10.x/database — retrieved 2025-01-09
- "Laravel Mail Configuration" — https://laravel.com/docs/10.x/mail — retrieved 2025-01-09
- "Redis Configuration Guide" — https://redis.io/docs/management/config/ — retrieved 2025-01-09