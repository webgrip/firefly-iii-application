---
title: "Entrypoint & CLI"
description: "Official entrypoint behavior and supported CLI flags used here."
tags: [entrypoint, cli]
search: { boost: 2, exclude: false }
icon: material/console
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Capture startup behavior and commands we rely on.

**Contents**
- [Entrypoint behavior](#entrypoint-behavior)
- [CLI commands & flags used](#cli-commands--flags-used)
- [Sources](#sources)

## Entrypoint behavior

The Firefly III container uses PHP-FPM as the primary entrypoint:

**Container startup sequence:**
1. Initialize environment variables from `.env` file
2. Wait for database connectivity (if configured)
3. Run database migrations automatically
4. Clear and optimize Laravel caches
5. Start PHP-FPM process on port 9000
6. Health check responds on `/health` endpoint

**Environment initialization:**
- Reads configuration from `/var/www/app/.env`
- Validates required environment variables
- Sets up Laravel framework configuration
- Connects to database and verifies schema

## CLI commands & flags used

**Artisan commands (Laravel CLI):**

| Command | Purpose | Usage in deployment | Source |
|---------|---------|---------------------|--------|
| `php artisan migrate` | Run database migrations | Automatic on startup | "Firefly III Installation" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `php artisan cache:clear` | Clear application cache | Automatic on startup | "Firefly III Installation" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `php artisan config:cache` | Cache configuration | Automatic on startup | "Firefly III Installation" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `php artisan route:cache` | Cache routes | Automatic on startup | "Firefly III Installation" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| `php artisan view:cache` | Cache Blade templates | Automatic on startup | "Firefly III Installation" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |

**Firefly III specific commands:**

| Command | Purpose | Usage | Source |
|---------|---------|-------|--------|
| `php artisan firefly-iii:create-admin` | Create admin user | Manual user creation | "Firefly III CLI" — https://docs.firefly-iii.org/how-to/firefly-iii/command-line/ — retrieved 2025-01-09 |
| `php artisan firefly-iii:verify` | Verify installation | Health check validation | "Firefly III CLI" — https://docs.firefly-iii.org/how-to/firefly-iii/command-line/ — retrieved 2025-01-09 |
| `php artisan firefly-iii:upgrade-database` | Upgrade database schema | Manual upgrades | "Firefly III CLI" — https://docs.firefly-iii.org/how-to/firefly-iii/command-line/ — retrieved 2025-01-09 |

**PHP-FPM process:**

| Flag | Purpose | Default | Notes |
|------|---------|---------|-------|
| `-F` | Run in foreground | Always used | Required for container operation |
| `-R` | Allow to run as root | Not used | Security: run as www-data |

**Container health check:**
- **Command**: `curl -f http://localhost:9000/health || exit 1`
- **Interval**: 30 seconds
- **Timeout**: 3 seconds
- **Retries**: 3
- **Start period**: 60 seconds

## Sources
- "Firefly III Installation Guide" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09
- "Firefly III Command Line" — https://docs.firefly-iii.org/how-to/firefly-iii/command-line/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/":"","https://docs.firefly-iii.org/how-to/firefly-iii/command-line/":""},"sections":{"entrypoint":""}}}
-->