---
title: "Mailer Integration"
description: "Email configuration for notifications and system communication."
tags: [mailer, email, smtp, firefly-iii]
search: { boost: 3, exclude: false }
icon: material/email
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Document email integration for Firefly III notifications.

**Contents**
- [Mail Configuration](#mail-configuration)
- [Supported Drivers](#supported-drivers)
- [Container Interface](#container-interface)
- [Sources](#sources)

### Mail Configuration

Firefly III uses email for:
- **User Registration**: Account verification emails
- **Password Reset**: Secure password reset tokens
- **Budget Notifications**: Alerts when budgets are exceeded
- **Bill Reminders**: Upcoming bill payment notifications
- **Import Results**: Data import completion and error reports
- **Security Alerts**: Login notifications and security events

**Default Configuration (Development):**
```bash
MAIL_MAILER=log                    # Log emails to file instead of sending
MAIL_HOST=                         # Not used with log driver
MAIL_PORT=                         # Not used with log driver
MAIL_USERNAME=                     # Not used with log driver
MAIL_PASSWORD=                     # Not used with log driver
MAIL_ENCRYPTION=                   # Not used with log driver
MAIL_FROM_ADDRESS=admin@example.com
MAIL_FROM_NAME="Firefly III"
```

### Supported Drivers

| Driver | Use Case | Configuration Requirements | Notes |
|--------|----------|---------------------------|-------|
| `log` | Development/Testing | None | Emails written to log files |
| `smtp` | Production SMTP | Host, port, username, password | Most common production setup |
| `sendmail` | Local sendmail | Path to sendmail binary | Linux server with sendmail installed |
| `mailgun` | Mailgun API | API key, domain | Third-party email service |
| `postmark` | Postmark API | API token | Third-party email service |
| `ses` | Amazon SES | AWS credentials, region | Cloud email service |

**SMTP Configuration Example:**
```bash
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-app-password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=firefly@yourdomain.com
MAIL_FROM_NAME="Your Firefly III"
```

### Container Interface

| Aspect | Value / Path | Notes |
|-------|---------------|-------|
| Mail Queue | Redis/Database | Background processing for bulk emails |
| Log Location | `/var/www/app/storage/logs/laravel.log` | When using log driver |
| Templates | `/var/www/app/resources/views/emails/` | Email template files |
| Localization | Supports multiple languages | Based on user preferences |

**Email Security:**
- **TLS/SSL**: Always use encrypted connections (TLS/STARTTLS)
- **Authentication**: Use application-specific passwords when possible
- **Rate Limiting**: Respect email provider rate limits
- **SPF/DKIM**: Configure DNS records for delivery reliability

**Testing Email Configuration:**
```bash
# Test email configuration
docker compose exec firefly-iii-application.application \
  php artisan tinker --execute="Mail::raw('Test email', function(\$message) { \$message->to('test@example.com')->subject('Test'); });"

# Check email logs (when using log driver)
docker compose exec firefly-iii-application.application \
  tail -f storage/logs/laravel.log | grep -i mail
```

**Queue Processing (for high-volume setups):**
```bash
# Process email queue
docker compose exec firefly-iii-application.application \
  php artisan queue:work --queue=mail --timeout=60
```

### Sources
- "Laravel Mail Configuration" — https://laravel.com/docs/10.x/mail — retrieved 2025-01-09
- "Firefly III Email Settings" — https://docs.firefly-iii.org/references/firefly-iii/environment-variables — retrieved 2025-01-09
- "SMTP Security Best Practices" — https://tools.ietf.org/html/rfc3207 — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://laravel.com/docs/10.x/mail":"sha256:pending","https://docs.firefly-iii.org/references/firefly-iii/environment-variables":"sha256:pending","https://tools.ietf.org/html/rfc3207":"sha256:pending"},"sections":{"mailer-integration":"sha256:pending"}}}
-->