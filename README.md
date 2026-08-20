# Missiv

Mobile-responsive email campaign + card-sending platform. Next.js frontend, Python
(FastAPI + Celery) backend, monorepo, Docker for local dev, GitHub Actions for CI/CD.

See [PROJECT_PLAN.md](./PROJECT_PLAN.md) for phases and definition-of-done per phase.

## Structure

```
apps/
  web/      Next.js (TypeScript, Tailwind)
  api/      FastAPI backend
  worker/   Celery workers + beat schedule
packages/
  shared/   Shared schemas/types between web and api
```

## Local development

```
cp .env.example .env   # fill in real values
docker compose up
```

- `web` → http://localhost:3000
- `api` → http://localhost:8000
- RabbitMQ management UI → http://localhost:15672

## Architecture

- **Broker/scheduler**: RabbitMQ + Celery (`celery beat` handles scheduled sends).
  Kafka is intentionally not used yet — see PROJECT_PLAN.md Phase 4 for when that's
  reconsidered.
- **Data**: Postgres. Campaigns, contacts, lists, sends, and delivery/open/click
  events all live there.
- **Email delivery**: outbound via a transactional ESP API (SES/SendGrid), not a
  self-hosted MTA.

## CI/CD

- `.github/workflows/ci.yml` — lint, type-check, test, Docker build validation on
  every PR to `main`.
- `.github/workflows/release.yml` — builds and pushes images to GHCR
  (`ghcr.io/<owner>/<repo>-{web,api,worker}`) on merge to `main` and on `v*.*.*` tags,
  with a Trivy vulnerability scan gating the push.