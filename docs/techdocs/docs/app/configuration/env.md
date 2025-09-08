---
title: "Environment Variables"
description: "Required env vars with purpose, defaults, and constraints."
tags: [configuration, environment, firefly-iii]
search: { boost: 4, exclude: false }
icon: material/form-textbox
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Enumerate required env vars and constraints for Firefly III.

**Contents**
- [Required variables](#required-variables)
- [Optional variables used here](#optional-variables-used-here)
- [Sources](#sources)

### Required variables

| Name | Purpose | Default | Constraints / Format | Source |
|------|---------|---------|----------------------|--------|
| APP_KEY | Laravel application encryption key | - | Base64 encoded 32-byte key | "Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |
| APP_URL | Base URL for the application | - | Valid HTTP/HTTPS URL | "Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |
| DB_CONNECTION | Database driver | mysql | mysql, pgsql, sqlite | "Database Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |
| DB_HOST | Database hostname | - | Hostname or IP address | "Database Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |
| DB_PORT | Database port | 3306 | Valid port number | "Database Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |
| DB_DATABASE | Database name | - | Valid database name | "Database Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |
| DB_USERNAME | Database username | - | Valid database user | "Database Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |
| DB_PASSWORD | Database password | - | Secure password | "Database Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |

### Optional variables used here

| Name | Purpose | Default | Notes | Source |
|------|---------|---------|-------|--------|
| APP_ENV | Application environment | production | local, production | "Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |
| APP_DEBUG | Enable debug mode | false | true/false, never enable in production | "Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |
| APP_LOCALE | Application locale | en | ISO 639-1 language codes | "Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |
| APP_TIMEZONE | Application timezone | UTC | Valid timezone identifier | "Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |
| CACHE_DRIVER | Cache driver | file | file, database, redis | "Caching" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |
| SESSION_DRIVER | Session driver | file | file, database, redis | "Sessions" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |
| REDIS_HOST | Redis hostname | 127.0.0.1 | Hostname or IP when using Redis | "Redis Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |
| REDIS_PORT | Redis port | 6379 | Valid port number | "Redis Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |
| MAIL_MAILER | Mail driver | log | log, smtp, sendmail | "Mail Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |
| TRUSTED_PROXIES | Trusted proxy IPs | - | Comma-separated IPs or * for all | "Proxy Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09 |

### Sources
- "Firefly III Docker Configuration" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09
- "Firefly III Environment Variables" — https://docs.firefly-iii.org/references/firefly-iii/environment-variables — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker":"sha256:pending","https://docs.firefly-iii.org/references/firefly-iii/environment-variables":"sha256:pending"},"sections":{"env-table":"sha256:pending"}}}
-->