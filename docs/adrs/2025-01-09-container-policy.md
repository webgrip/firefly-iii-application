---
title: "Containerization & Compose Policy"
description: "Org-owned images from pinned upstream bases; official app image preferred, source-build only if unsupported or unavailable."
tags: [operations, containerization, docker, compose, policy]
search: { boost: 4, exclude: false }
icon: material/steam
author: "<Infrastructure/Platform Team>"
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

**Security & Reproducibility**: Org-owned images provide:
- Complete Software Bill of Materials (SBOM) tracking
- Controlled base image updates and CVE patching
- Reproducible builds with pinned dependencies
- Custom security scanning and compliance validation

**Operational Control**: Source-building when needed allows:
- Custom features or patches not in upstream images
- Integration with organizational monitoring/logging
- Compliance with internal security policies

## Consequences

**Pros:**
- Enhanced security posture with full supply chain visibility
- Predictable updates and rollback capabilities
- Compliance with organizational container policies
- Consistent image layering and optimization

**Cons:**
- Additional maintenance overhead for base image updates
- Potential lag behind upstream releases
- Increased storage requirements for org registry

**Upgrade Cadence**: Base images updated monthly or when critical CVEs identified. Firefly III version updates follow upstream stable releases.

## Sources
- "Firefly III Installation Guide" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{
  "last_audit": "2025-01-09",
  "fingerprints": {
    "sources": {
      "https://docs.firefly-iii.org/how-to/firefly-iii/installation/": ""
    },
    "sections": {
      "policy": ""
    }
  }
}
-->