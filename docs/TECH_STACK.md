# MediQueue — Technology Stack

**Owner:** ChamathDilshanC · **Status:** proposed · **Updated:** 2026-09-28

This document defines the initial web-first stack. Mobile uses the same APIs later. Versions are pinned in lockfiles and reviewed before implementation; this document intentionally avoids unverified version numbers.

The current implemented API is a modular FastAPI deployment with identity, scheduling and queue routers. Its `/docs` reference is served as lightweight HTML/CSS/JavaScript and reads the live OpenAPI contract; `/swagger` retains interactive request execution. The Next.js application remains a separate planned frontend. Authentication now includes a Supabase REST facade plus database-backed branch memberships; see the backend README for migrations and authorization rollout requirements.

| Layer | Choice | Purpose |
| --- | --- | --- |
| Web | Next.js, React, TypeScript | Admin, reception, doctor, patient, TV and kiosk interfaces |
| UI | Tailwind CSS, shadcn/ui, Lucide | Accessible components and consistent design tokens |
| API state | TanStack Query | Fetch, cache and invalidate server data |
| Local UI state | Zustand | Small client-only state; never authoritative queue state |
| Forms | React Hook Form + Zod | Form state and typed validation |
| Mobile, later | React Native + Expo | Patient/staff app using shared HTTP and event contracts |
| Backend runtime | Python | Long-running API services and worker |
| HTTP framework | FastAPI | Typed routes, validation and OpenAPI generation |
| Database | Supabase PostgreSQL | Durable source of truth for appointments, visits, queues and audit |
| Data access | SQLAlchemy + Alembic migrations | Typed queries and controlled schema evolution |
| Authentication | Supabase Auth | Identity; backend verifies JWT and enforces tenant/branch roles |
| Realtime | Supabase Realtime Broadcast | Sanitized queue updates to authorized channels |
| Storage | Supabase Storage | Approved assets and documents, if needed; private by default |
| Background work | Python worker/job + PostgreSQL outbox | Notifications, retries and asynchronous projections outside request execution |
| Contract | OpenAPI + JSON Schema | Generated API client and versioned event payloads |
| Testing | pytest, Playwright, contract/integration tests | Queue concurrency, access isolation and core web flow |
| Monitoring | OpenTelemetry-compatible tracing, structured logs, Sentry | Diagnose errors and performance |
| Deployment | Vercel serverless Python functions for FastAPI; Vercel for Next.js | API and web deployments use pinned commits; workers run as separate managed jobs |
| CI/CD | GitHub Actions | Per-repo checks and main-repo integration deployment |

## Repository mapping

| Repository slug | Responsibilities |
| --- | --- |
| `mediqueue` | Integration, `architecture.md`, `TECH_STACK.md`, `AGENTS.md`, deployment and pinned submodules |
| `mediqueue-frontend` | Web app, UI package, generated API client and frontend tests |
| `mediqueue-backend` | Identity, scheduling, queue, config API, notification worker and owned migrations |
| `mediqueue-configuration` | Validated defaults, environment/tenant overrides and public config allowlist |

The three child repos are Git submodules in the main repo. Main CI checks the exact pinned commits. Backend services are independently deployable within the single backend repo. Vercel builds must initialize the pinned `configuration/` submodule before validating and packaging its snapshot. Read `architecture.md` for service ownership, queue transactions and config delivery.

## Data flow

1. Web/mobile authenticates with Supabase Auth and calls versioned FastAPI APIs.
2. Backend validates JWT, membership, role and tenant scope before changing PostgreSQL state in a transaction.
3. The outbox publishes sanitized events through Realtime Broadcast and queues notification jobs.
4. Clients fetch a fresh authoritative snapshot after reconnect or event gaps.
5. Vercel checks out the pinned configuration submodule during the build, validates it, packages an immutable release snapshot and serves it through the Config service. Only safe public settings reach the browser.

## Deliberately deferred

- **Redis:** add only after measured database or realtime bottlenecks. It must never own durable token state.
- **Kafka/message broker:** PostgreSQL outbox is sufficient for the initial scale; reassess on measured throughput and integration needs.
- **AI ETA:** start with explainable rolling-duration estimates and measured accuracy.
- **Direct browser writes to queue tables:** all queue transitions go through backend authorization and transactions.

## Hosting checklist

Deploy the FastAPI API to Vercel using its Python serverless-function deployment model and configure the build to initialize the pinned `configuration/` submodule. Run notification/outbox processing as a separate managed worker or scheduled job because Vercel functions are request-oriented and are not a durable worker host. Keep development, staging and production Supabase projects separate, configure secrets through Vercel environment variables, and add health checks, logs and alerts before a live hospital rollout.
