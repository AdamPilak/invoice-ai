# Roadmap

Status legend: `[ ]` todo, `[~]` in progress, `[x]` done.
Task modes: **[SAM]** the developer writes it, **[CC]** Claude Code writes it (see `CLAUDE.md`). Every [CC] task ends with `/grill-me`.

**Target:** version 1.0 in 4 weeks (about 8 hours a day), by 2026-12-06.
**Current:** 1.0, week 1.

"Related theory" points to modules of the Interview Prep JS course. It is background for interview prep, not a prerequisite: the project does not wait for the course. When a topic shows up before its module, it is explained briefly during the spec.

## [SAM] showcase areas

These are the parts the developer writes and defends in interviews. Everything else is [CC].

1. **Review screen**: document preview + form built with React Hook Form and Zod.
2. **Data fetching pattern** with TanStack Query, including loading and error states.
3. **AI extraction pipeline** with confidence scores: prompts, structured output, validation.
4. **Core API with authorization**: endpoints, validation, error handling.

---

## Version 1.0 (4 weeks)

Scope: one company per account, one role (owner), Polish UI only. Data model keeps `companyId` on tenant tables from day one so companies and roles in 1.1 do not need a rewrite.

### Week 1: setup, database, sign-in (2026-11-09 to 2026-11-15)

Goal: a monorepo that builds, tests and runs locally; a user can sign up and sign in.

Related theory: 3 TypeScript, 7 Node.js and Express, 8 PostgreSQL and Prisma, 9 Security and auth, 11 Docker and CI/CD.

- [ ] [CC] Monorepo setup (pnpm workspaces), shared `tsconfig`, ESLint, Prettier
- [ ] [CC] Docker Compose: Postgres with pgvector, Redis, MinIO (local S3)
- [ ] [CC] GitHub Actions: lint, typecheck, test on every PR
- [ ] [CC] `apps/web`: Next.js App Router skeleton with Tailwind
- [ ] [CC] `packages/shared` setup and env validation with Zod
- [ ] Spec: `docs/specs/auth.md`
- [ ] ADR 002: auth approach (session cookies vs JWT, library)
- [ ] [CC] Prisma schema and first migration: User, Company, Document, Extraction, Invoice, InvoiceLine, Category
- [ ] [SAM] Express app structure: error-handling middleware, Zod validation middleware, consistent error format
- [ ] [SAM] Auth endpoints (sign up, sign in, sign out, me) and auth middleware that resolves the user and their company
- [ ] [CC] Web: sign-in and sign-up pages, protected layout
- [ ] [CC] Tests for auth endpoints

### Week 2: upload, queue, AI extraction (2026-11-16 to 2026-11-22)

Goal: an uploaded photo or PDF lands in S3, gets processed in the background, and ends up with structured data and per-field confidence.

Related theory: 2 Async and event loop, 3 TypeScript (Zod), 7 Node.js and Express, 12 AWS (S3, presigned URLs).

- [ ] Spec: `docs/specs/document-upload.md`
- [ ] Spec: `docs/specs/ai-extraction.md`
- [ ] ADR 003: model provider and structured output approach
- [ ] [SAM] Document endpoints: presigned upload URL, confirm upload, get document; file type and size limits
- [ ] [CC] Document status machine (`UPLOADED`, `PROCESSING`, `NEEDS_REVIEW`, `APPROVED`, `FAILED`)
- [ ] [CC] Web: upload with drag and drop and progress
- [ ] [CC] BullMQ queue and worker process, retries, failure handling
- [ ] [SAM] Extraction schema in `packages/shared` (counterparty, NIP, dates, net/VAT/gross, line items, confidence per field)
- [ ] [SAM] Extraction service: prompt, structured output call, Zod validation, storing the result
- [ ] [SAM] Sanity checks that lower confidence (net + VAT = gross, NIP checksum, sum of lines)
- [ ] [CC] Simple rule-based categorization
- [ ] [CC] Tests: schema validation, worker with a mocked model, a small set of sample invoices

### Week 3: review screen, document list, dashboard (2026-11-23 to 2026-11-29)

Goal: the user reviews and approves extracted data, browses documents by status, and sees a simple spending overview.

Related theory: 4 React fundamentals, 5 React performance and state (TanStack Query), 6 Next.js App Router, 8 PostgreSQL (aggregations, indexes).

- [ ] Spec: `docs/specs/review-screen.md`
- [ ] Spec: `docs/specs/documents-and-dashboard.md`
- [ ] [SAM] TanStack Query setup and the data fetching pattern: query keys, custom hooks, loading and error states, mutations with invalidation
- [ ] [SAM] Review screen: document preview (image and PDF) next to the form
- [ ] [SAM] Review form with React Hook Form and the shared Zod schema, low-confidence fields highlighted, approve and reject
- [ ] [SAM] Endpoints for updating and approving an invoice
- [ ] [CC] Document list endpoint with filtering by status, sorting and pagination
- [ ] [CC] Web: document list with statuses (uses the [SAM] fetching pattern)
- [ ] [CC] Dashboard endpoint: spending by month and by category
- [ ] [CC] Web: dashboard with simple charts
- [ ] [CC] RTL tests for the review form

### Week 4: tests, deploy, README, demo (2026-11-30 to 2026-12-06)

Goal: 1.0 is live on AWS, the critical paths are covered by tests, and the project is ready to show to recruiters.

Related theory: 10 Testing, 11 Docker, CI/CD and architecture, 12 AWS, 13 Final (AI workflow, presenting the project).

- [ ] [CC] Playwright: sign up → upload → review → approve → appears in the list and dashboard
- [ ] [CC] Fill gaps in unit and integration tests for the [SAM] areas
- [ ] ADR 004: AWS target (App Runner vs ECS Fargate), RDS, ElastiCache, S3, Secrets Manager
- [ ] [CC] Production Dockerfiles for web, api and worker
- [ ] [CC] Infrastructure and deploy pipeline in GitHub Actions
- [ ] [CC] Logging (CloudWatch), rate limiting on auth and upload
- [ ] [CC] Seed data for the demo account
- [ ] [CC] README: what it is, architecture diagram, tech choices with links to ADRs, how to run locally, screenshots
- [ ] [SAM] Demo recording (2 to 3 minutes)
- [ ] Release: tag `v1.0.0`

---

## Version 1.1+

Added while job hunting, roughly in this order. Each gets a spec and, where needed, an ADR before work starts.

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
