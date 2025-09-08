# Firefly III Application

[![Template Sync](https://img.shields.io/github/actions/workflow/status/webgrip/application-template/sync-template-files.yml?label=template%20sync&style=flat-square)](https://github.com/webgrip/application-template/actions/workflows/sync-template-files.yml)
[![License](https://img.shields.io/github/license/webgrip/application-template?style=flat-square)](LICENSE)
[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-orange.svg?style=flat-square)](https://www.conventionalcommits.org)
[![SemVer](https://img.shields.io/badge/semver-2.0.0-blue?style=flat-square)](https://semver.org)
[![Dockerized](https://img.shields.io/badge/containerized-docker-2496ED?logo=docker&logoColor=white&style=flat-square)](https://www.docker.com/)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square)](https://github.com/webgrip/application-template/issues)

> Containerized Firefly III personal finance manager with production-ready deployment capabilities, automated secret management, and comprehensive operational documentation.

---

## Project Overview

This repository packages **Firefly III**, a free and open-source personal finance manager, into a containerized application stack suitable for deployment across development, staging, and production environments. Firefly III helps users track expenses, manage budgets, reconcile accounts, and gain insights into their financial health through comprehensive reporting and analytics.

### What is Firefly III?

Firefly III is a web-based personal finance manager that provides:
- **Transaction Management**: Track income, expenses, and transfers across multiple accounts
- **Budget Monitoring**: Set and monitor budgets with visual feedback and alerts
- **Multi-Currency Support**: Handle international finances with currency conversion
- **Account Reconciliation**: Match transactions with bank statements
- **Comprehensive Reporting**: Generate detailed financial reports and analytics
- **API Access**: RESTful API for integrations and mobile applications
- **Self-Hosted Privacy**: Complete control over your financial data

## At a Glance

| Aspect | What You Get |
| ------ | ------------- |
| **Application** | Firefly III v6.x with Laravel 11.x framework |
| **Architecture** | Multi-container stack (App + Nginx + MariaDB + Redis) |
| **Security** | Encrypted secrets, HTTPS support, data sovereignty |
| **Operations** | Automated backups, monitoring, health checks |
| **Deployment** | Docker Compose for local dev, Helm charts for Kubernetes |
| **Documentation** | Comprehensive guides for setup, operation, and troubleshooting |

## Quick Start

Get Firefly III running locally in under 5 minutes:

```bash
# 1. Clone the repository
git clone https://github.com/webgrip/firefly-iii-application.git
cd firefly-iii-application

# 2. Set up environment
cp .env.example .env

# 3. Generate application encryption key
echo "APP_KEY=base64:$(openssl rand -base64 32)" >> .env

# 4. Start all services
make start

# 5. Access the application
open http://localhost:8080
```

The application will be available at `http://localhost:8080` where you can complete the setup wizard and create your first user account.

## Features

### Core Financial Management
- **Double-Entry Bookkeeping**: Accurate transaction tracking with source and destination accounts
- **Multi-Currency Support**: Handle multiple currencies with automatic conversion rates
- **Budget Management**: Create and monitor budgets with spending alerts and visual progress
- **Account Types**: Support for asset, expense, revenue, and liability accounts
- **Transaction Rules**: Automated transaction categorization and processing
- **Recurring Transactions**: Set up automatic recurring income and expenses

### Reporting and Analytics
- **Financial Reports**: Income/expense reports, budget analysis, account summaries
- **Visual Charts**: Spending trends, budget progress, account balances over time
- **Export Capabilities**: CSV and JSON export for external analysis
- **Tag System**: Flexible tagging for advanced transaction organization

### Security and Privacy
- **Self-Hosted**: Complete control over financial data with no third-party dependencies
- **User Authentication**: Secure login with optional multi-factor authentication
- **Data Encryption**: Financial data encrypted at rest and in transit
- **Audit Logging**: Complete audit trail of all financial transactions and changes

### Technical Features
- **RESTful API**: Complete API access for custom integrations and mobile apps
- **Import/Export**: Support for various bank formats (CSV, OFX, etc.)
- **Backup System**: Automated database backups with point-in-time recovery
- **Health Monitoring**: Built-in health checks and monitoring endpoints

## Architecture

### Container Services
- **Application**: PHP 8.x + Laravel 11.x running Firefly III
- **Web Server**: Nginx configured for optimal Laravel performance
- **Database**: MariaDB 12.x with UTF8MB4 support for international data
- **Cache**: Redis for session storage and application caching

### Deployment Options
- **Local Development**: Docker Compose with live reload and debugging
- **Production**: Kubernetes deployment with Helm charts
- **Cloud**: Support for AWS, Azure, and Google Cloud deployments

## Documentation

Comprehensive documentation is available in the `docs/` directory:

- **[Getting Started](docs/techdocs/docs/getting-started.md)** - Initial setup and first-time configuration
- **[Configuration](docs/techdocs/docs/configuration.md)** - Environment variables and system settings
- **[Architecture](docs/techdocs/docs/architecture.md)** - System design and component overview
- **[Deployment](docs/techdocs/docs/deployment.md)** - Production deployment procedures
- **[Operations](docs/techdocs/docs/operations.md)** - Day-to-day operational procedures
- **[Security](docs/techdocs/docs/security.md)** - Security configuration and best practices
- **[Troubleshooting](docs/techdocs/docs/troubleshooting.md)** - Common issues and solutions

### Architectural Decision Records
- **[ADR-004](docs/adrs/0004-application-firefly-iii-finance-manager.md)** - Decision to use Firefly III
- **[ADR-001](docs/adrs/0001-framework-docker-for-local-development.md)** - Docker for development
- **[ADR-002](docs/adrs/0002-framework-kubernetes-as-deployment-platform.md)** - Kubernetes deployment
- **[ADR-003](docs/adrs/0003-framework-helm-for-deployment.md)** - Helm deployment automation

## Security and Secrets Management

### Encrypted Secrets

This repository uses SOPS with age encryption for secure secret management:

```bash
# Initialize encryption keys
make secrets:init

# Edit secrets (creates age.agekey and age.pubkey)
# Add age.agekey as SOPS_AGE_KEY in repository secrets

# Configure application secrets
vim ops/secrets/firefly-iii-application-secrets/values.dec.yaml

# Encrypt secrets for repository storage
make secrets:encrypt SECRETS_DIR=./ops/secrets/firefly-iii-application-secrets
```

### Production Security

- **HTTPS Enforcement**: SSL/TLS termination with automatic certificate renewal
- **Secret Management**: Encrypted secrets with rotation capabilities
- **Access Controls**: Role-based access and API authentication
- **Data Protection**: Financial data encryption and secure backup procedures

## Operations

### Daily Operations
```bash
# Start services
make start

# View logs
make logs

# Stop services
make stop

# Create user account
make user:create EMAIL=admin@example.com PASS=securepassword

# Access application container
make enter
```

### Monitoring and Maintenance
```bash
# Check service health
curl -f http://localhost:8080/health

# Run database maintenance
make run CMD="php artisan firefly-iii:verify-database"

# Create backup
make backup

# Update application
git pull && make start
```

## Compliance with Upstream

**Verification Statement**: This deployment follows official Firefly III installation and configuration practices as documented at https://docs.firefly-iii.org/. All environment variables, container configurations, and operational procedures align with upstream recommendations.

**No Deviations**: This repository implements standard Firefly III deployment patterns with additional operational enhancements for container orchestration and automated operations.

## Contributing

Contributions welcome! Please:

1. Open an issue describing the change
2. Use Conventional Commits for branch + commit messages
3. Add / adjust tests where behavior changes
4. Update docs (README / TechDocs / ADRs) when altering architecture

## License

Distributed under the terms of the MIT license. See `LICENSE` for details.

---

**Firefly III** is developed by James Cole and the Firefly III community. This repository provides containerized deployment and operational tooling. Visit the [official Firefly III website](https://www.firefly-iii.org/) for more information about the application itself.
