# Roadmap

Status legend: `[ ]` todo, `[~]` in progress, `[x]` done.

**Target:** version 1.0 in 4 weeks (about 8 hours a day), by 2026-12-06.
**Current:** 1.0, week 1.

---

## Version 1.0 (4 weeks)

Scope: one company per account, one role (owner), Polish UI only. Data model keeps `companyId` on tenant tables from day one so companies and roles in 1.3 do not need a rewrite.

### Week 1: setup, database, sign-in (2026-11-09 to 2026-11-15)

Goal: a monorepo that builds, tests and runs locally; a user can sign up and sign in.

- [x] Spec 001: `docs/specs/001-project-setup.md`
- [ ] Monorepo setup (pnpm workspaces), shared `tsconfig`, ESLint, Prettier
- [ ] Docker Compose: Postgres with pgvector, Redis, MinIO (local S3)
- [ ] GitHub Actions: lint, typecheck, test on every PR
- [ ] `apps/web`: Next.js App Router skeleton with Tailwind
- [ ] `packages/shared` setup and env validation with Zod
- [ ] Spec 002: `docs/specs/002-auth.md`
- [ ] ADR 002: auth approach (session cookies vs JWT, library)
- [ ] Prisma schema and first migration: User, Company, Document, Extraction, Invoice, InvoiceLine, Category
- [ ] Express app structure: error-handling middleware, Zod validation middleware, consistent error format
- [ ] Auth endpoints (sign up, sign in, sign out, me) and auth middleware that resolves the user and their company
- [ ] Web: sign-in and sign-up pages, protected layout
- [ ] Tests for auth endpoints

### Week 2: upload, queue, AI extraction (2026-11-16 to 2026-11-22)

Goal: an uploaded photo or PDF lands in S3, gets processed in the background, and ends up with structured data and per-field confidence.

- [ ] Spec 003: `docs/specs/003-document-upload.md`
- [ ] Spec 004: `docs/specs/004-ai-extraction.md`
- [ ] ADR 003: model provider and structured output approach
- [ ] Document endpoints: presigned upload URL, confirm upload, get document; file type and size limits
- [ ] Document status machine (`UPLOADED`, `PROCESSING`, `NEEDS_REVIEW`, `APPROVED`, `FAILED`)
- [ ] Web: upload with drag and drop and progress
- [ ] BullMQ queue and worker process, retries, failure handling
- [ ] Extraction schema in `packages/shared` (counterparty, NIP, dates, net/VAT/gross, line items, confidence per field)
- [ ] Extraction service: prompt, structured output call, Zod validation, storing the result
- [ ] Sanity checks that lower confidence (net + VAT = gross, NIP checksum, sum of lines)
- [ ] Simple rule-based categorization
- [ ] Tests: schema validation, worker with a mocked model, a small set of sample invoices

### Week 3: review screen, document list, dashboard (2026-11-23 to 2026-11-29)

Goal: the user reviews and approves extracted data, browses documents by status, and sees a simple spending overview.

- [ ] Spec 005: `docs/specs/005-review-screen.md`
- [ ] Spec 006: `docs/specs/006-documents-and-dashboard.md`
- [ ] TanStack Query setup and the data fetching pattern: query keys, custom hooks, loading and error states, mutations with invalidation
- [ ] Review screen: document preview (image and PDF) next to the form
- [ ] Review form with React Hook Form and the shared Zod schema, low-confidence fields highlighted, approve and reject
- [ ] Endpoints for updating and approving an invoice
- [ ] Document list endpoint with filtering by status, sorting and pagination
- [ ] Web: document list with statuses (uses the shared fetching pattern)
- [ ] Dashboard endpoint: spending by month and by category
- [ ] Web: dashboard with simple charts
- [ ] RTL tests for the review form

### Week 4: tests, deploy, README, demo (2026-11-30 to 2026-12-06)

Goal: 1.0 is live on AWS, the critical paths are covered by tests, and the project is ready to show to recruiters.

- [ ] Playwright: sign up → upload → review → approve → appears in the list and dashboard
- [ ] Fill gaps in unit and integration tests for the core areas (see `CLAUDE.md`)
- [ ] ADR 004: AWS target (App Runner vs ECS Fargate), RDS, ElastiCache, S3, Secrets Manager
- [ ] Production Dockerfiles for web, api and worker
- [ ] Infrastructure and deploy pipeline in GitHub Actions
- [ ] Logging (CloudWatch), rate limiting on auth and upload
- [ ] Seed data for the demo account
- [ ] README: what it is, architecture diagram, tech choices with links to ADRs, how to run locally, screenshots
- [ ] Demo recording (2 to 3 minutes)
- [ ] Release: tag `v1.0.0`

---

## Version 1.1+

Added while job hunting, roughly in this order. Each gets a numbered spec and, where needed, an ADR before work starts.

### 1.1: i18n
- Polish default with an English switch; move all UI strings to locale files.

### 1.2: CSV export
- Export approved invoices, Polish number and date formats, streaming.

### 1.3: Companies, roles and invitations
- Multiple companies per user, roles (owner, accountant), invitations, role-based access control.

### 1.4: Duplicate detection
- Exact match on key fields plus pgvector similarity; warning on the review screen.

### 1.5: Data chat (text-to-SQL, read-only)
- Read-only database role, company-scoped views, SQL validation, timeouts and row limits.

### Later
- KSeF integration (import invoices from the national e-invoicing system).
- Optional Python AI microservice (FastAPI) if extraction outgrows the Node API.
