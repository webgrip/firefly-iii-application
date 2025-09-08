---
title: "Containerization & Compose Policy"
description: "Org-owned images from pinned upstream bases; official app image preferred, source-build only if unsupported or unavailable."
tags: [operations, containerization, docker, compose, policy]
search: { boost: 4, exclude: false }
icon: material/steam
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Record our image and Compose policies for this repo.

**Contents**
- [Policy](#policy)
- [Rationale](#rationale)
- [Consequences](#consequences)
- [Sources](#sources)

## Policy
- Use **org-owned images** derived from **pinned upstream bases** (no `latest`).
- Prefer **official upstream app image**; fall back to source-build only when upstream image is unavailable or missing required features.
- Compose uses **only our images**; tag pinning enforced.
- Healthchecks and ports match upstream guidance.

## Rationale
This policy ensures:
- **Security**: All images are built from known, verified sources with controlled supply chain
- **Reproducibility**: Pinned tags guarantee consistent builds across environments
- **Auditability**: Org-owned images provide complete Software Bill of Materials (SBOM)
- **Compliance**: Meets enterprise security requirements for container image governance

## Consequences
**Pros:**
- Complete control over security patches and CVE remediation
- Consistent behavior across development, staging, and production
- Ability to implement custom security hardening

**Cons:**
- Additional maintenance overhead for image builds and updates
- Longer initial setup time compared to using upstream images directly
- Need to monitor upstream for security updates and rebuild accordingly

**Upgrade cadence:** Monthly security updates, quarterly feature updates, immediate critical security patches.

## Sources
- "Firefly III Docker Documentation" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker — retrieved 2025-01-09
- "Official Firefly III Docker Images" — https://hub.docker.com/r/fireflyiii/core — retrieved 2025-01-09

<!-- ai-docs-metadata
{
  "last_audit": "2025-01-09",
  "fingerprints": {
    "sources": {
      "https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker": "sha256:pending",
      "https://hub.docker.com/r/fireflyiii/core": "sha256:pending"
    },
    "sections": {
      "policy": "sha256:pending"
    }
  }
}
-->