---
title: "Lifecycle & Support Policy"
description: "Documented support windows, LTS, and deprecation cadence."
tags: [versions, lifecycle, firefly-iii]
search: { boost: 3, exclude: false }
icon: material/timeline-clock
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Make upgrade expectations explicit.

**Contents**
- [Policy](#policy)
- [Currently supported versions](#currently-supported-versions)
- [Upgrade guidance](#upgrade-guidance)
- [Sources](#sources)

### Policy

**Upstream Policy (Firefly III):**
- **Release Cadence**: Regular releases every 2-3 months
- **Support Model**: Latest stable version + critical security patches for previous version
- **Breaking Changes**: Major versions may include breaking changes with migration paths
- **PHP Compatibility**: Follows PHP's active support lifecycle
- **Database Support**: Maintains compatibility with LTS database versions

**Our Repository Policy:**
- **Tracking**: Follow latest stable upstream releases within 2 weeks
- **Testing**: All updates tested in staging before production promotion
- **Rollback**: Previous version images retained for 90 days
- **Security**: Critical security updates deployed within 24 hours

### Currently supported versions

| Version | Status | PHP Requirement | Database Support | EOL Date |
|---------|--------|-----------------|------------------|----------|
| 6.1.x | Current Stable | PHP 8.1+ | MySQL 8.0+, MariaDB 10.3+, PostgreSQL 12+ | TBD |
| 6.0.x | Security Only | PHP 8.1+ | MySQL 8.0+, MariaDB 10.3+, PostgreSQL 12+ | 2025-06-01 |
| 5.8.x | End of Life | PHP 8.0+ | MySQL 5.7+, MariaDB 10.2+ | 2024-12-31 |

### Upgrade guidance

**Pre-Upgrade Checklist:**
1. **Backup**: Complete database and file storage backup
2. **Version Check**: Review upstream release notes for breaking changes
3. **Dependencies**: Verify PHP and database version compatibility
4. **Testing**: Test upgrade in staging environment first
5. **Rollback Plan**: Ensure rollback procedure is documented and tested

**Upgrade Process:**
```bash
# 1. Backup current state
make backup

# 2. Stop services
make stop

# 3. Update image tags in docker-compose.yml
# Edit: image: webgrip/firefly-iii-application:6.1.0

# 4. Start with new version
make start

# 5. Monitor logs for migration completion
make logs

# 6. Verify application functionality
curl http://localhost:8080/health
```

**Rollback Procedure:**
```bash
# 1. Stop services
make stop

# 2. Restore previous image tag
# Edit docker-compose.yml back to previous version

# 3. Restore database if needed
docker compose exec firefly-iii-application.mariadb mysql \
  -u application -papplication application < backup-pre-upgrade.sql

# 4. Restart with previous version
make start
```

**Version Pinning:**
- Production environments should pin specific version tags
- Never use `latest` tag in production
- Update tags in controlled releases after testing

### Sources
- "Firefly III Release Notes" — https://github.com/firefly-iii/firefly-iii/releases — retrieved 2025-01-09
- "Firefly III Upgrade Guide" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/upgrade — retrieved 2025-01-09
- "PHP Supported Versions" — https://www.php.net/supported-versions.php — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://github.com/firefly-iii/firefly-iii/releases":"sha256:pending","https://docs.firefly-iii.org/how-to/firefly-iii/installation/upgrade":"sha256:pending","https://www.php.net/supported-versions.php":"sha256:pending"},"sections":{"policy":"sha256:pending"}}}
-->