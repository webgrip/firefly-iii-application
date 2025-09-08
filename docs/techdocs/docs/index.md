# Firefly III Application Documentation

**Purpose**: This documentation provides comprehensive information about the Firefly III application deployment, configuration, and operational guidance for this containerized implementation.

## Table of Contents

- [Project Overview](#project-overview)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Architecture](#architecture)
- [Operations](#operations)
- [Troubleshooting](#troubleshooting)

## Project Overview

This repository packages [Firefly III](https://www.firefly-iii.org/), a free and open-source personal finance manager, into a containerized application stack suitable for deployment in both development and production environments.

**Firefly III** is a web-based personal finance manager that helps you track your expenses, manage budgets, and monitor your financial health. It supports multiple currencies, bank account reconciliation, and provides detailed reporting capabilities.

### Architecture Components

- **Application**: PHP 8.x Laravel application running on PHP-FPM
- **Web Server**: Nginx reverse proxy handling HTTP requests
- **Database**: MariaDB 12.x for data persistence
- **Cache**: Redis for session storage and application caching

### Key Features Provided

- Multi-user personal finance management
- Transaction tracking and categorization
- Budget management and monitoring
- Multi-currency support
- Bank account reconciliation
- Comprehensive reporting and analytics
- API access for integrations

## Quick Start

To start the application stack locally:

```bash
# Copy environment configuration
cp .env.example .env

# Start all services
make start

# Access the application
open http://localhost:8080
```

For detailed setup instructions, see the [Getting Started](getting-started.md) guide.

## Documentation Sections

- **[Getting Started](getting-started.md)** - Initial setup and configuration
- **[Configuration](configuration.md)** - Environment variables and settings
- **[Architecture](architecture.md)** - System design and component overview
- **[Deployment](deployment.md)** - Production deployment guidance
- **[Operations](operations.md)** - Day-to-day operational procedures
- **[Troubleshooting](troubleshooting.md)** - Common issues and solutions
- **[Security](security.md)** - Security considerations and best practices

---

**Note**: This documentation is based on official Firefly III sources and adapted for this containerized deployment. All behavior documented aligns with upstream Firefly III guidelines and best practices.
