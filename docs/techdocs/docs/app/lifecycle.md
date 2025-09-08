---
title: "Lifecycle & Support Policy"
description: "Documented support windows, LTS, and deprecation cadence."
tags: [versions, lifecycle, support]
search: { boost: 3, exclude: false }
icon: material/timeline-clock
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Understand Invoice Ninja version lifecycle and our support policy for container deployments.

**Contents**
- [Upstream Version Strategy](#upstream-version-strategy)
- [Container Image Lifecycle](#container-image-lifecycle)
- [Support Windows](#support-windows)
- [Upgrade Policy](#upgrade-policy)
- [End-of-Life Process](#end-of-life-process)
- [Sources](#sources)

## Upstream Version Strategy

### Invoice Ninja Release Cadence
Invoice Ninja follows a **continuous delivery model** with regular releases that include:

- **Major Releases**: Significant feature additions and breaking changes (approximately yearly)
- **Minor Releases**: New features and enhancements (monthly to quarterly)
- **Patch Releases**: Bug fixes and security updates (as needed, typically bi-weekly)
- **Security Releases**: Critical security fixes (immediate, out-of-band)

### Version Numbering
Invoice Ninja uses **semantic versioning** (SemVer):
- `MAJOR.MINOR.PATCH` format (e.g., `5.8.42`)
- Breaking changes increment MAJOR version
- New features increment MINOR version  
- Bug fixes increment PATCH version

### Current Supported Versions
As of 2025-01-09, Invoice Ninja actively supports:

| Version | Status | Release Date | Support Until | Notes |
|---------|--------|-------------|---------------|-------|
| 5.8.x | **Current Stable** | 2024-Q4 | Active | Recommended for production |
| 5.7.x | **Maintenance** | 2024-Q3 | 2025-Q2 | Security fixes only |
| 5.6.x | **End of Life** | 2024-Q2 | 2024-Q4 | No longer supported |
| 4.x.x | **Legacy** | 2019-2023 | Ended 2024-Q1 | Migration required |

**Current Deployment Version**: Invoice Ninja v5.8.42 (as configured in container images)

## Container Image Lifecycle

### Our Image Versioning Strategy
WebGrip container images follow this versioning pattern:
- `webgrip/firefly-iii-application:5.8.42-1` (Invoice Ninja version + build number)
- `webgrip/firefly-iii-application:5.8.42-alpine` (base OS variant)
- `webgrip/firefly-iii-application:latest` (development only, not for production)

### Base Image Updates
We monitor and update base images for:
- **Security Updates**: Alpine Linux security advisories (weekly review)
- **PHP Updates**: PHP 8.1/8.2 security and feature releases
- **Library Updates**: Composer dependencies and system packages

### Image Support Timeline
| Component | Update Frequency | Support Window |
|-----------|------------------|----------------|
| **Invoice Ninja Application** | Follow upstream releases | Match upstream support |
| **PHP Runtime** | Monthly security review | Current + previous major |
| **Alpine Base** | Weekly security scan | Current stable + 6 months |
| **MariaDB** | Quarterly LTS alignment | LTS versions only |
| **Redis** | Stable release tracking | Current stable |
| **Nginx** | Stable release tracking | Current stable |

## Support Windows

### Production Support
- **Current Version**: Full support including features, bug fixes, and security updates
- **Previous Minor**: Security updates and critical bug fixes for 6 months
- **Previous Major**: Security updates only for 12 months after new major release

### Development Support
- **Latest Development**: Always uses most recent stable upstream release
- **Feature Branches**: Use specific version tags for reproducible builds
- **Security Scanning**: All images scanned for vulnerabilities before release

### Emergency Support
- **Critical Security Issues**: Hotfix images built within 24-48 hours
- **Data Loss Prevention**: Emergency rollback procedures maintained
- **Service Continuity**: Zero-downtime deployment capabilities for security patches

## Upgrade Policy

### Automated Upgrades
- **Patch Releases**: Automatic upgrade recommendations for security fixes
- **Minor Releases**: Staged upgrade process with testing period
- **Major Releases**: Manual upgrade process with migration validation

### Upgrade Testing
Before any production upgrade:
1. **Database Backup**: Full backup with verified restore capability
2. **Staging Deployment**: Test upgrade on staging environment
3. **Migration Validation**: Verify data integrity and application functionality
4. **Rollback Testing**: Confirm rollback procedures work correctly
5. **Performance Testing**: Validate performance meets SLA requirements

### Breaking Changes
When upstream introduces breaking changes:
- **Migration Guides**: Detailed upgrade documentation provided
- **Deprecation Warnings**: 30-day advance notice minimum
- **Compatibility Shims**: Temporary compatibility layers where possible
- **Support Extension**: Extended support for previous version during transition

## End-of-Life Process

### End-of-Life Timeline
When Invoice Ninja versions reach end-of-life:

**90 Days Before EOL:**
- Notification to all stakeholders
- Upgrade recommendations provided
- Testing environments updated

**30 Days Before EOL:**
- Final security updates applied
- Documentation marked as deprecated
- New deployment restrictions implemented

**EOL Date:**
- No further updates to container images
- Images remain available but marked as unsupported
- Security scanning continues for 30 days

**30 Days After EOL:**
- Images moved to archived status
- Container registry cleanup initiated
- Documentation archived

### Migration Support
For end-of-life versions, we provide:
- **Migration Documentation**: Step-by-step upgrade procedures
- **Data Export Tools**: Scripts for data extraction and migration
- **Rollback Plans**: Emergency procedures if migration fails
- **Extended Support**: Optional paid support for delayed migrations

## Sources

- "Invoice Ninja Release Notes" — https://github.com/invoiceninja/invoiceninja/releases — retrieved 2025-01-09
- "Invoice Ninja Version Support Policy" — https://invoiceninja.github.io/en/release-notes/ — retrieved 2025-01-09
- "Semantic Versioning Specification" — https://semver.org/ — retrieved 2025-01-09
- "Laravel Support Policy" — https://laravel.com/docs/10.x/releases — retrieved 2025-01-09
- "Alpine Linux Security Advisories" — https://alpinelinux.org/security/ — retrieved 2025-01-09