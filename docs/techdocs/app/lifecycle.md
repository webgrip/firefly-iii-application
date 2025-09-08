---
title: "Lifecycle & Support Policy"
description: "Documented support windows, versioning, and upgrade guidance."
tags: [versions, lifecycle]
search: { boost: 3, exclude: false }
icon: material/timeline-clock
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Make upgrade expectations explicit.

**Contents**
- [Policy](#policy)
- [Currently supported versions](#currently-supported-versions)
- [Upgrade guidance](#upgrade-guidance)
- [Sources](#sources)

## Policy

Firefly III follows a **rolling release model** with semantic versioning:

- **Major versions** (e.g., 6.x) introduce breaking changes or significant architecture updates
- **Minor versions** (e.g., 6.1.x) add new features while maintaining backward compatibility
- **Patch versions** (e.g., 6.1.1) fix bugs and security issues

**Support Policy:**
- Only the latest stable version receives active development and support
- Critical security fixes may be backported to the previous major version
- Users are encouraged to upgrade regularly to benefit from bug fixes and new features
- Database migrations are provided for all version upgrades

## Currently supported versions

| Version | Status | PHP Requirement | Database Support | Notes |
|---------|--------|----------------|------------------|--------|
| 6.x | Current Stable | PHP 8.2+ | MySQL 8.0+, MariaDB 10.9+, PostgreSQL 13+ | Active development |
| 5.x | Legacy | PHP 8.1+ | MySQL 5.7+, PostgreSQL 10+ | Security fixes only |

## Upgrade guidance

**Before upgrading:**
1. Create a full database backup
2. Test the upgrade in a staging environment
3. Review the changelog for breaking changes
4. Ensure PHP and database versions meet requirements

**Upgrade process:**
1. Stop the Firefly III application
2. Update container images to the target version
3. Run database migrations (automatic on container startup)
4. Verify application functionality
5. Update any custom configurations if needed

**Rollback procedure:**
1. Stop the application
2. Restore database from backup
3. Revert to previous container image version
4. Restart services

**Tag management in this repository:**
- Production deployments use specific version tags (e.g., `6.1.17`)
- Never use `latest` tag in production
- Update tags through pull requests with proper testing

## Sources
- "Firefly III Upgrade Guide" — https://docs.firefly-iii.org/how-to/firefly-iii/upgrade/ — retrieved 2025-01-09
- "Firefly III Installation Guide" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/upgrade/":"","https://docs.firefly-iii.org/how-to/firefly-iii/installation/":""},"sections":{"policy":""}}}
-->