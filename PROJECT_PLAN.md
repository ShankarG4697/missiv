# Project plan

Assumptions this plan is built on (confirm/correct before Phase 1 starts):
- Early-stage scale, single-tenant, no org/team RBAC yet.
- Outbound email via a transactional ESP API (SES or SendGrid), not self-hosted SMTP.
- RabbitMQ + Celery for queueing and scheduling. Kafka is deferred, not adopted.
- Cards (wedding/birthday) reuse the campaign template engine but are a distinct send
  flow, built after core campaigns work.

## Phase 0 — Foundation (this branch)

- Monorepo scaffold (`apps/web`, `apps/api`, `apps/worker`, `packages/shared`)
- `docker-compose.yml` for local dev (postgres, redis, rabbitmq, web, api, worker)
- CI: lint, type-check, test, Docker build validation on every PR
- CD: build + push images to GHCR on merge to `main`, Trivy scan gates the push

**Definition of done**: `docker compose up` boots the full stack locally. A PR to
`main` triggers CI. A merge to `main` produces images visible in the repo's GHCR
package list.

## Phase 1 — Core campaign flow

- Auth (email/password or magic link)
- Contact and list CRUD
- Template CRUD (MJML or rich-text based, not a custom drag-and-drop builder)
- Campaign creation and immediate send via Celery → ESP

**Definition of done**: a list can be created, a template written, and a real
campaign sent end-to-end through the ESP sandbox/test mode. Unit and integration
tests cover the send path. No secrets committed — config is environment-based.

## Phase 2 — Scheduling & tracking

- `celery beat` scheduled sends
- Webhook ingestion for delivered/opened/clicked/bounced events
- Send status visible in the UI

**Definition of done**: a scheduled campaign fires at the correct time under a
fixed/mocked clock in tests. Webhook handling is idempotent — an ESP retry does not
double-count an event.

## Phase 3 — Cards

- Single-recipient personalized card flow (wedding, birthday, etc.), reusing the
  Phase 1 template engine

**Definition of done**: card sends are tested as a distinct flow from bulk campaigns,
and template reuse between the two is verified (no duplicated rendering logic).

## Phase 4 — Analytics

- Aggregate reporting on top of the `events` table

**Definition of done for introducing Kafka here**: a written comparison showing
Postgres aggregation is actually insufficient (query latency, consumer count, or
replay requirements) — not just "it would be nice to use Kafka."