# ADR 0001: GitHub as primary CI, CNB as code mirror

- Status: accepted
- Date: 2026-08-09

## Context

The code already lives on GitHub (`carlchanchina/dtc-starter`) with a GitHub
Actions workflow (`.github/workflows/build-image.yml`) that builds the Medusa
backend image and pushes it to GHCR. The user additionally wants the repository
on CNB ("把这个文件夹也上传到 CNB，新建一个仓库").

GitHub Actions only triggers on a GitHub-hosted repository, and CNB git URLs are
not valid GitHub remotes. So "code on CNB" and "image built by GitHub Actions"
cannot both be the primary without duplication.

## Decision

- Keep **GitHub as the primary remote and the sole CI**: it builds the backend
  image and publishes it to GHCR.
- Add **CNB (the `cowin3332` group) as a mirror remote** for code availability /
  backup only. No second image build runs on CNB.
- The image remains **backend-only**.

## Consequences

- Code is duplicated across two platforms; only GitHub drives automation.
- The image lives only on GHCR. To also publish to the CNB registry later, add a
  CNB pipeline (`.cnb.yml`) — explicitly out of scope here.

## Trade-offs considered

- *CNB as primary + CNB pipeline builds the image* — rejected: more work, and it
  would create a second, diverging image source.
- *Both platforms build the image* — rejected: highest maintenance, two sources
  of truth for the artifact.
- *GitHub primary + CNB mirror* (chosen): one CI, one image, code still on CNB.
