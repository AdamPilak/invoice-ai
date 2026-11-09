# Architecture

This document describes the intended shape of the system. It is updated when the architecture changes; the reasons for changes live in `docs/decisions/`.

Sections marked with a version (for example **(1.5)**) describe features planned after 1.0. Version 1.0 has one company and one role (owner) per account and a Polish-only UI.

## Overview

```
 Browser
   │
   ▼
 apps/web (Next.js)  ──HTTP/JSON──▶  apps/api (Express)  ──▶  PostgreSQL (+ pgvector)
   │                                     │      │
   │ PUT file (presigned URL)            │      └──▶ Redis ──▶ BullMQ worker ──▶ AI model API
   ▼                                     ▼                          │
  S3  ◀──────────── presigned URLs ──────┘                          └──▶ PostgreSQL
```

- **apps/web**: Next.js App Router. UI and routing; Polish UI in 1.0, i18n with English from 1.1. Fetches data from the API with TanStack Query. Does not access the database directly.
- **apps/api**: Express in TypeScript. REST API, auth, authorization, business logic, Prisma access, enqueuing jobs, issuing presigned S3 URLs.
- **worker**: runs from the `apps/api` codebase as a separate process. Consumes BullMQ jobs (AI extraction, categorization, duplicate check).
- **packages/shared**: Zod schemas and TypeScript types shared by web and api (request/response contracts, extraction schema, enums).

See `docs/decisions/001-nextjs-plus-express.md` for why the frontend and backend are separate.

## Repository layout

```
apps/
  web/
    src/app/              routes (App Router)
    src/features/<name>/  components, hooks, api client per feature
    src/i18n/             locale files (pl, en), from 1.1
  api/
    src/features/<name>/  router, service, repository, schemas per feature
    src/worker/           BullMQ processors
    src/lib/              db, s3, queue, config
    prisma/               schema and migrations
packages/
  shared/src/             Zod schemas, types, constants
docs/
  ROADMAP.md  ARCHITECTURE.md  WORKFLOW.md
  specs/      decisions/
```

Code is grouped by feature, not by layer. A feature folder in the API typically has `*.router.ts` (HTTP), `*.service.ts` (business logic), `*.repository.ts` (Prisma), and `*.test.ts`.

## Main flows

### Upload and extraction

1. Web asks the API for a presigned upload URL (`POST /documents/upload-url`).
2. Web uploads the file straight to S3.
3. Web confirms the upload (`POST /documents`); the API creates a `Document` with status `UPLOADED` and enqueues an extraction job.
4. The worker downloads the file, calls the AI model with a structured output schema, validates the result with Zod, runs sanity checks, and stores fields with confidence scores. Status becomes `NEEDS_REVIEW` (or `FAILED`).
5. Categorization runs after extraction (simple rules in 1.0). Duplicate detection is added in 1.4.
6. The user corrects fields on the review screen and approves; status becomes `APPROVED`.

### Data chat (1.5)

1. The user asks a question.
2. The API sends the question plus a description of company-scoped views to the model, which returns SQL.
3. The SQL is validated (single `SELECT`, allowed views only, row limit) and executed with a read-only database role, scoped to the user's company, with a statement timeout.
4. The result is returned to the user, optionally summarized by the model.

## Data model (initial)

- `User`
- `Company`
- `Membership` (user, company, role: `OWNER` | `ACCOUNTANT`), from 1.3; in 1.0 a user belongs to one company as owner
- `Document` (company, S3 key, mime type, status, uploaded by, timestamps)
- `Extraction` (document, extracted fields, per-field confidence, model and version)
- `Invoice` (approved data: counterparty, NIP, issue and due dates, net, VAT, gross, currency, category)
- `InvoiceLine` (description, quantity, unit price, VAT rate, amounts)
- `Category`
- (1.4) Embeddings for duplicate detection stored with pgvector

Money is stored as integer minor units plus a currency code.

## Cross-cutting concerns

- **Multi-tenancy**: every tenant table has `companyId` and every query filters by it, already in 1.0. Authorization middleware resolves the user's company (and from 1.3, the active company and role).
- **Validation**: Zod at every boundary (HTTP input, env vars, AI output, CSV import later).
- **Security**: presigned URLs are short-lived and scoped to one key; secrets come from environment variables (Secrets Manager in production); (1.5) the data chat uses a read-only role.
- **i18n** (1.1): all UI strings come from locale files, Polish default, English switch.
- **Testing**: Vitest for units and API routes, React Testing Library for components, Playwright for critical end-to-end paths.
- **CI**: GitHub Actions runs lint, typecheck and tests on every PR.

## Deployment (planned)

AWS: containers for web, api and worker; RDS PostgreSQL with pgvector; ElastiCache Redis; S3 for documents. The exact compute service is chosen in ADR 004 in week 4 of 1.0.

## Possible future changes

- KSeF integration as a separate feature module with its own jobs.
- A Python FastAPI service for AI extraction if the Node implementation becomes a bottleneck or Python-only tooling is needed.
