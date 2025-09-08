---
title: "Firefly III Application — Quickstart & Overview"
description: "How to run and verify the Firefly III personal finances manager locally using Make + Compose."
tags: [quickstart, compose, onboarding, firefly-iii, personal-finance]
search: { boost: 4, exclude: false }
icon: material/home
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Help you start the Firefly III personal finances manager stack and reach the app fast.

**Contents**
- [Prerequisites](#prerequisites)
- [Quickstart](#quickstart)
- [Verification](#verification)
- [What this project packages](#what-this-project-packages)
- [Sources](#sources)

### Prerequisites
- Docker (minimum version 20.10.0)
- Docker Compose (plugin version 2.0+)
- GNU Make

### Quickstart
```bash
make start       # start services; wait for healthchecks to pass
make logs        # tail logs
make stop        # stop services
```

### Verification

* Open: `http://localhost:8080` → should load the Firefly III login page.
* Healthcheck: container reports `healthy` in `docker ps`.
* Default credentials: Create an account through the registration flow.

### What this project packages

Firefly III is a free, open source personal finance manager that helps you track your expenses, manage budgets, and gain insights into your financial habits. This repository provides a containerized deployment of Firefly III using Docker Compose with MariaDB for data persistence and Redis for caching and session management.

This packaging includes:
- **Application**: Firefly III web interface and API
- **Database**: MariaDB for data persistence  
- **Cache**: Redis for session and cache storage
- **Reverse Proxy**: Nginx for web serving

### Sources

* "Firefly III - A free and open source personal finance manager" — https://github.com/firefly-iii/firefly-iii — retrieved 2025-01-09
* "Firefly III Documentation" — https://docs.firefly-iii.org — retrieved 2025-01-09

<!-- ai-docs-metadata
{
  "last_audit": "2025-01-09",
  "fingerprints": {
    "sources": {
      "https://github.com/firefly-iii/firefly-iii": "sha256:pending",
      "https://docs.firefly-iii.org": "sha256:pending"
    },
    "sections": {
      "quickstart": "sha256:pending",
      "verification": "sha256:pending"
    }
  }
}
-->