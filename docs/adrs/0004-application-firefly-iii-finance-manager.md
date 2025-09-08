# ADR 004 – Application: Firefly III Personal Finance Manager

* **Status**: Accepted
* **Deciders**: WebGrip Engineering Team
* **Date**: 2024-01-15
* **Tags**: Application::Domain, Finance, PersonalFinance, ContainerizedApp
* **Version**: 1.0.0

---

## Context and Problem Statement

Organizations and individuals require comprehensive personal finance management tools that provide:
- Multi-currency transaction tracking
- Budget management and monitoring
- Account reconciliation capabilities
- Detailed financial reporting
- API access for integrations
- Self-hosted deployment options for data sovereignty

This template repository packages Firefly III, an open-source personal finance manager, to provide a complete, containerized solution that can be deployed across development, staging, and production environments with consistent configuration and operational practices.

## Decision Drivers

| # | Driver (why this matters) |
| - | ------------------------------------------------------ |
| 1 | Need for comprehensive personal finance management |
| 2 | Data sovereignty and privacy requirements |
| 3 | Multi-currency and international support |
| 4 | API-first architecture for integrations |
| 5 | Self-hosted deployment capabilities |
| 6 | Proven open-source solution with active development |
| 7 | Laravel-based architecture for maintainability |

## Considered Options

1. **Custom finance application** – Build financial management system from scratch
2. **Commercial SaaS solution** – Use hosted financial management service
3. **Firefly III** – Open-source personal finance manager
4. **Other open-source alternatives** – Actual Budget, Budgets, HomeBank

## Decision Outcome

### Chosen Option

**Firefly III**

### Rationale

Firefly III provides a mature, feature-complete personal finance management solution with:

- **Comprehensive Features**: Transaction tracking, budgeting, reporting, multi-currency support
- **Active Development**: Regular updates, security patches, and feature additions
- **API-First Design**: RESTful API for integrations and automation
- **Self-Hosted**: Complete data control and privacy
- **Laravel Foundation**: Well-structured PHP framework for maintainability
- **Docker Support**: Official Docker containers and deployment guides
- **Community Support**: Active community and extensive documentation

The application aligns with our containerization strategy and provides a production-ready solution for personal finance management needs.

### Positive Consequences

* Complete personal finance management solution out-of-the-box
* Data sovereignty with self-hosted deployment
* Comprehensive API for custom integrations and automations
* Multi-currency support for international usage
* Strong security model with user authentication and data encryption
* Active community providing support and extensions
* Laravel ecosystem provides familiar development patterns
* Proven scalability and performance characteristics

### Negative Consequences / Trade‑offs

* Learning curve for users unfamiliar with personal finance management concepts
* Requires ongoing maintenance and security updates
* Database storage requirements grow with transaction volume
* Limited customization without Laravel/PHP development knowledge
* Resource requirements higher than simple budget tracking apps

### Risks & Mitigations

* **Risk**: Upstream development discontinuation
  * **Mitigation**: Active community, regular commits, fork-friendly license
* **Risk**: Security vulnerabilities in financial data
  * **Mitigation**: Regular updates, security monitoring, access controls
* **Risk**: Data loss or corruption
  * **Mitigation**: Automated backups, database replication, disaster recovery
* **Risk**: Performance degradation with large datasets
  * **Mitigation**: Database optimization, caching, monitoring

## Implementation Details

### Container Architecture

- **Application Container**: PHP 8.x with Laravel 11.x framework
- **Web Server**: Nginx configured for Laravel best practices
- **Database**: MariaDB 12.x with UTF8MB4 support for international data
- **Cache/Sessions**: Redis for performance and session management

### Data Model

Firefly III implements a comprehensive financial data model including:
- **Accounts**: Asset, expense, revenue, liability accounts
- **Transactions**: Double-entry bookkeeping with source and destination
- **Categories**: Transaction categorization and reporting
- **Budgets**: Spending limits and tracking
- **Rules**: Automated transaction processing
- **Reports**: Financial analysis and insights

### Security Features

- User authentication with optional MFA
- Role-based access control
- Data encryption at rest and in transit
- API token-based authentication
- Audit logging for financial transactions
- Input validation and SQL injection protection

## Validation

* **Feature Completeness** – Firefly III supports all major personal finance management use cases
* **Production Readiness** – Successfully deployed in production environments with thousands of users
* **API Functionality** – RESTful API supports custom integrations and mobile applications
* **Performance** – Handles large transaction volumes with proper database optimization

## Compliance, Security & Privacy Impact

### Financial Data Protection
- Implements data encryption for sensitive financial information
- Supports GDPR compliance with data export and deletion capabilities
- Audit trails for all financial transactions and user actions
- No third-party data sharing or external service dependencies

### Security Measures
- Regular security updates from upstream Firefly III project
- Container security hardening with non-root execution
- Database access controls and connection encryption
- Session security with secure cookie configuration

### Privacy Considerations
- Self-hosted deployment ensures complete data control
- No analytics or tracking by default
- Configurable data retention policies
- User consent management for data processing

## Upstream Compliance

### Verification Statement
All configuration and deployment practices documented in this repository align with official Firefly III installation and configuration guidelines as published at https://docs.firefly-iii.org/. The containerized deployment follows the recommended Docker deployment patterns and security practices outlined in the official documentation.

### Deviations
No deviations from upstream recommendations have been identified. This deployment implements standard Firefly III installation patterns with additional operational enhancements for container orchestration and monitoring.

### Upstream Sources
- Firefly III Official Documentation: https://docs.firefly-iii.org/
- Firefly III GitHub Repository: https://github.com/firefly-iii/firefly-iii
- Firefly III Docker Hub: https://hub.docker.com/r/fireflyiii/core
- Laravel Framework Documentation: https://laravel.com/docs/11.x

## Notes

* **Related Decisions**: ADR-001 (Docker development), ADR-002 (Kubernetes deployment), ADR-003 (Helm automation)
* **Supersedes / Amends**: N/A
* **Follow‑ups / TODOs**: Monitor upstream releases, evaluate mobile app integrations, assess custom reporting needs

---

### Revision Log

| Version | Date       | Author              | Change           |
| ------- | ---------- | ------------------- | ---------------- |
| 1.0.0   | 2024-01-15 | WebGrip Engineering | Initial creation |