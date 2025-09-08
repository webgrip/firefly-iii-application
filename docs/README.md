---
title: "Invoice Ninja Application — Quickstart & Overview"
description: "How to run and verify the Invoice Ninja application locally using Make + Compose."
tags: [quickstart, compose, onboarding]
search: { boost: 4, exclude: false }
icon: material/home
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Help you start the Invoice Ninja stack and reach the application fast.

**Contents**
- [Prerequisites](#prerequisites)
- [Quickstart](#quickstart)
- [Verification](#verification)
- [What this project packages](#what-this-project-packages)
- [Sources](#sources)

### Prerequisites
- Docker (version 20.10+ recommended)
- Docker Compose (version 2.0+ or Docker Desktop with Compose)
- GNU Make

### Quickstart
```bash
# Clone and navigate to repository
git clone <repository-url>
cd firefly-iii-application

# Bootstrap environment and start services
cp .env.example .env          # Copy environment template
make start                    # Start all services; wait for healthchecks to pass
make logs                     # View service logs (optional)

# Wait for services to be ready
make wait-ready URL=http://localhost:8080

# Create initial admin user (optional)
make user:create EMAIL=admin@example.com PASS=password
```

### Verification

- **Primary Access**: Open `http://localhost:8080` → should load the Invoice Ninja application interface
- **Health Check**: Run `docker ps` → all containers should show `healthy` status
- **Database Connection**: Application should connect to MariaDB automatically
- **Cache/Sessions**: Redis should be operational for session storage

Expected behavior on first access:
- Invoice Ninja setup/welcome screen or login page
- No database connection errors
- Responsive web interface

### What this project packages

This repository provides a containerized deployment of **Invoice Ninja**, a self-hosted invoicing and billing application. Invoice Ninja is an open-source solution for freelancers and small businesses to create invoices, track expenses, manage clients, and handle payments.

**Key Features Provided:**
- Complete Invoice Ninja application stack
- MariaDB database for data persistence  
- Redis for caching, sessions, and queue management
- Nginx reverse proxy for web serving
- Development-ready configuration with Docker Compose
- Production-ready Helm charts for Kubernetes deployment

**Repository Scope:**
- Custom container images built from upstream Invoice Ninja
- Environment configuration optimized for containerized deployment
- Automated setup and management via Make targets
- Integrated secrets management with SOPS/age encryption

### Sources

*Research in progress - official sources to be documented*

- "Invoice Ninja Documentation" — https://invoiceninja.github.io/ — retrieved 2025-01-09
- "Invoice Ninja GitHub Repository" — https://github.com/invoiceninja/invoiceninja — retrieved 2025-01-09
- "Invoice Ninja Docker Hub" — https://hub.docker.com/r/invoiceninja/invoiceninja — retrieved 2025-01-09