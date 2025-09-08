---
title: "Containerization & Compose Policy"
description: "Org-owned images from pinned upstream bases; official app image preferred, source-build only if unsupported or unavailable."
tags: [operations, containerization, docker, compose, policy]
search: { boost: 4, exclude: false }
icon: material/steam
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Record our image and Compose policies for this Invoice Ninja repository.

**Contents**
- [Policy](#policy)
- [Rationale](#rationale)
- [Consequences](#consequences)
- [Sources](#sources)

## Policy

### Image Strategy
- Use **org-owned images** (`webgrip/firefly-iii-application:*`) derived from **pinned upstream bases** (no `latest` tags in production)
- Build custom images from source due to specific configuration requirements for our deployment environment
- Pin all base images to specific versions for reproducibility and security
- All images built locally with `pull_policy: never` for development consistency

### Compose Configuration
- Compose uses **only our custom images** with explicit version pinning
- Healthchecks implemented for all critical services (database, cache, application)
- Service dependencies properly defined with `depends_on` conditions
- Volume persistence configured for application data and database storage

### Service Architecture
- **Application Container**: Custom PHP/Laravel build with Invoice Ninja source
- **Web Server**: Custom Nginx container with optimized configuration
- **Database**: Custom MariaDB container with proper character sets and collation
- **Cache**: Custom Redis container for sessions, cache, and queue management

## Rationale

### Security and Compliance
- **Supply Chain Security**: Custom builds allow full control over base images and dependencies
- **Vulnerability Management**: Faster CVE response through controlled base image updates
- **Secret Management**: Integration with SOPS/age for encrypted configuration
- **Compliance**: Auditable build process with Software Bill of Materials (SBOM) capability

### Operational Benefits
- **Reproducibility**: Pinned versions ensure consistent deployments across environments
- **Customization**: Tailored configuration for WebGrip deployment requirements
- **Performance**: Optimized images with only necessary components
- **Debugging**: Local build capability for faster development iteration

### Risk Management
- **Isolation**: Custom images reduce dependency on external registry availability
- **Rollback**: Version pinning enables reliable rollback procedures
- **Testing**: Complete control over image contents enables thorough security scanning

## Consequences

### Advantages
- **Security**: Reduced attack surface through minimal custom images
- **Reliability**: Controlled update process reduces unexpected breaking changes
- **Compliance**: Full audit trail of all components and dependencies
- **Performance**: Optimized builds specific to deployment requirements

### Trade-offs
- **Maintenance Overhead**: Responsibility for monitoring upstream security updates
- **Build Complexity**: Custom Dockerfiles require maintenance and testing
- **Update Cadence**: Manual coordination required for base image updates
- **Storage**: Multiple custom images increase registry storage requirements

### Mitigation Strategies
- **Automated Scanning**: Integrate security scanning into CI/CD pipeline
- **Update Monitoring**: Automated notifications for upstream base image updates
- **Testing Pipeline**: Comprehensive testing of image builds before deployment
- **Documentation**: Clear procedures for emergency security updates

## Sources

- "Invoice Ninja Self-Hosting Guide" — https://invoiceninja.github.io/en/self-host-installation/ — retrieved 2025-01-09
- "Docker Official Images Best Practices" — https://docs.docker.com/develop/dev-best-practices/ — retrieved 2025-01-09
- "Container Security Best Practices" — https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html — retrieved 2025-01-09