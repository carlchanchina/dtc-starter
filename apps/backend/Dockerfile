# syntax=docker/dockerfile:1
#
# Medusa backend image (Medusa DTC starter / Turborepo + pnpm monorepo)
#
# Builds the Medusa server with `medusa build`, then produces a self-contained
# production deploy with `pnpm deploy --prod`, and runs `medusa start`.
#
# Build with the repository root as the build context:
#   docker build -f apps/backend/Dockerfile -t dtc-backend .

# ==========================================================
# base — shared image with Node, pnpm, and native build tools
# ==========================================================
FROM node:22-slim AS base

# pnpm comes from the repo's `packageManager` field (pnpm@10.11.1).
# Pin it GLOBALLY with `corepack prepare --activate` so every stage uses 10.11.1
# regardless of cwd's packageManager field. Without this, the `builder` stage
# (WORKDIR /app/apps/backend, which has no packageManager field) falls back to
# the Node 22 distro default pnpm (11.x) and `pnpm exec medusa build` fails
# with a deps-status-check error (runDepsStatusCheck -> sync2).
RUN corepack enable && corepack prepare pnpm@10.11.1 --activate

# Native-dependency build tooling (sharp, swc, msgpackr-extract, etc.)
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        python3 make g++ openssl ca-certificates \
    && rm -rf /var/lib/apt/lists/*

ENV PNPM_HOME=/pnpm \
    PATH=/pnpm:$PATH \
    NODE_ENV=development

WORKDIR /app

# Cache the pnpm store / download between builds
RUN pnpm config set store-dir /pnpm-store

# ==========================================================
# deps — install the full workspace (frozen lockfile)
# ==========================================================
FROM base AS deps
# Workspace manifests — every importer referenced by the lockfile must be
# present for `--frozen-lockfile` and later `pnpm deploy` to resolve.
COPY pnpm-workspace.yaml pnpm-lock.yaml package.json ./
COPY apps/backend/package.json  apps/backend/package.json
COPY apps/storefront/package.json apps/storefront/package.json

RUN pnpm install --frozen-lockfile

# ==========================================================
# builder — compile the Medusa server
# ==========================================================
FROM base AS builder
COPY --from=deps /app/node_modules ./node_modules
COPY --from=deps /app/apps/backend/node_modules ./apps/backend/node_modules

# Backend source only (it has no local workspace deps)
COPY apps/backend ./apps/backend

# Compile the Medusa server (creates apps/backend/.medusa/server)
WORKDIR /app/apps/backend
RUN pnpm exec medusa build

# Produce a clean, self-contained production package via pnpm deploy.
# `pnpm deploy` resolves workspace links and keeps only production deps.
WORKDIR /app
RUN pnpm --filter @dtc/backend deploy --prod /app/deployed \
    && rm -rf /app/deployed/.medusa \
    && cp -r apps/backend/.medusa /app/deployed/.medusa

# ==========================================================
# runner — minimal production image
# ==========================================================
FROM node:22-slim AS runner
RUN corepack enable && corepack prepare pnpm@10.11.1 --activate
ENV NODE_ENV=production \
    PORT=9000 \
    PATH=/app/node_modules/.bin:$PATH

WORKDIR /app

# Non-root user
RUN groupadd --system medusa && useradd --system --gid medusa medusa
COPY --from=builder --chown=medusa:medusa /app/deployed ./
USER medusa

EXPOSE 9000

# healthcheck hits Medusa's health endpoint (HOME workaround for non-root
# medusa-* packages that write to the home dir)
ENV HOME=/tmp
HEALTHCHECK --interval=30s --timeout=5s --start-period=40s --retries=3 \
    CMD node -e "fetch('http://127.0.0.1:9000/health').then(r=>{if(!r.ok)process.exit(1)}).catch(()=>process.exit(1))"

# Migrations are run separately (init container / release step); start only.
CMD ["medusa", "start"]
