# firefly-iii-application

## Badges

[![Template Sync](https://img.shields.io/github/actions/workflow/status/webgrip/application-template/sync-template-files.yml?label=template%20sync&style=flat-square)](https://github.com/webgrip/application-template/actions/workflows/sync-template-files.yml)
[![License](https://img.shields.io/github/license/webgrip/application-template?style=flat-square)](LICENSE)
[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-orange.svg?style=flat-square)](https://www.conventionalcommits.org)
[![SemVer](https://img.shields.io/badge/semver-2.0.0-blue?style=flat-square)](https://semver.org)
[![Dockerized](https://img.shields.io/badge/containerized-docker-2496ED?logo=docker&logoColor=white&style=flat-square)](https://www.docker.com/)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square)](https://github.com/webgrip/application-template/issues)

> Self-hosted Firefly III personal finance management application with automated deployment, security, and operational workflows.

---

## At a Glance

| Aspect | What You Get |
| ------ | ------------- |
| Application | Firefly III personal finance management |
| Containerization | Multi-service Docker setup with org-owned images |
| CI/CD  | GitHub Actions pipelines & template sync |
| Consistency | Automatic sync of core config & workflow files to app repos (opt‑in via topic) |
| Quality | Conventional Commits + Semantic Versioning scaffolding |
| Documentation | TechDocs-ready structure for internal/platform portals |
| Security | Encrypted secrets handling via SOPS + age |
| Developer UX | Pre-configured editor & workflow automation |

## Features

- **Self-maintained image policy**: All services use org-owned images built from official upstream sources
- **Firefly III**: Complete personal finance management application stack
- **Security-first**: Official upstream images with exact version pinning
- **Port hygiene**: Only reverse proxy exposes host ports
- **Automated template file synchronization** (opt‑in per repo by GitHub topic)
- **Semantic release readiness** (`.releaserc.json` included)
- **Encrypted secrets workflow** (age / SOPS)
- **Curated GitHub workflow set** (docs changes, source changes)
- **Opinionated baseline configs**: EditorConfig, VSCode settings, .gitignore
- **Structured test categories** (unit, integration, functional, contract, e2e, smoke, performance, behavioral)

## Description

Self-hosted Firefly III personal finance management application with comprehensive containerization following WebGrip's self-maintained image policy. Every service uses org-owned images built from pinned official upstream sources for enhanced security and supply chain control.

**Containerization Architecture:**
- **Application**: Firefly III core application (`fireflyiii/core:version-6.1.21`)
- **Data Importer**: Firefly III data importer (`fireflyiii/data-importer:version-1.5.4`)
- **Reverse Proxy**: Nginx (`nginx:1.26.2-alpine3.20`)
- **Database**: MariaDB (`mariadb:12.0.2-noble`)
- **Cache**: Redis (`redis:8.2.1-alpine3.20`)

All images are rebuilt and tagged as `webgrip/firefly-iii-application.<service>:latest` with minimal overlays for project-specific configuration.

## Template Synchronization

This repository serves as a template that can automatically sync certain files to application repositories. To enable template sync for your application repository, add the `application` topic to your repository.

**Synced Files Include:**

| Category | Files |
| -------- | ----- |
| Workflows | `.github/workflows/*.yml` (selected core automation) |
| Config | `.editorconfig`, `.gitignore`, `.releaserc.json` |
| Dev UX | `.vscode/settings.json` |

These represent the "source of truth"; local divergent changes in target repos will be overwritten (review PRs carefully).

For detailed information, see the [Template Sync Documentation](docs/techdocs/template-sync.md).


## Getting Started

### Encrypted secrets

```bash
make init-encrypt
```

Creates:

- `age.agekey` → add to repo secret `SOPS_AGE_KEY`
- `age.pubkey` → used for encryption

Add plaintext secrets to:

```bash
ops/secrets/firefly-iii-application-secrets/values.dec.yaml
```

Encrypt them:

```bash
make encrypt-secrets SECRETS_DIR=./ops/secrets/firefly-iii-application-secrets
```

This produces `values.sops.yaml` (commit this).

---

### Docker

Start the full Firefly III stack:

```bash
make start
```

Stop and clean up:

```bash
make stop
```

The application will be available at http://localhost:8080

**Services:**
- **Nginx**: Reverse proxy exposing the application (port 8080)
- **Firefly III Application**: Personal finance management application
- **Firefly III Data Importer**: Data import and synchronization service
- **MariaDB**: Database backend
- **Redis**: Cache and session storage

**Port Policy:**
- Only the nginx reverse proxy exposes host ports (8080)
- Internal application services communicate via Docker network
- Database and Redis are only accessible within the container network

---

## Contributing

Contributions welcome! Please:

1. Open an issue describing the change
2. Use Conventional Commits for branch + commit messages
3. Add / adjust tests where behavior changes
4. Update docs (README / TechDocs / ADRs) when altering architecture

## Roadmap (Excerpt)

- [ ] Add coverage reporting & badge
- [ ] Introduce example service code scaffolding
- [ ] Provide k6 performance test harness
- [ ] Optional Terraform module integration

## License

Distributed under the terms of the MIT license. See `LICENSE` for details.
