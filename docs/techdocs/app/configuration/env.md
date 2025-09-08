---
title: "Environment Variables"
description: "Required env vars with purpose, defaults, and constraints."
tags: [configuration, environment]
search: { boost: 4, exclude: false }
icon: material/form-textbox
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Enumerate required env vars and constraints.

**Contents**
- [Required variables](#required-variables)
- [Optional variables used here](#optional-variables-used-here)
- [Sources](#sources)

## Required variables

| Name | Purpose | Default | Constraints / Format | Source |
|------|---------|---------|----------------------|--------|
| APP_KEY | Laravel application encryption key | None | base64:44-char string | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| APP_URL | Base URL for the application | None | Valid HTTP/HTTPS URL | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| DB_CONNECTION | Database type | None | mysql, pgsql, sqlite | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| DB_DATABASE | Database name | None | String | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| DB_HOST | Database host | None | Hostname or IP | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| DB_PASSWORD | Database password | None | String | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| DB_PORT | Database port | 3306 (MySQL), 5432 (PostgreSQL) | Valid port number | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| DB_USERNAME | Database username | None | String | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |

## Optional variables used here

| Name | Purpose | Default | Notes | Source |
|------|---------|---------|-------|--------|
| APP_DEBUG | Enable debug mode | false | Set to true only in development | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| APP_ENV | Application environment | production | local, production, staging | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| APP_LOCALE | Application language | en | ISO 639-1 language codes | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| APP_TIMEZONE | Application timezone | UTC | PHP timezone identifiers | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| CACHE_DRIVER | Cache driver | file | file, redis, memcached | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| IS_DOCKER | Enable Docker-specific optimizations | false | Set to true in containerized environments | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| MAIL_DRIVER | Mail driver | log | smtp, sendmail, mailgun, log | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| MAIL_ENCRYPTION | Mail encryption | None | tls, ssl | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| MAIL_FROM_ADDRESS | Sender email address | None | Valid email address | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| MAIL_HOST | SMTP host | None | Hostname or IP | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| MAIL_PORT | SMTP port | 587 | Valid port number | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| REDIS_HOST | Redis host | None | Hostname or IP | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| REDIS_PASSWORD | Redis password | None | String | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| REDIS_PORT | Redis port | 6379 | Valid port number | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| SESSION_DRIVER | Session storage driver | file | file, redis, database | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| TRUSTED_PROXIES | Trusted proxy IPs | None | IP addresses or CIDR blocks | "Firefly III Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |

## Sources
- "Firefly III Configuration Reference" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/":""},"sections":{"env-table":""}}}
-->