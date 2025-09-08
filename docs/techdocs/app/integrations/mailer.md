---
title: "Mailer Integration"
description: "Email configuration, supported drivers, and notification setup."
tags: [mail, email, integration, smtp]
search: { boost: 3, exclude: false }
icon: material/email
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Document email integration for notifications and user communications.

**Contents**
- [Supported mail drivers](#supported-mail-drivers)
- [SMTP configuration](#smtp-configuration)
- [Email notifications](#email-notifications)
- [Testing and troubleshooting](#testing-and-troubleshooting)
- [Sources](#sources)

## Supported mail drivers

Firefly III supports multiple email backends:

| Driver | Use Case | Configuration Complexity | Source |
|--------|----------|--------------------------|--------|
| `smtp` | **Production recommended** | Medium | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `log` | **Development/testing** | Low | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `sendmail` | Local server setup | Low | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `mailgun` | Mailgun service | Medium | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `postmark` | Postmark service | Medium | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |

## SMTP configuration

**Environment variables:**

| Variable | Purpose | Example | Required |
|----------|---------|---------|----------|
| `MAIL_MAILER` | Mail driver selection | smtp | Yes |
| `MAIL_HOST` | SMTP server hostname | smtp.gmail.com | Yes (SMTP) |
| `MAIL_PORT` | SMTP server port | 587, 465, 25 | Yes (SMTP) |
| `MAIL_USERNAME` | SMTP authentication username | user@example.com | Yes (SMTP) |
| `MAIL_PASSWORD` | SMTP authentication password | (from secrets) | Yes (SMTP) |
| `MAIL_ENCRYPTION` | Encryption method | tls, ssl, null | Recommended |
| `MAIL_FROM_ADDRESS` | Sender email address | noreply@example.com | Yes |
| `MAIL_FROM_NAME` | Sender display name | Firefly III | Yes |

**Common SMTP configurations:**

| Provider | Host | Port | Encryption | Notes |
|----------|------|------|------------|-------|
| Gmail | smtp.gmail.com | 587 | tls | Requires app password |
| Outlook | smtp-mail.outlook.com | 587 | tls | Modern authentication |
| SendGrid | smtp.sendgrid.net | 587 | tls | API key as password |
| Mailgun | smtp.mailgun.org | 587 | tls | SMTP credentials |
| AWS SES | email-smtp.region.amazonaws.com | 587 | tls | IAM credentials |

**Docker Compose configuration:**
```yaml
firefly-iii-application.application:
  environment:
    MAIL_MAILER: smtp
    MAIL_HOST: smtp.example.com
    MAIL_PORT: 587
    MAIL_USERNAME: firefly@example.com
    MAIL_ENCRYPTION: tls
    MAIL_FROM_ADDRESS: firefly@example.com
    MAIL_FROM_NAME: "Firefly III"
    # MAIL_PASSWORD loaded from secrets
```

## Email notifications

**Firefly III sends emails for:**

| Event | Description | Frequency | Can Disable |
|-------|-------------|-----------|-------------|
| Registration | New user account creation | Once | No |
| Password Reset | Password reset requests | On demand | No |
| Budget Alerts | Budget overspend notifications | Configurable | Yes |
| Bill Reminders | Upcoming bill notifications | Configurable | Yes |
| Report Generation | Automated report delivery | Scheduled | Yes |
| Data Import | Import completion notifications | Per import | Yes |

**Notification preferences:**
- Users can configure notification preferences in their profile
- Admins can set system-wide default notification settings
- Email frequency can be adjusted per notification type

## Testing and troubleshooting

**Test email functionality:**
```bash
# Send test email from container
docker exec firefly-iii-application.application php artisan tinker
>>> Mail::raw('Test email', function($msg) { $msg->to('test@example.com')->subject('Test'); });
```

**Debugging email issues:**

1. **Check logs:**
```bash
# View application logs
docker logs firefly-iii-application.application | grep -i mail

# View Laravel logs
docker exec firefly-iii-application.application tail -f storage/logs/laravel.log
```

2. **Verify SMTP connectivity:**
```bash
# Test SMTP connection
docker exec firefly-iii-application.application php -r "
  $smtp = fsockopen('smtp.example.com', 587, $errno, $errstr, 30);
  echo $smtp ? 'SMTP reachable' : 'SMTP unreachable: ' . $errstr;
  if ($smtp) fclose($smtp);
"
```

3. **Common issues:**
   - **Authentication failures**: Check username/password credentials
   - **TLS errors**: Verify encryption settings and certificate validity
   - **Port blocking**: Ensure firewall allows outbound SMTP traffic
   - **Rate limiting**: Check provider rate limits and daily quotas

**Log driver (development):**
When using `MAIL_MAILER=log`, emails are written to Laravel logs instead of being sent:
```bash
# View logged emails
docker exec firefly-iii-application.application tail -f storage/logs/laravel.log | grep -A 20 "mail.local"
```

## Security considerations

**Best practices:**
- Use application-specific passwords (Gmail, Outlook)
- Enable two-factor authentication on email provider
- Use TLS encryption for SMTP connections
- Rotate SMTP credentials regularly
- Monitor for bounce/rejection rates

**Avoid common pitfalls:**
- Don't use personal email accounts for system notifications
- Don't hardcode email passwords in configuration files
- Don't use unencrypted SMTP connections (port 25)
- Don't ignore SPF/DKIM/DMARC setup for deliverability

## Sources
- "Firefly III Configuration Reference" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09
- "Laravel Mail Configuration" — https://laravel.com/docs/mail — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/":"","https://laravel.com/docs/mail":""},"sections":{"mailer-integration":""}}}
-->