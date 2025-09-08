---
title: "Application Overview"
description: "What Firefly III is and what this repository packages."
tags: [overview, upstream]
search: { boost: 4, exclude: false }
icon: material/information
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Summarize the Firefly III application and packaging approach.

**Contents**
- [What is Firefly III?](#what-is-firefly-iii)
- [Core features](#core-features)
- [Supported versions & lifecycle](#supported-versions--lifecycle)
- [Sources](#sources)

## What is Firefly III?

Firefly III is a free, open source, self-hosted personal finance manager. It can help you keep track of your expenses and income, so you can spend less and save more. Firefly III supports the use of budgets, categories and tags, and can import data from external sources. It's built with the Laravel framework in PHP and uses a MySQL or PostgreSQL database.

This repository provides a complete containerized deployment of Firefly III with supporting services (MariaDB database, Redis cache, Nginx reverse proxy) using Docker Compose and Kubernetes manifests.

## Core features

- **Transaction Management**: Track and categorize income, expenses, and transfers
- **Account Types**: Support for asset accounts, expense accounts, revenue accounts, and liabilities
- **Budgets & Categories**: Organize spending with budgets and detailed categorization
- **Bills & Recurring Transactions**: Automated handling of recurring payments and income
- **Data Import**: Import from banks, CSV files, and other financial software
- **Multi-Currency Support**: Handle multiple currencies with automatic conversion
- **Reports & Charts**: Comprehensive financial reporting and visualization
- **Rules Engine**: Automatic transaction categorization and processing
- **API Access**: RESTful API for third-party integrations
- **Data Export**: Export financial data in various formats

## Supported versions & lifecycle

- **Current Version**: Firefly III follows semantic versioning with regular feature releases
- **LTS Policy**: No formal LTS policy; users should stay on latest stable release
- **PHP Requirements**: PHP 8.2+ required for current versions
- **Database Support**: MySQL 8.0+, MariaDB 10.9+, PostgreSQL 13+
- **Update Cadence**: Feature releases every few months, patch releases as needed

## Sources
- "Firefly III Documentation Home" — https://docs.firefly-iii.org — retrieved 2025-01-09
- "Firefly III About Page" — https://docs.firefly-iii.org/how-to/firefly-iii/about/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org":"","https://docs.firefly-iii.org/how-to/firefly-iii/about/":""},"sections":{"overview":""}}}
-->