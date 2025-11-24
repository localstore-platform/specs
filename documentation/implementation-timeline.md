Implementation Timeline — QR-Session MVP (Engineering Plan)

Purpose

A concrete 8-week engineering timeline broken into epics, owners (roles), and acceptance criteria so the dev team can implement the QR-session MVP (POST writes + cursor GET), reconciliation, and pilot readiness.

Assumptions

- Repo is documentation-first. Implementation will follow these specs.
- Stack: NestJS backend, Postgres, Redis, Next.js frontend (or Flutter mobile), S3 for cold storage. (Consistent with repo docs.)
- Pilot tenants and merchant materials prepared separately.

Roles (suggested)

- PM: Product Manager (0.2 FTE during implementation)
- Tech Lead / Backend Eng (1.0 FTE)
- Integration Eng / Backend (1.0 FTE)
- Frontend Eng (0.6 FTE)
- Data Engineer (0.5 FTE)
- QA Engineer (0.4 FTE)
- DevOps / SRE (0.3 FTE)
- Ops / Merchant Success (0.2 FTE during onboarding)

Overview: 8-week Plan (week numbers relative to start)

Week 0 — Kickoff & Design (Prep)

- Epics:
  - Review and freeze API contracts (`architecture/api-specification.md`, reconciliation API).
  - Finalize DB schema and retention rules (`architecture/qr-session-database-schema.md`).
  - Prepare staging environment and webhook testing harnesses for VN payment sandboxes.
- Owners: PM, Tech Lead, DevOps
- Deliverables:
  - API acceptance criteria document
  - Staging env with test webhook endpoints
- Acceptance: design docs approved by Tech Lead and PM

Week 1 — Core Event Writes & Idempotency

- Epics:
  - Implement append-only `session_events` endpoint (POST /sessions/{id}/events).
  - Enforce idempotency_key dedupe at DB layer and return event_id/event_seq.
  - Unit tests for idempotency and ordering.
- Owners: Backend Eng, QA
- Deliverables:
  - POST event endpoint with idempotency tests
  - Error handling and rate limits
- Acceptance: tests passing; demo of write/retry behavior

Week 2 — Cursor GET, Snapshot & Session Lifecycle

- Epics:
  - Implement cursor-based GET for events and snapshot endpoint for UI hydration.
  - Implement session create (QR generation) and TTL/expiration logic.
  - Add session state transitions (active → paid → expired → cancelled).
- Owners: Backend Eng, Frontend (API contracts), QA
- Deliverables:
  - Cursor GET implemented and documented
  - Session create flow with QR URL generation
- Acceptance: snapshot + incremental fetch works end-to-end in staging

Week 3 — Payments Binding & Webhooks

- Epics:
  - Integrate payment gateway webhooks (stubs for MoMo/VNPay/ZaloPay) and normalize payment_reference.
  - Implement POST /sessions/{id}/payments and webhook handlers that mark session `paid` upon durable confirmation.
  - Provide manual payment API for merchant staff to mark cash.
- Owners: Backend Eng, Integration Eng, QA
- Deliverables:
  - Webhook handlers + manual payment endpoint
  - End-to-end test: gateway test webhook → session marked paid
- Acceptance: gateway webhook tests pass in staging; manual payment records link to session

Week 4 — Reconciliation Job & Merchant UI Scaffolding

- Epics:
  - Implement nightly reconciliation job per `architecture/reconciliation-job-spec.md`.
  - Create `reconciliation_issues` table and job pipeline.
  - Build basic merchant UI scaffolding: issues list, issue detail, manual payment modal (wireframe-driven).
- Owners: Backend Eng, Data Engineer, Frontend Eng, QA
- Deliverables:
  - Nightly job runnable in staging with sample issues created
  - Minimal React/Next pages for reconciliation list/detail
- Acceptance: job produces expected `reconciliation_issues` rows; UI lists them and can create manual payment

Week 5 — Analytics Ingestion & Aggregations

- Epics:
  - Stream `session_events` into analytics pipeline (materialized views or ETL into data warehouse).
  - Implement `daily_metrics` and top-selling items aggregation for the Dashboard API.
  - Add ingestion job monitoring and basic data quality checks (duplicate_event_rate, late_event_rate).
- Owners: Data Engineer, Backend Eng
- Deliverables:
  - Aggregation queries and dashboards in staging
  - Data quality alerts wired into monitoring
- Acceptance: Top-selling endpoint returns consistent values for test data

Week 6 — Observability, Load Testing & Security

- Epics:
  - Add metrics: session_created_count, payment_match_rate, duplicate_event_rate, reconciliation_issue_count.
  - Run load tests for polling patterns (cursor GET) and POST write traffic.
  - Verify security: auth guards, rate-limits, webhook verification, PII masking policies.
- Owners: SRE/DevOps, Backend Eng, QA
- Deliverables:
  - Dashboards and alerts configured
  - Load test report with tuning actions
- Acceptance: System meets latency/throughput targets for pilot scale and security checklist passes

Week 7 — Pilot Prep & Merchant Onboarding

- Epics:
  - Finalize merchant quickstart materials and runbook; schedule onboarding sessions.
  - Print QR assets and prepare pilot merchant packs.
  - Dry-run pilot onboarding with 1–2 friendly stores.
- Owners: Ops, PM, Frontend Eng
- Deliverables:
  - Onboarding schedule and merchant packs
  - Dry-run report and adjustments
- Acceptance: Dry-run successful with resolved issues and staff trained

Week 8 — Pilot Launch & Support

- Epics:
  - Onboard remaining pilot stores; enable verbose telemetry and reconciliation runs nightly.
  - Provide immediate instrumentation for first-week daily health checks.
  - Capture feedback and triage issues into backlog.
- Owners: PM, Ops, Backend Eng, Frontend Eng, Data Eng
- Deliverables:
  - Pilot live in 10 stores
  - Daily health report template and support rota
- Acceptance: Pilot launched and first daily health metrics reported

Post-pilot (Weeks 9–12)

- Analyze data quality and AI readiness; iterate on SKU mapping, data cleanup, and model training.
- Decide on POS research funding and roadmap based on pilot ROI and data quality.

Risks & Mitigations

- Risk: low order_capture_rate due to cash workflows → mitigate with training + simple manual payment UX.
- Risk: webhook flakiness → mitigate with retries, manual lookup by payment_reference, and reconciliation job.
- Risk: data pollution from abandoned carts → mitigate by excluding expired/abandoned sessions from revenue training and adding heuristics.

Acceptance Criteria (overall)

- APIs implemented as documented; test coverage for idempotency, ordering, and payment flows.
- Analytics pipeline produces daily_metrics and top-selling endpoints for pilot stores.
- Merchant reconciliation flow allows resolution of unmatched items within 24 hours.
- Pilot telemetry is reporting and alerts are configured.

# End of implementation timeline
