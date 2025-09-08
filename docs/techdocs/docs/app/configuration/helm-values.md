---
title: "Helm — Used Values"
description: "Only the values from values.yaml that this repo actually uses."
tags: [helm, configuration, kubernetes]
search: { boost: 2, exclude: false }
icon: material/helm
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Keep Helm docs tight and truthful.

**Contents**
- [Values we set](#values-we-set)
- [Sources](#sources)

### Values we set

| Key | Example | Notes / Constraints | Source |
|-----|---------|---------------------|--------|
| `image.repository` | `webgrip/firefly-iii-application` | Org-owned image policy | "Container Policy ADR" — docs/adrs/2025-01-09-container-policy.md — retrieved 2025-01-09 |
| `image.tag` | `6.1.0` | Pinned version, no latest | "Container Policy ADR" — docs/adrs/2025-01-09-container-policy.md — retrieved 2025-01-09 |
| `env.APP_ENV` | `production` | Production environment | "Environment Variables" — docs/techdocs/docs/app/configuration/env.md — retrieved 2025-01-09 |
| `env.DB_CONNECTION` | `mysql` | Database type | "Database Integration" — docs/techdocs/docs/app/integrations/database.md — retrieved 2025-01-09 |
| `persistence.enabled` | `true` | Required for data persistence | "Files & Paths" — docs/techdocs/docs/app/configuration/files-and-paths.md — retrieved 2025-01-09 |
| `redis.enabled` | `true` | Cache and session storage | "Cache Integration" — docs/techdocs/docs/app/integrations/cache.md — retrieved 2025-01-09 |

**Note:** This repository currently uses Docker Compose for local development. Helm chart values will be documented here when Kubernetes deployment is implemented.

### Sources
- "Helm Chart Best Practices" — https://helm.sh/docs/chart_best_practices/ — retrieved 2025-01-09
- "Firefly III Helm Chart" — https://github.com/firefly-iii/helm-chart — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://helm.sh/docs/chart_best_practices/":"sha256:pending","https://github.com/firefly-iii/helm-chart":"sha256:pending"},"sections":{"helm-values":"sha256:pending"}}}
-->