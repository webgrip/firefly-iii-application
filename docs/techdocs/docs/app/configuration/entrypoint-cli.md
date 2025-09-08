---
title: "Entrypoint & CLI"
description: "Official entrypoint behavior and supported CLI flags used here."
tags: [entrypoint, cli, firefly-iii, artisan]
search: { boost: 2, exclude: false }
icon: material/console
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Capture startup behavior and commands we rely on.

**Contents**
- [Entrypoint behavior](#entrypoint-behavior)
- [CLI commands & flags used](#cli-commands--flags-used)
- [Sources](#sources)

### Entrypoint behavior

**Container Startup Sequence:**
1. **Environment validation**: Check required variables (APP_KEY, DB_* settings)
2. **Database connection**: Wait for database readiness and test connectivity
3. **Migration execution**: Run any pending Laravel migrations automatically
4. **Cache optimization**: Generate configuration and route cache for performance
5. **Application start**: Launch PHP-FPM process to handle web requests

**Startup Scripts:**
- `/entrypoint.sh`: Primary container entrypoint script
- Migration check: `php artisan migrate --force` (auto-runs on startup)
- Cache warming: `php artisan config:cache && php artisan route:cache`
- Health verification: Internal health checks before marking container ready

### CLI commands & flags used

| Command | Purpose | Usage Example | Notes |
|---------|---------|---------------|-------|
| `php artisan migrate` | Database schema management | `--force` for non-interactive | Auto-executed on startup |
| `php artisan key:generate` | Generate application encryption key | `--show` to display key | Required for initial setup |
| `php artisan config:cache` | Cache configuration for performance | No flags | Performance optimization |
| `php artisan route:cache` | Cache routes for faster routing | No flags | Performance optimization |
| `php artisan firefly-iii:create-user` | Create new user account | `--email=user@example.com` | User management |
| `php artisan firefly-iii:reset-password` | Reset user password | `--email=user@example.com` | Password recovery |
| `php artisan queue:work` | Process background jobs | `--daemon --sleep=3` | Queue processing |
| `php artisan schedule:run` | Execute scheduled tasks | Typically via cron | Recurring operations |

**Development Commands:**
```bash
# Generate application key
docker compose run --rm firefly-iii-application.application \
  php artisan key:generate --show

# Run migrations manually
docker compose exec firefly-iii-application.application \
  php artisan migrate

# Create admin user
docker compose exec firefly-iii-application.application \
  php artisan firefly-iii:create-user --email=admin@example.com
```

**Production Considerations:**
- All commands run as www-data user for security
- Critical commands include error handling and logging
- Database commands include connection timeout and retry logic
- Cache commands are safe to run repeatedly

### Sources
- "Firefly III Docker Entrypoint" — https://github.com/firefly-iii/docker/blob/main/entrypoint.sh — retrieved 2025-01-09
- "Laravel Artisan Commands" — https://laravel.com/docs/10.x/artisan — retrieved 2025-01-09
- "Firefly III Command Reference" — https://docs.firefly-iii.org/references/firefly-iii/command-line — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://github.com/firefly-iii/docker/blob/main/entrypoint.sh":"sha256:pending","https://laravel.com/docs/10.x/artisan":"sha256:pending","https://docs.firefly-iii.org/references/firefly-iii/command-line":"sha256:pending"},"sections":{"entrypoint":"sha256:pending"}}}
-->