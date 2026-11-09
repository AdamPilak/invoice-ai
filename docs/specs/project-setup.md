# Project setup

Status: draft
Roadmap: 1.0 / week 1

## Goal

A pnpm monorepo with `apps/web`, `apps/api` and `packages/shared` that installs, lints, typechecks, tests and runs locally with one command each, with local Postgres (pgvector), Redis and S3-compatible storage in Docker Compose, and CI that runs the same checks on every pull request.

This spec covers the first five week 1 roadmap items:

1. Monorepo setup (pnpm workspaces), shared `tsconfig`, ESLint, Prettier
2. Docker Compose: Postgres with pgvector, Redis, MinIO (local S3)
3. GitHub Actions: lint, typecheck, test on every PR
4. `apps/web`: Next.js App Router skeleton with Tailwind
5. `packages/shared` setup and env validation with Zod

## User stories

- As a developer, I want to clone the repo and run `pnpm install`, `docker compose up -d` and `pnpm dev` so that the whole stack runs locally in a few minutes.
- As a developer, I want one set of lint, format and TypeScript rules for every package so that code looks and behaves the same everywhere.
- As a developer, I want CI to block a PR that fails lint, typecheck or tests so that `main` always builds.
- As a developer, I want the apps to fail at startup with a clear message when an environment variable is missing or invalid, instead of failing later at runtime.

## Acceptance criteria

### Monorepo and tooling

- [ ] `pnpm-workspace.yaml` includes `apps/*` and `packages/*`. Packages are named `@invoice-ai/web`, `@invoice-ai/api`, `@invoice-ai/shared`.
- [ ] Node version pinned in `.nvmrc` and `engines`; pnpm version pinned with the `packageManager` field (Corepack).
- [ ] Root `tsconfig.base.json` with `strict: true`, `noUncheckedIndexedAccess: true`, ESM settings; every package extends it.
- [ ] ESLint flat config (`eslint.config.mjs`) at the root with `typescript-eslint` (type-aware rules) and the Next.js rules applied to `apps/web` only. ESLint does not do formatting (`eslint-config-prettier`).
- [ ] Prettier config at the root, `.prettierignore` excludes build output and lockfile.
- [ ] `.editorconfig`, `.gitignore` (node_modules, build output, `.env*` except `.env.example`, coverage, `.next`).
- [ ] Root scripts that run across all packages: `dev`, `build`, `lint`, `typecheck`, `test`, `format`, `format:check`.
- [ ] Every script passes on a clean clone after `pnpm install`.

### Packages

- [ ] `packages/shared`: TypeScript source package exported via `exports` (no build step), with Vitest and one smoke test. Contains a placeholder `src/index.ts` only; real schemas arrive with their features.
- [ ] `apps/api`: Express + TypeScript skeleton. `pnpm --filter @invoice-ai/api dev` starts the server with `tsx watch`; `build` bundles it with `tsup` (including `@invoice-ai/shared`). One route, `GET /health` returning `{ "status": "ok" }`, and one Vitest + Supertest test for it. No middleware, error format or feature folders yet; those belong to the "Express app structure" task.
- [ ] `apps/web`: Next.js App Router with TypeScript, `src/` directory and Tailwind. A placeholder home page in Polish. `transpilePackages` includes `@invoice-ai/shared`. Vitest + React Testing Library configured, with one smoke test of the home page.
- [ ] `apps/web` imports something from `@invoice-ai/shared` and `apps/api` does too, to prove the workspace link works in dev, build and test.

### Environment validation

- [ ] `apps/api/src/lib/config.ts` parses `process.env` with a Zod schema once at startup and exports a typed `config` object. Invalid or missing variables stop the process with a readable list of problems.
- [ ] `apps/web` validates its variables the same way (server-side variables and `NEXT_PUBLIC_*` separately), failing the build or startup on invalid values.
- [ ] Code reads configuration only through the validated object, never through `process.env` directly (enforced by an ESLint `no-restricted-properties` rule outside the config files).
- [ ] `.env.example` in each app documents every variable with a safe local default. Real `.env` files are git-ignored.
- [ ] Initial variables: api `NODE_ENV`, `PORT`, `DATABASE_URL`, `REDIS_URL`, `S3_ENDPOINT`, `S3_REGION`, `S3_BUCKET`, `S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY`, `WEB_ORIGIN`; web `NEXT_PUBLIC_API_URL`. Variables not used yet by code are still validated so the config is ready for the next tasks.

### Docker Compose

- [ ] `docker-compose.yml` at the root with:
  - Postgres with pgvector (`pgvector/pgvector` image, pinned major version), named volume, healthcheck.
  - Redis (pinned major version), healthcheck.
  - MinIO (pinned image tag) with the console exposed, plus a one-shot init container that creates the documents bucket.
- [ ] Ports and credentials match the defaults in `.env.example`.
- [ ] `docker compose up -d` on a clean machine results in all services healthy and the bucket present.
- [ ] Apps run on the host (not in Compose) during development. Production Dockerfiles come in week 4.

### CI

- [ ] `.github/workflows/ci.yml` runs on `pull_request` and on `push` to `main`.
- [ ] Steps: checkout, pnpm setup, Node from `.nvmrc` with pnpm cache, `pnpm install --frozen-lockfile`, `format:check`, `lint`, `typecheck`, `test`, `build`.
- [ ] CI does not need Docker services yet (no test touches the database in this task).
- [ ] A PR with a lint, type or test error fails the check.

### Docs

- [ ] `CLAUDE.md` "Commands" section lists the real commands.
- [ ] `README.md` gets a short "Run locally" section (prerequisites, install, compose, dev).
- [ ] `docs/ROADMAP.md` items 1 to 5 of week 1 are marked done.

## Out of scope

- Prisma schema, migrations and the database client (next task).
- Express middleware, error format, validation middleware, feature folders (separate task).
- Auth, BullMQ queue and worker process, S3 client code.
- Playwright and end-to-end tests (week 4).
- Production Dockerfiles and deployment (week 4).
- Pre-commit hooks (Husky, lint-staged). CI is the gate; hooks can be added later if needed.
- i18n setup (1.1).

## Technical choices

| Choice | Picked | Alternative and trade-off |
| --- | --- | --- |
| Task runner | Plain `pnpm -r` / `pnpm --filter` scripts | Turborepo adds caching and task graphs, but with three packages the speed gain is small and it is one more tool to configure and explain. Easy to add later. |
| Consuming `packages/shared` | Source-only internal package (TypeScript exported directly, compiled by each consumer) | A compiled package with its own `build` step and project references is closer to a publishable library, but needs a watch process in dev and build ordering in CI. Nothing here is published. |
| API dev and build | `tsx watch` in dev, `tsup` bundle for production | Plain `tsc` output keeps file structure but cannot inline the source-only shared package and needs `.js` extensions in imports under `NodeNext`. |
| Module system | ESM everywhere, `moduleResolution: "Bundler"` | CommonJS is still the Express default, but ESM is the standard going forward and Next.js already uses it. |
| Lint and format | ESLint for code quality, Prettier for formatting | Biome is faster and does both, but has fewer type-aware rules and no official Next.js plugin. |
| Env validation | Hand-written Zod schema per app | `@t3-oss/env-nextjs` solves the same problem with extra helpers, but it is another dependency and hides a small amount of code that is worth understanding. |
| Local S3 | MinIO in Docker Compose | LocalStack emulates more AWS services but is heavier and needs an account for newer versions. See open questions. |

## API / data changes

- `GET /health` on the API: `200 { "status": "ok" }`. Used by local checks now and by container health checks in week 4.
- No database schema yet.

## Edge cases

- Running scripts with the wrong Node or pnpm version: `engines` plus `engine-strict` in `.npmrc` stops `pnpm install` with a clear message.
- Missing `.env`: the app exits on startup listing the missing variables, and the message points to `.env.example`.
- Port conflicts with services already running on the host (5432, 6379, 9000): ports are configurable through Compose variables with documented defaults.
- `NEXT_PUBLIC_*` variables are inlined at build time, so changing them requires a rebuild. Documented in `.env.example`.
- A package imports `@invoice-ai/shared` but the dependency is missing in its `package.json`: pnpm's strict linking makes this fail locally, not only in CI.

## Open questions

- MinIO stopped publishing official prebuilt community images in late 2025. Proposal: pin the last published MinIO image tag for local development only (production uses real S3). If that tag stops being pullable, switch to another S3-compatible server (for example SeaweedFS or RustFS) behind the same env variables.
- Node version: proposal is the current Active LTS line at setup time, the same version later used in the production images.
- Should `GET /health` also check database and Redis connectivity? Proposal: no for now; add a separate readiness check when the database client exists.
