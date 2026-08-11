# CONTEXT.md

Glossary for the dtc-starter deployment / CI design. Devoid of implementation
detail — a shared vocabulary only.

## Terms

- **Primary remote** — the git remote that is the source of truth and triggers
  CI. Currently `origin` → `github.com/carlchanchina/dtc-starter`.
- **Mirror remote** — a secondary git remote kept in sync for availability /
  backup; it does **not** trigger CI. Currently `cnb` →
  `cnb.cool/cowin3332/dtc-starter`.
- **Image registry** — where built container images are published. Currently
  GitHub Container Registry (GHCR): `ghcr.io/carlchanchina/dtc-starter/dtc-backend`.
- **Backend-only image** — the produced Docker image contains only the Medusa
  backend (`apps/backend`); the storefront is excluded and deployed separately.
- **pnpm version pin** — fixing the pnpm major version across every Docker build
  stage via `corepack prepare pnpm@10.11.1 --activate`, so the version matches
  the lockfile regardless of the working directory's `packageManager` field.

## Resolved decisions

- Topology (GitHub primary + CI, CNB code mirror): see
  `docs/adr/0001-github-primary-cnb-mirror.md`.

- pnpm version pin (build fix): see `docs/adr/0002-pnpm-version-pin.md`.
