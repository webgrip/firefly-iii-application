---
title: "Application Overview"
description: "What Firefly III is and what this repository packages."
tags: [overview, upstream, firefly-iii, personal-finance]
search: { boost: 4, exclude: false }
icon: material/information
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Summarize Firefly III and our packaging approach.

**Contents**
- [What is Firefly III?](#what-is-firefly-iii)
- [Core features](#core-features)
- [Supported versions & lifecycle](#supported-versions--lifecycle)
- [Sources](#sources)

### What is Firefly III?

Firefly III is a free, open source personal finance manager that helps individuals track their expenses, manage budgets, and gain insights into their financial habits. Written in PHP using the Laravel framework, it provides a web-based interface for comprehensive financial management.

This repository packages Firefly III as a containerized solution using Docker Compose, with MariaDB for data persistence, Redis for caching and session management, and Nginx as a reverse proxy.

### Core features

- **Account Management**: Track checking, savings, credit cards, and asset accounts
- **Transaction Tracking**: Record income, expenses, and transfers with detailed categorization  
- **Budget Management**: Set and monitor budgets with visual progress tracking
- **Bill Management**: Automate recurring bill tracking and predictions
- **Reporting**: Generate comprehensive financial reports with charts and analytics
- **Rule Engine**: Automate transaction categorization and processing
- **Multi-Currency Support**: Handle multiple currencies with conversion tracking
- **Data Import**: Import transactions from banks via CSV, APIs, and other formats
- **REST API**: Programmatic access for integrations and automation

### Supported versions & lifecycle

- **Current Version**: 6.x series (actively developed)
- **LTS Policy**: No formal LTS, but stable releases receive security updates
- **Update Cadence**: Regular releases every 2-3 months
- **PHP Requirements**: PHP 8.1+ required for current versions
- **Database Support**: MySQL 8.0+, PostgreSQL 12+, SQLite (development only)

**Upgrade Strategy**: This repository tracks the latest stable release tags from upstream. Major version upgrades require data migration procedures documented in the operations runbook.

### Sources
- "Firefly III Official Documentation" — https://docs.firefly-iii.org — retrieved 2025-01-09
- "Firefly III About Page" — https://docs.firefly-iii.org/how-to/firefly-iii/about — retrieved 2025-01-09
- "Firefly III GitHub Releases" — https://github.com/firefly-iii/firefly-iii/releases — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org":"sha256:pending","https://docs.firefly-iii.org/how-to/firefly-iii/about":"sha256:pending","https://github.com/firefly-iii/firefly-iii/releases":"sha256:pending"},"sections":{"overview":"sha256:pending"}}}
-->