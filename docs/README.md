---
title: "firefly-iii-application — Quickstart & Overview"
description: "How to run and verify the Firefly III application locally using Make + Compose."
tags: [quickstart, compose, onboarding]
search: { boost: 4, exclude: false }
icon: material/home
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Help you start the Firefly III stack and reach the app fast.

**Contents**
- [Prerequisites](#prerequisites)
- [Quickstart](#quickstart)
- [Verification](#verification)
- [What this project packages](#what-this-project-packages)
- [Sources](#sources)

## Prerequisites
- Docker 20.10+ (for compose v2 support)
- Docker Compose (if separate installation)
- GNU Make

## Quickstart
```bash
# First time setup
cp .env.example .env  # copy environment template
make start            # start services; wait for healthchecks to pass
make logs             # tail logs
make stop             # stop services
```

## Verification

* Open: `http://localhost:8080` → should load the Firefly III landing page.
* Healthcheck: containers report `healthy` in `docker ps`.

## What this project packages

Firefly III is a free, open source, self-hosted personal finance manager. This repository provides a complete containerized deployment of Firefly III with supporting services (database, cache, reverse proxy) using Docker Compose and Kubernetes manifests.

## Sources

* "Firefly III Documentation" — https://docs.firefly-iii.org — retrieved 2025-01-09

<!-- ai-docs-metadata
{
  "last_audit": "2025-01-09",
  "fingerprints": {
    "sources": {
      "https://docs.firefly-iii.org": ""
    },
    "sections": {
      "quickstart": "",
      "verification": ""
    }
  }
}
-->