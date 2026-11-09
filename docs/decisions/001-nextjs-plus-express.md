# 001. Next.js frontend with a separate Express backend

Date: 2026-11-07
Status: accepted

## Context

Invoice AI needs a web UI and a backend that handles auth, multi-tenant data access, file uploads via S3 presigned URLs, background jobs (AI extraction, categorization, duplicate detection) with BullMQ, and a read-only text-to-SQL chat. Next.js alone could serve both the UI and the API (Route Handlers, Server Actions).

The project is also a learning and portfolio project. It should demonstrate both modern React and a classic Node.js backend, which are common interview topics.

## Decision

Use a monorepo with two apps and a shared package:

- `apps/web`: Next.js (App Router, React, TypeScript) for the UI only.
- `apps/api`: Node.js + Express in TypeScript for the REST API and the background worker.
- `packages/shared`: Zod schemas and types used by both.

The web app talks to the API over HTTP. Only the API accesses the database.

## Alternatives considered

- **Next.js full stack (Route Handlers / Server Actions).** Less infrastructure and one deployment. But long-running and background work (queues, workers) does not fit the request model well, serverless limits are a risk for AI processing, and backend logic gets tied to the frontend framework.
- **NestJS instead of Express.** More structure out of the box, but more framework magic to learn at once. Express keeps the backend explicit, which suits the [SAM] tasks.
- **tRPC between web and api.** Strong end-to-end types, but couples the clients to TypeScript and hides HTTP details. Shared Zod schemas give most of the type safety while keeping a plain REST API.

## Consequences

- Clear separation: the API can be tested, scaled and deployed independently, and the worker shares its code.
- A Python FastAPI service for AI can be added later without touching the frontend.
- More moving parts: two deployments, CORS and auth across origins (or a reverse proxy), and API contracts to maintain. Shared Zod schemas in `packages/shared` reduce contract drift.
- Next.js server features (Server Components, Server Actions) are used for rendering only, not for business logic or database access.
