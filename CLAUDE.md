# CLAUDE.md

Guidance for Claude Code working in this repository. Read this first in every session.

## Project

Invoice AI is an invoice and expense assistant for small businesses. A user uploads a photo or PDF of an invoice, AI extracts structured data (counterparty, tax ID / NIP, amounts, VAT, line items) with per-field confidence, the document is categorized, and the user verifies it on a review screen. On top of that: a document list with statuses and a spending dashboard.

**Version 1.0** (4 weeks, target 2026-12-06): one company and one role per account, Polish UI only. **Later versions (1.1+)**: i18n, CSV export, companies with roles and invitations, duplicate detection, a read-only text-to-SQL data chat, KSeF, an optional Python AI service. See `docs/ROADMAP.md`. Do not build 1.1+ features during 1.0 unless the roadmap says so.

## Sources of truth

Read before planning any change:

- `docs/ROADMAP.md`: stages, current stage, task list with `[SAM]` / `[CC]` tags.
- `docs/ARCHITECTURE.md`: system shape, modules, data flow.
- `docs/WORKFLOW.md`: how a task goes from spec to commit.
- `docs/specs/`: one lightweight spec per feature. Implement against the spec.
- `docs/decisions/`: ADRs. Do not contradict an accepted ADR; propose a new one instead.

If the code and the docs disagree, stop and ask which one is right.

## Task modes

Every task in the roadmap is tagged:

- **[SAM]**: the developer writes the code. Do NOT write the solution. Explain concepts, point to docs, ask guiding questions, review what was written. Show at most small snippets that illustrate an API, not the implementation. [SAM] is reserved for the four showcase areas the developer defends in interviews: the review screen (preview + React Hook Form + Zod), the TanStack Query data fetching pattern, the AI extraction pipeline with confidence scores, and the core API with authorization (endpoints, validation, error handling).
- **[CC]**: everything else (config, boilerplate, styling, simple components, remaining logic, extra tests). Claude Code writes the code, and the developer must understand every line: keep diffs small, explain non-obvious choices, list what to check during review. Every [CC] task ends with `/grill-me`.

If a task has no tag, ask before writing code.

## Language

- Everything in the repo is in English: code, identifiers, comments, commit messages, PRs, docs.
- App UI is Polish. In 1.0 there is no i18n library; i18n with an English switch comes in 1.1. Keep UI copy in the components readable and consistent so it is easy to extract later.
- Conversations with the developer may be in Polish; that does not change the rule above.

## Monorepo layout

```
apps/
  web/        Next.js (App Router, React, TypeScript)
  api/        Node.js + Express (TypeScript)
packages/
  shared/     Zod schemas, shared types, constants
docs/
```

Inside each app, organize code by feature (`features/invoices/`, `features/auth/`, ...), not by technical layer.

## Stack

TypeScript everywhere. PostgreSQL + Prisma (+ pgvector from 1.4), Zod (schemas live in `packages/shared` and are reused by web and api), TanStack Query, React Hook Form, Tailwind, Redis + BullMQ for background jobs, S3 with presigned URLs for files, Docker, GitHub Actions, Vitest, React Testing Library, Playwright. Deployed to AWS.

## Conventions

- Validate every input at the boundary with Zod (HTTP body, query, env vars, AI output).
- The API is the only component that talks to the database. The web app talks to the API.
- AI extraction output must be validated against a Zod schema; never trust raw model output.
- Every query that touches tenant data filters by `companyId`, even in 1.0 where each account has one company.
- (1.5) The text-to-SQL chat will run with a read-only database role scoped to the user's company. Never relax this.
- Money is stored as integer minor units (grosze) plus a currency code, never as floats.
- No secrets in the repo. Use `.env` files ignored by git and document variables in `.env.example`.

## Commands

To be filled in as the tooling lands (install, dev, test, lint, typecheck, e2e, db migrate).

## Before you finish a task

1. Tests for the change pass; add tests for new behavior.
   If the task is [CC], run `/grill-me` before closing it.
2. Lint and typecheck pass.
3. Update `docs/ROADMAP.md` (task status) and add an ADR in `docs/decisions/` if a decision was made.
4. Commit using Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`, `test:`, `refactor:`).
