# ADR 0003: Use `pnpm deploy --legacy` for the production image

- Status: accepted
- Date: 2026-08-11

## Context

After the pnpm-version pin (ADR 0002) fixed the `medusa build` step, the image
build still failed at `pnpm --filter @dtc/backend deploy --prod /app/deployed`
with:

    ERR_PNPM_DEPLOY_NONINJECTED_WORKSPACE
    By default, starting from pnpm v10, we only deploy from workspaces that
    have "inject-workspace-packages=true" set

pnpm v10 refuses `pnpm deploy` unless the workspace opts in to injected
dependencies. The repo's `.npmrc` does not set `inject-workspace-packages`, and
the Dockerfile does not copy `.npmrc` into the build context, so the deploy step
aborted (exit 1) under the pinned pnpm 10.11.1.

Notably, the CNB build "passed" on the same Dockerfile because its runner fell
back to pnpm 11 (where the restriction is enforced differently), producing a
non-reproducible image. Reproducibility between GitHub and CNB required forcing
the same behaviour under pnpm 10.11.1.

## Decision

Add the `--legacy` flag to the deploy command:

    RUN pnpm --filter @dtc/backend deploy --legacy --prod /app/deployed \

`--legacy` is the exact escape hatch pnpm suggests in the error message; it
restores the v9 deploy behaviour without changing the repo-wide `.npmrc` (which
would also affect local development).

Alternative considered — setting `inject-workspace-packages=true` in `.npmrc`:
rejected because it changes workspace linking for every local `pnpm install`,
not just the deploy, and the Dockerfile intentionally does not copy `.npmrc`
into the build.

## Consequences

- `pnpm deploy` succeeds under the pinned pnpm 10.11.1 on both GitHub Actions
  and CNB with identical behaviour.
- The Dockerfile remains self-contained (no dependency on repo `.npmrc`).
