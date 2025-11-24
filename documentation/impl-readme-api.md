README Template — local-store-api (REST)

Purpose
This repository hosts the REST API services (NestJS) for sessions, payments, reconciliation, and admin endpoints.

Quick summary

- Tech: Node.js, NestJS, TypeScript
- Audience: Backend engineers, integration engineers
- Responsibility: Durable APIs, webhook handlers, reconciliation jobs

Suggested layout

- /src/
  - /controllers/
  - /services/
  - /dto/
  - /migrations/
- /tests/
- /contracts/ (OpenAPI copies or generated clients)

Onboarding

1. Install Node 18+, pnpm
2. Setup local Postgres + Redis (see orchestration docs)
3. pnpm install && pnpm start:dev

CI & Deploy

- CI: lint, unit tests, integration tests against ephemeral Postgres, contract tests
- CD: build docker image, push to registry, deploy via `local-store-infra`

Security & PCI

- Payment secrets must use org secret manager; never commit keys.
- Webhook verification required; see `security.md` in infra repo.

Contracts

- Accept canonical OpenAPI artifacts from `local-store-contracts`; bump contract version on breaking changes and follow contract-change-process.md.

Ownership & Contacts

- CODEOWNERS: backend lead
- Contacts: backend@ourplatform

Notes

- Enforce tenant scoping and RLS where applicable.
- Add metrics: session_created, event_append_count, payment_match_rate.
